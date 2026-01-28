---
title: 코덱
description: 객체를 직렬화 혹은 역직렬화하기 위한 Mojang의 코덱 시스템을 이해하고 사용하기 위한 종합 가이드입니다.
authors:
  - enjarai
  - Syst3ms
---

코덱은 Java 객체를 쉽게 직렬화하기 위한 시스템이며, Minecraft에 포함되어 있는 Mojang의 DataFixerUpper (DFU) 라이브러리에 포함되어 있습니다. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

코덱은 DFU의 다른 API, `DynamicOps`와 함께 사용됩니다. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## 코덱 사용하기 {#using-codecs}

### 직렬화와 역직렬화 {#serializing-and-deserializing}

코덱의 기본 사용법은 객체를 특정한 형식으로 직렬화 혹은 역직렬화하는 것입니다.

몇몇 기본 코덱이 이미 정의되었으므로, 이를 예시로 사용할 수 있습니다. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// BlockPos를 JsonElement로 직렬화합니다
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

코덱을 사용하면, 값은 `DataResult`의 형식으로 반환됩니다. This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

그러면 직렬화된 값을 가져와 다시 `BlockPos`로 바꾸어 보겠습니다:

```java
// When actually writing a mod, you'll want to properly handle empty Optionals of course
JsonElement json = result.resultOrPartial(LOGGER::error).orElseThrow();

// Here we have our JSON value, which should correspond to `[1, 2, 3]`,
// as that's the format used by the BlockPos codec.
LOGGER.info("Serialized BlockPos: {}", json);

// Now we'll deserialize the JsonElement back into a BlockPos
DataResult<BlockPos> result = BlockPos.CODEC.parse(JsonOps.INSTANCE, json);

// Again, we'll just grab our value from the result
BlockPos pos = result.resultOrPartial(LOGGER::error).orElseThrow();

// And we can see that we've successfully serialized and deserialized our BlockPos!
LOGGER.info("Deserialized BlockPos: {}", pos);
```

### 내장된 코덱 {#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

Codec API는 또한 자체로 `Codec.INT`와 `Codec.STRING`과 같은 기본 유형에 대한 일부 코덱을 포함하고 있습니다. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## 코덱 만들기 {#building-codecs}

이제 코덱을 사용하는 방법을 보았으니, 코덱을 어떻게 만들 수 있는지 살펴보겠습니다. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

```java
public class CoolBeansClass {

    private final int beansAmount;
    private final Item beanType;
    private final List<Integer> beanPositions;

    public CoolBeansClass(int beansAmount, Item beanType, List<BlockPos> beanPositions) {...}

    public int getBeansAmount() { return this.beansAmount; }
    public Item getBeanType() { return this.beanType; }
    public List<BlockPos> getBeanPositions() { return this.beanPositions; }
}
```

The corresponding JSON file might look something like this:

```json
{
  "beans_amount": 5,
  "bean_type": "beanmod:mythical_beans",
  "bean_positions": [
    [1, 2, 3],
    [4, 5, 6]
  ]
}
```

우리는 작은 코덱 여러개를 더 큰 코덱으로 합쳐서 이 클래스를 위한 코덱을 만들 수 있습니다. In this case, we'll need one for each field:

- `Codec<Integer>`
- `Codec<Item>`
- `Codec<List<BlockPos>>`

`Codec` 클래스에 정의된 기본 코덱들 중에서, 특히 `Codec.INT`를 사용해서 첫 번째 값을 얻을 수 있습니다. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. `List<BlockPos>`에 대한 기본 코덱이 없지만 `BlockPos.CODEC`에서 만들 수 있습니다.

### 리스트 {#lists}

`Codec#listOf`는 모든 버전의 코덱을 만드는 데 이용할 수 있습니다:

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

이러한 방식으로 생성된 코덱은 항상 `ImmutableList`로 역직렬화된다는 점에 유의해야 합니다. If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### 레코드와 같은 클래스를 위한 코덱 합치기 {#merging-codecs-for-record-like-classes}

이제 각 필드마다 별도의 코덱을 구성했으므로, RecordCodecBuilder를 사용해 이를 하나의 클래스용 코덱으로 합칠 수 있습니다. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

이제 우리 CoolBeansClass에 대한 코덱을 생성하는 방법을 살펴보겠습니다:

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

group 내의 각 줄은 코덱, 필드 이름, 그리고 getter 메서드를 지정합니다. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

You can also use `Codec#optionalFieldOf` in this context to make a field optional, as explained in
the [Optional Fields](#optional-fields) section.

### Codec과 구별해야 할 MapCodec<0> {#mapcodec}

Calling `Codec#fieldOf` will convert a `Codec<T>` into a `MapCodec<T>`, which is a variant, but not direct
implementation of `Codec<T>`. `MapCodec`s, as their name suggests are guaranteed to serialize into a
key to value map, or its equivalent in the `DynamicOps` used. 어느 함수는 일반 코덱이 필요하지 않을 수 있습니다.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

But when converted into a `MapCodec<BlockPos>` using `BlockPos.CODEC.fieldOf("pos")`, it would look like this:

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### 추가 필드 {#optional-fields}

`Codec#optionalFieldOf` can be used to create an optional map codec. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### 상수, 제한사항, 그리고 구성 {#constants-constraints-composition}

#### Unit {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### 숫자 범위 {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. 이는 직렬화와 역질렬화에도 적용됩니다.

```java
// 2보다 클 수 없음
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Pair{#pair}

`Codec.pair` merges two codecs, `Codec<A>` and `Codec<B>`, into a `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
The resulting codec will serialize to a map combining the fields of both codecs used.

예를 들어, 아래의 코드를 실행하면:

```java
// 두 개의 구분된 코덱 생성
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// 하나의 짝 코덱으로 병합
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// 직렬화
DataResult<JsonElement> result = pairCodec.encodeStart(JsonOps.INSTANCE, Pair.of(23, true));
```

Will output this JSON:

```json
{
  "i_am_number": 23,
  "this_statement_is_false": true
}
```

#### Either {#either}

`Codec.either` combines two codecs, `Codec<A>` and `Codec<B>`, into a `Codec<Either<A, B>>`. The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
If the second one also fails, the error of the _second_ codec will be returned.

#### Maps {#maps}

For processing maps with arbitrary keys, such as `HashMap`s, `Codec.unboundedMap` can be used. This returns a
`Codec<Map<K, V>>` for a given `Codec<K>` and `Codec<V>`. The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. 아래의 예시를 참고하세요:

```java
// Create a codec for a map of Identifiers to integers
Codec<Map<Identifier, Integer>> mapCodec = Codec.unboundedMap(Identifier.CODEC, Codec.INT);

// Use it to serialize data
DataResult<JsonElement> result = mapCodec.encodeStart(JsonOps.INSTANCE, Map.of(
    Identifier.fromNamespaceAndPath("example", "number"), 23,
    Identifier.fromNamespaceAndPath("example", "the_cooler_number"), 42
));
```

This will output this JSON:

```json
{
  "example:number": 23,
  "example:the_cooler_number": 42
}
```

As you can see, this works because `Identifier.CODEC` serializes directly to a string value. A similar effect can be achieved for simple objects that don't serialize to strings by using [xmap & friends](#mutually-convertible-types) to convert them.

### 가변적 타입(Mutually Convertible Types) {#mutually-convertible-types}

#### `xmap` {#xmap}

Say we have two classes that can be converted to each other, but don't have a parent-child relationship. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos`는 이미 코덱을 가지고 있지만, 여기서는 사용하지 않습니다. We can create one for it by basing it on the
codec for `Vec3d` like this:

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Convert Vec3d to BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Convert BlockPos to Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// 이미 존재하는 클래스를 (예를 들어 `X`) 다른 클래스로 변환할 때 (`Y`)
// 메서드 `toX`와 정적 메서드`fromX`를 `Y`에 추가하고,
// `xmap`호출에서 메서드 래퍼런스를 사용해도 됨.
```

#### flatComapMap, comapFlatMap과 flatXMap {#flatcomapmap-comapflatmap-flatxmap}

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
Because of this, its built-in codec is actually a `comapFlatMap` on `Codec.STRING`, nicely
illustrating how to use it:

```java
public class Identifier {
    public static final Codec<Identifier> CODEC = Codec.STRING.comapFlatMap(
        Identifier::validate, Identifier::toString
    );

    // ...

    public static DataResult<Identifier> validate(String id) {
        try {
            return DataResult.success(Identifier.parse(id));
        } catch (InvalidIdentifierException e) {
            return DataResult.error("Not a valid identifier: " + id + " " + e.getMessage());
        }
    }

    // ...
}
```

While these methods are really helpful, their names are a bit confusing, so here's a table to help you remember which one to use:

| 메소드                     | A -> B는 항상 유효한가? | B -> A는 항상 유효한가? |
| ----------------------- | ---------------- | ---------------- |
| `Codec<A>#xmap`         | Yes              | Yes              |
| `Codec<A>#comapFlatMap` | 불가               | Yes              |
| `Codec<A>#flatComapMap` | Yes              | 불가               |
| `Codec<A>#flatXMap`     | 불가               | 불가               |

### 레지스트리 전송 {#registry-dispatch}

`Codec#dispatch`는 코덱의 레지스트리를 정의하도록 하고, 직렬화된 데이터의 어느 필드 값으로 특정한 곳에 전달할 수 있도록 합니다. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

예를 들어, 추상 인터페이스 `Bean`를 구현하는 두 클래스 `StringyBean`과 `CountingBean`이 있다고 가정해 봅시다. To serialize these with a registry dispatch, we'll need a few things:

- bean의 모든 종류에 대한 코덱을 분리하기.
- A `BeanType<T extends Bean>` class or record that represents the type of bean, and can return the codec for it.
- `BeanType<?>`를 가져오기 위한 `Bean`내의 함수.
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- 이 레지스트리에 대한 `Codec<BeanType<?>>`. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

이와 같이, bean의 레지스트리 디스패치 코덱을 만들 수 있습니다:

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// 이전에 만들어진 레지스트리로
// bean 타입 코덱을 생성함
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// 그 레지스트리를 기반으로 bean타입의 레지스트리 디스패치 코덱을 만듦
// 첫 번째 매개변수는 bean타입의 필드명
// 비워두면, 기본값인 "type" 으로 설정됨
Codec<Bean> beanCodec = beanTypeCodec.dispatch("type", Bean::getType, BeanType::codec);
```

Our new codec will serialize beans to JSON like this, grabbing only fields that are relevant to their specific type:

```json
{
  "type": "example:stringy_bean",
  "stringy_string": "This bean is stringy!"
}
```

```json
{
  "type": "example:counting_bean",
  "counting_number": 42
}
```

### 반복적 코덱 {#recursive-codecs}

Sometimes it is useful to have a codec that uses _itself_ to decode specific fields, for example when dealing with certain recursive data structures. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Such a codec can be constructed using `Codec#recursive`.

예를 들어, 단방향 연결 리스트를 직렬화하려고 합니다. This way of representing lists consists of a bunch of nodes that hold both a value and a reference to the next node in the list. The list is then represented by its first node, and traversing the list is done by following the next node until none remain. Here is a simple implementation of nodes that store integers.

```java
public record ListNode(int value, ListNode next) {}
```

We can't construct a codec for this by ordinary means, because what codec would we use for the `next` field? We would need a `Codec<ListNode>`, which is what we are in the middle of constructing! `Codec#recursive`는 특별한 람다 함수를 사용하여 도와줍니다:

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // codec 이름
  selfCodec -> {
    // `selfCodec`은 이미 생성된 것과 같은 `Codec<ListNode>`를 나타냄
    // 이 람다는 처음에 쓰려고 하던 codec을 반환함
    // 또한, `selfCodec`로 자신을 참조함
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // `next`필드는 본인의 코덱으로 반복적으로 처리될 것임
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

직렬화된 `ListNode`는 아래와 같습니다:

```json
{
  "value": 2,
  "next": {
    "value": 3,
    "next": {
      "value": 5
    }
  }
}
```

## 참조 {#references}

- A much more comprehensive documentation of Codecs and related APIs can be found at the
  [Unofficial DFU JavaDoc](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec).
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
