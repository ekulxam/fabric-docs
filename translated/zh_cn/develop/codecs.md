---
title: Codec
description: 一份用于理解和使用 Mojang 的 codec 系统以序列化和反序列化对象的全面指南。
authors:
  - enjarai
  - Syst3ms
---

Codec 是用于简单地解析 Java 对象的系统，被包含在 Minecraft 所包含的 Mojang 的 DataFixerUpper (DFU) 库中。 In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Codec 与 DFU 的另一个 API `DynamicOps` 一起使用。 A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## 使用 Codecs{#using-codecs}

### 序列化和反序列化 {#serializing-and-deserializing}

Codec 的基本用法是将对象序列化为特定格式或反序列化为特定格式。

一些原版的类已经定义了 Codecs，这些我们可以用作示例。 Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// 序列化该 BlockPos 为 JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

使用 codec 时，返回的结果为 `DataResult` 的形式。 This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

那么，让我们获取我们的序列化值，并将其转换回 `BlockPos` ：

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

### 内置的 Codec{#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

Codec API 自己也包含一些基础类型的 codec，例如 `Codec.INT` 和 `Codec.STRING`。 These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## 构建 Codec{#building-codecs}

现在我们已经知道如何使用 codec，让我们看看我们如何构建自己的 codec。 Suppose we have the following class, and we want to deserialize instances of it from JSON files:

```java
public class CoolBeansClass {

    private final int beansAmount;
    private final Item beanType;
    private final List<BlockPos> beanPositions;

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

我们可以将多个较小的 codec 组合成较大的 codec，从而为这个类制作 codec。 In this case, we'll need one for each field:

- 一个 `Codec<Integer>`
- 一个 `Codec<Item>`
- 一个 `Codec<List<BlockPos>>`

第一个可以从前面提到的 `Codec` 类中的基本类型 codec 中得到，也就是 `Codec.INT`。 While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. 我们没有用于 `List<BlockPos>` 的默认 codec，但我们可以从 `BlockPos.CODEC` 制作一个。

### 列表{#lists}

`Codec#listOf` 可用于创建任意 codec 的列表版本。

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

应该注意的是，以这种方式创建的 codec 总是会反序列化为一个 `ImmutableList`。 If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### 合并用于类似 Record 类的 Codec{#merging-codecs-for-record-like-classes}

现在每个字段都有了单独的 codec，我们可以使用 `RecordCodecBuilder` 为我们的类将其合并为一个 codec。 This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

来看看如何为我们的 `CoolBeansClass` 创建一个 codec：

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

在 group 中的每一行指定 codec、字段名称和 getter 方法。 The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

这里也可以使用 `Codec#optionalFieldOf` 使字段可选，在 [可选字段](#optional-fields) 章节会有解释。

### 不要将 MapCodec 与 Codec&lt;Map&gt; 混淆{#mapcodec}

调用 `Codec#fieldOf` 会将 `Codec<T>` 转换成 `MapCodec<T>`，这是 `Codec<T>` 的一个变体，但不是直接实现。 正如其名称所示，`MapCodec` 保证序列化为
键到值的映射，或所使用的 `DynamicOps` 类似类型。 一些函数可能需要使用 `MapCodec` 而不是常规的 codec。

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

但当使用 `BlockPos.CODEC.fieldOf("pos")` 转换为 `MapCodec<BlockPos>` 时，看起来像是这样：

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### 可选字段{#optional-fields}

`Codec#optionalFieldOf` 可用于创建一个可选的 map codec。 This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### 常量、约束和组合{#constants-constraints-composition}

#### Unit{#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### 数值范围{#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. 这适用于序列化和反序列化。

```java
// 不能大于 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Pair{#pair}

`Codec.pair` 将两个 codec `Codec<A>` 和 `Codec<B>` 合并为 `Codec<Pair<A, B>>`。 Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
结果的 codec 将序列化为结合了两个使用的 codec 字段的 map。

例如，运行这些代码：

```java
// 创建两个单独的装箱的 codec
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// 将其合并为 pair codec
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// 用它序列化数据
DataResult<JsonElement> result = pairCodec.encodeStart(JsonOps.INSTANCE, Pair.of(23, true));
```

Will output this JSON:

```json
{
  "i_am_number": 23,
  "this_statement_is_false": true
}
```

#### Either{#either}

`Codec.either` 将两个 codec `Codec<A>` 和 `Codec<B>` 组合为 `Codec<Either<A, B>>`。 The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
如果第二个也失败，则会返回第二个 codec 的错误。

#### Map{#maps}

要处理有任意键的 map，如 `HashMap`，可以使用 `Codec.unboundedMap`。 这将返回给定 `Codec<K>` 和 `Codec<V>` 的 `Codec<Map<K, V>>`。 The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. 看看下面的例子：

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

### 相互可转换的类型{#mutually-convertible-types}

#### `xmap`{#xmap}

我们有两个可以互相转换的类，但没有继承关系。 For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` 已有 codec，但让我们假装它不存在。 我们可以基于 `Vec3d` 的 codec 这样为它创建一个：

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // 转换 Vec3d 到 BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // 转换 BlockPos 到 Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// 当转换一个存在的类 （比如 `X`）到您自己的类 (`Y`)
// 最好是在 `Y` 类中添加 `toX` 和静态的 `fromX` 方法
// 并且使用在您的 `xmap` 调用中使用方法引用
```

#### flatComapMap、comapFlatMap 与 flatXMap{#flatcomapmap-comapflatmap-flatxmap}

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
正因此，其内置 codec 实际上是 `Codec.STRING` 上的 `comapFlatMap`，很好地说明了如何使用：

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

| 方法                      | A -> B 总是有效？ | B -> A 总是有效？ |
| ----------------------- | ------------ | ------------ |
| `Codec<A>#xmap`         | Yes          | Yes          |
| `Codec<A>#comapFlatMap` | 否            | Yes          |
| `Codec<A>#flatComapMap` | Yes          | 否            |
| `Codec<A>#flatXMap`     | 否            | 否            |

### 注册表分派{#registry-dispatch}

`Codec#dispatch` 让我们可以定义一个 codec 的注册表，并根据序列化数据中字段的值分派到一个特定的 codec。 This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

例如我们有一个抽象的 `Bean` 接口与两个实现类：`StringyBean` 和 `CountingBean`。 To serialize these with a registry dispatch, we'll need a few things:

- 每个 bean 类型的独立 codec。
- 一个 `BeanType<T extends Bean>` 类或 record，代表 bean 的类型并可返回它的 codec。
- 一个在 `Bean` 中可以用于检索其 `BeanType<?>` 的函数。
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- 一个基于该注册表的 `Codec<BeanType<?>>`。 If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

有了这些，就可以创建一个 bean 的注册表分派 codec。

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// 现在我们可以创建一个用于 bean 类型的 codec
// 基于之前创建的注册表
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// 基于那个，这是我们用于 bean 的注册表分派 codec！
// 第一个参数是这个 bean 类型的字段名
// 当省略时默认是 "type"。
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

### 递归 Codec{#recursive-codecs}

有时，使用_自身_来解码特定字段的 codec 很有用，例如在处理某些递归数据结构时。 在原版代码中，这用于 `Component` 对象，可能会存储其他的 `Component` 作为子对象。 可以使用 `Codec#recursive` 构建这样的 codec。

例如，让我们尝试序列化单链列表。 列表是由一组节点的表示的，这些节点既包含一个值，也包含对列表中下一个节点的引用。 然后列表由其第一个节点表示，遍历列表是通过跟随下一个节点来完成的，直到没有剩余节点。 以下是存储整数的节点的简单实现。

```java
public record ListNode(int value, ListNode next) {}
```

我们无法通过普通方法为此构建 codec，因为对 `next` 字段要使用什么 codec？ 我们需要一个 `Codec<ListNode>`，这就是我们还在构建的！ `Codec#recursive` 能让我们使用看上去像魔法的 lambda 来达到这点。

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // codec的名称
  selfCodec -> {
    // 这里，`selfCodec` 代表 `Codec<ListNode>`，就像已经构造好了一样
    // 这个 lambda 应该返回我们从一开始就想要使用的 codec，
    // 通过 `selfCodec` 引用自身
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // `next`字段将使用自己 codec 递归处理
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

序列化的 `ListNode` 可能看起来像这样：

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

## 参考{#references}

- 关于 codec 及相关 API 的更全面的文档，可以在[非官方 DFU JavaDoc](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec) 中找到。
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
