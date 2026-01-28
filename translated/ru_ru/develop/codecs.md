---
title: Кодеки
description: Исчерпывающее руководство по пониманию и использованию системы кодеков Mojang для сериализации и десериализации объектов.
authors:
  - enjarai
  - Syst3ms
---

Codec — это система для простой сериализации объектов Java, входящая в библиотеку DataFixerUpper (DFU) от Mojang, которая включена в Minecraft. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Кодеки используются вместе с другим API из DFU — `DynamicOps`. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## Использование кодеков {#using-codecs}

### Сериализация и десериализация {#serializing-and-deserializing}

Основное использование кодека — это сериализация и десериализация объектов в определённый формат и из него.

Поскольку для некоторых стандартных классов уже определены кодеки, мы можем использовать их в качестве примера. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// Сериализуем BlockPos в JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

Когда мы используем кодек, возвращаемые данные будут в виде `DataResult`. This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

Итак, теперь возьмём сериализованное нами значение и превратим его обратно в `BlockPos`:

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

### Встроенные кодеки {#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

Сам API Codec также содержит некоторые кодеки для примитивных типов, такие как `Codec.INT` и `Codec.STRING`. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## Сборка кодеков {#building-codecs}

Теперь, когда мы увидели, как использовать кодеки, давайте посмотрим, как мы можем собрать свои собственные. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

Мы можем создать кодек для этого класса, объединив несколько меньших кодеков в один большой. In this case, we'll need one for each field:

- `Codec<Integer>`
- `Codec<Item>`
- `Codec<List<BlockPos>>`

Мы можем получить первый из упомянутых ранее примитивных кодеков в классе `Codec`, а именно `Codec.INT`. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. У нас нет стандартного кодека для `List<BlockPos>`, но мы можем создать его из `BlockPos.CODEC`.

### Списки {#lists}

Метод `Codec#listOf` можно использовать для создания версии кодека для списка любого типа:

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

Следует отметить, что кодеки, созданные таким образом, всегда будут десериализоваться в `ImmutableList`. If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### Объединение кодеков для классов, похожих на Record {#merging-codecs-for-record-like-classes}

Теперь, когда у нас есть отдельные кодеки для каждого поля, мы можем объединить их в один кодек для нашего класса, используя `RecordCodecBuilder`. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Давайте посмотрим, как создать кодек для нашего `CoolBeansClass`:

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

Каждая строка в группе указывает кодек, название поля и метод доступа. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

Вы также можете использовать `Codec#optionalFieldOf` в этом контексте, чтобы сделать поле необязательным, как объясняется в разделе [«Необязательные поля»](#optional-fields).

### MapCodec, не путать с Codec&amp;lt;Map&amp;gt; {#mapcodec}

Вызов `Codec#fieldOf` преобразует `Codec<T>` в `MapCodec<T>`, который является вариантом, но не прямой реализацией `Codec<T>`. `MapCodec`, как следует из названия, гарантированно сериализуется в карту «ключ-значение» или её эквивалент в используемых `DynamicOps`. Некоторые функции могут требовать именно MapCodec вместо обычного кодека.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

Но при преобразовании в `MapCodec<BlockPos>` с помощью `BlockPos.CODEC.fieldOf("pos")` он будет выглядеть так:

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### Необязательные поля {#optional-fields}

`Codec#optionalFieldOf` можно использовать для создания необязательного MapCodec. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### Константы, ограничения и композиция {#constants-constraints-composition}

#### Unit {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### Числовые диапазоны {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. Это применяется как к сериализации, так и к десериализации.

```java
// Не может быть больше 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Пары {#pair}

`Codec.pair` объединяет два кодека, `Codec<A>` и `Codec<B>`, в `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
Полученный кодек будет сериализоваться в карту, объединяющую поля обоих используемых кодеков.

Например, запуск такого кода:

```java
// Создаём два отдельных коробочных кодека
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// Объединяем их в пару кодеков
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// Используем её для сериализации данных
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

`Codec.either` объединяет два кодека, `Codec<A>` и `Codec<B>`, в `Codec<Either<A, B>>`. The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
Если второй также не справится, будет возвращена ошибка _второго_ кодека.

#### Карты {#maps}

Для обработки карт с произвольными ключами, таких как `HashMap`, можно использовать `Codec.unboundedMap`. Это возвращает `Codec<Map<K, V>>` для заданных `Codec<K>` и `Codec<V>`. The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. Смотрите пример ниже:

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

### Взаимно преобразуемые типы {#mutually-convertible-types}

#### `xmap` {#xmap}

Предположим, у нас есть два класса, которые могут быть преобразованы друг в друга, но не имеют отношения наследования. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` уже имеет кодек, но предположим, что его нет. Мы можем создать его, основываясь на кодеке для `Vec3d`, следующим образом:

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Преобразование Vec3d в BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Преобразование BlockPos в Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// При преобразовании существующего класса (например, `X`)
// в ваш собственный класс (`Y`) таким образом, может быть полезно
// добавить методы `toX` и статический `fromX` в `Y` и использовать
// ссылки на методы в вашем вызове `xmap`.
```

#### flatComapMap, comapFlatMap и flatXMap {#flatcomapmap-comapflatmap-flatxmap}

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
Из-за этого его встроенный кодек на самом деле является `comapFlatMap` на `Codec.STRING`, что хорошо иллюстрирует, как его использовать:

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

| Метод                   | A → B всегда допустимо? | B → A всегда допустимо? |
| ----------------------- | ----------------------- | ----------------------- |
| `Codec<A>#xmap`         | Yes                     | Yes                     |
| `Codec<A>#comapFlatMap` | Нет                     | Yes                     |
| `Codec<A>#flatComapMap` | Yes                     | Нет                     |
| `Codec<A>#flatXMap`     | Нет                     | Нет                     |

### Диспетчер реестров {#registry-dispatch}

`Codec#dispatch` позволяет нам определить реестр кодеков и направлять к конкретному кодеку на основе значения поля в сериализованных данных. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

Например, предположим, у нас есть абстрактный интерфейс `Bean` с двумя реализующими классами: `StringyBean` и `CountingBean`. To serialize these with a registry dispatch, we'll need a few things:

- Отдельные кодеки для каждого типа бобов (beans; как мы их назвали ранее);
- Класс или запись `BeanType<T extends Bean>`, представляющий тип боба и способный возвращать кодек для него;
- Функция в `Bean` для получения его `BeanType<?>`;
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- `Codec<BeanType<?>>`, основанный на этом реестре. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

Имея всё это, мы можем создать кодек диспетчера реестров для бобов:

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// Теперь мы можем создать кодек для типов бобов
// на основе ранее созданного реестра
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// И на основе этого, вот наш кодек диспетчера реестра для бобов!
// Первый аргумент — это название поля для типа боба.
// Если его не указать, по умолчанию будет «type».
Codec<Bean> beanCodec = beanTypeCodec.dispatch("type", Bean::getType, BeanType::codec);
```

Our new codec will serialize beans to JSON like this, grabbing only fields that are relevant to their specific type:

```json
{
  "type": "example:stringy_bean",
  "stringy_string": "Этот боб — строка!"
}
```

```json
{
  "type": "example:counting_bean",
  "counting_number": 42
}
```

### Рекурсивные кодеки {#recursive-codecs}

Иногда бывает полезно иметь кодек, который использует сам себя для декодирования определённых полей, например, при работе с некоторыми рекурсивными структурами данных. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Такой кодек можно построить с помощью `Codec#recursive`.

Например, давайте попробуем сериализовать односвязный список. Этот способ представления списков состоит из набора узлов, которые хранят значение и ссылку на следующий узел в списке. Список представлен своим первым узлом, и обход списка осуществляется путём следования по следующему узлу, пока они не закончатся. Вот простая реализация узлов, которые хранят целые числа:

```java
public record ListNode(int value, ListNode next) {}
```

Мы не можем создать кодек для этого обычными средствами, потому что какой кодек мы будем использовать для поля `next`? Нам нужен `Codec<ListNode>`, который мы как раз и пытаемся создать! `Codec#recursive` позволяет нам достичь этого, используя «волшебную» лямбду:

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // название для кодека
  selfCodec -> {
    // Здесь `selfCodec` представляет `Codec<ListNode>`, как если бы он уже был создан
    // Эта лямбда должна возвращать кодек, который мы хотим использовать, и который ссылается на себя через `selfCodec`
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
        // поле `next` будет обрабатываться рекурсивно с помощью `selfCodec`
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

Сериализованный `ListNode` может выглядеть так:

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

## Ссылки {#references}

- Более полную документацию по кодекам и связанным API можно найти в [неофициальной Java-документации DFU](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec);
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
