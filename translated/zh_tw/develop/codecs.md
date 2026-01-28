---
title: Codec
description: 用於理解和使用 Mojang 編解碼器系統來序列化和反序列化物件的綜合指南。
authors:
  - enjarai
  - Syst3ms
---

Codec 是一個方便序列化 Java 物件的系統，包含在 Mojang 的 DataFixerUpper (DFU) 函式庫中，而 DFU 包含在 Minecraft 中。 In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Codec 與 DFU 的另一個 API `DynamicOps` 結合使用。 A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## 使用 Codec {#using-codec}

### 序列化和反序列化 {#serializing-and-deserializing}

Codec 的基本用法是將物件序列化和反序列化為特定格式。

由於一些原版類別已經定義了 codec，我們可以將它們作為範例。 Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// 將 BlockPos 序列化為 JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

當使用 codec 時，數值會以 `DataResult` 的形式回傳。 This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

所以讓我們取得序列化的數值，並將它轉換回 `BlockPos`：

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

### 內建 Codec {#built-in-codec}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

Codec API 本身也包含一些用於原始類型的 codec，例如 `Codec.INT` 和 `Codec.STRING`。 These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## 建構 codec {#building-codec}

現在我們已經了解了如何使用 codec，讓我們來看看如何建立自己的 codec。 Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

我們可以將多個較小的 codec 組合到一個較大的 codec 中，來為這個類別建立 codec。 In this case, we'll need one for each field:

- 一個 `Codec<Integer>`
- 一個 `Codec<Item>`
- 一個 `Codec<List<BlockPos>>`

我們可以從前面提到的 `Codec` 類別中的原始 codec 取得第一個，具體來說就是 `Codec.INT`。 While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. 我們沒有用於 `List<BlockPos>` 的預設 codec，但我們可以從 `BlockPos.CODEC` 建立一個。

### 清單 {#lists}

`Codec#listOf` 可用於建立任何 codec 的清單版本：

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

應該注意的是，以這種方式建立的 codec 將始終反序列化為 `ImmutableList`。 If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### 合併用於 Record 類別的 Codec {#merging-codec-for-record-like-classes}

現在我們有每個欄位的單獨 codec，我們可以將它們合併為一個用於我們的類別的 codec，使用 `RecordCodecBuilder`。 This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

讓我們看看如何為我們的 `CoolBeansClass` 建立編碼器：

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

群組中的每一行都指定了一個編碼器、一個欄位名稱和一個 getter 方法。 The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

您也可以在此上下文中使用 `Codec#optionalFieldOf` 來使欄位成為可選欄位，如 [可選欄位](#optional-fields) 區段中所述。

### MapCodec，不要與 Codec&amp;lt;Map&amp;gt; 混淆 {#mapcodec}

呼叫 `Codec#fieldOf` 會將 `Codec<T>` 轉換為 `MapCodec<T>`，這是 `Codec<T>` 的變體，但不是直接實作。 `MapCodec`，顧名思義，保證序列化為鍵到值的映射，或其在使用的 `DynamicOps` 中的等效項。 某些函數可能需要一個而不是普通的編碼器。

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

但當使用 `BlockPos.CODEC.fieldOf("pos")` 將其轉換為 `MapCodec<BlockPos>` 時，它將如下所示：

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### 可選欄位 {#optional-fields}

`Codec#optionalFieldOf` 可以用來建立可選的 map 編碼器。 This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### 常數、約束和組合 {#constants-constraints-composition}

#### 單元 {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### 數值範圍 {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. 這適用於序列化和反序列化。

```java
// 不能超過 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### 配對 {#pair}

`Codec.pair` 將兩個編碼器 `Codec<A>` 和 `Codec<B>` 合併為 `Codec<Pair<A, B>>`。 Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
產生的編碼器將序列化為一個結合兩個使用編碼器欄位的映射。

例如，執行以下程式碼：

```java
// 建立兩個獨立的已封裝編碼器
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// 將它們合併為配對編碼器
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// 使用它來序列化資料
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

`Codec.either` 將兩個 codec `Codec<A>` 和 `Codec<B>` 合併到一個 `Codec<Either<A, B>>` 中。 The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
如果第二個編碼器也失敗，則會回傳**第二個**編碼器的錯誤。

#### Maps {#maps}

為了處理具有任意鍵的 maps，例如 `HashMap`s，可以使用 `Codec.unboundedMap`。 這會為指定的 `Codec<K>` 和 `Codec<V>` 回傳一個 `Codec<Map<K, V>>`。 The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. 請參閱以下範例：

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

### 相互轉換的類型 {#mutually-convertible-types}

#### `xmap` {#xmap}

假設我們有兩個可以互相轉換的類別，但沒有父子關係。 For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` 已經有一個 codec，但讓我們假裝它沒有。 我們可以透過基於 `Vec3d` 的 codec 來為它建立一個：

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // 將 Vec3d 轉換為 BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // 將 BlockPos 轉換為 Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// 當以這種方式將現有類別（例如 `X`）
// 轉換為你自己的類別 (`Y`) 時，最好
// 在 `Y` 中新增 `toX` 和 static `fromX` 方法，並在你的 `xmap`
// 呼叫中使用方法參考。
```

#### flatComapMap、comapFlatMap 和 flatXMap {#flatcomapmap-comapflatmap-flatxmap}

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
因此，它的內建 codec 實際上是 `Codec.STRING` 上的 `comapFlatMap`，很好地說明了如何使用它：

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

| 方法                      | A -> B 一律有效嗎？ | B -> A 一律有效嗎？ |
| ----------------------- | ------------- | ------------- |
| `Codec<A>#xmap`         | Yes           | Yes           |
| `Codec<A>#comapFlatMap` | 否             | Yes           |
| `Codec<A>#flatComapMap` | Yes           | 否             |
| `Codec<A>#flatXMap`     | 否             | 否             |

### 登錄分派 {#registry-dispatch}

`Codec#dispatch` 讓我們定義一個 codecs 的登錄，並根據序列化資料中的欄位值分派到特定的 codec。 This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

例如，假設我們有一個抽象的 `Bean` 介面，有兩個實作類別：`StringyBean` 和 `CountingBean`。 To serialize these with a registry dispatch, we'll need a few things:

- 每個 bean 類型的單獨 codec。
- 一個 `BeanType<T extends Bean>` 類別或 record，它代表 bean 的類型，並且可以回傳它的 codec。
- `Bean` 上的一個函數，用於檢索它的 `BeanType<?>`。
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- 一個基於此登錄的 `Codec<BeanType<?>>`。 If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

有了所有這些，我們可以為 beans 建立一個登錄分派 codec：

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// 現在我們可以基於先前建立的登錄，建立一個用於 bean 類型的 codec
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// 基於此，這是我們用於 beans 的登錄分派 codec！
// 第一個參數是 bean 類型的欄位名稱。
// 預設情況下，它將預設為 "type"。
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

### 遞迴 Codec {#recursive-codecs}

有時，擁有一個使用 _自身_ 來解碼特定欄位的 codec 很有用，例如在處理某些遞迴資料結構時。 In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. 可以使用 `Codec#recursive` 建構這樣的 codec。

例如，讓我們嘗試序列化一個單向連結的清單。 這種表示清單的方式由一堆節點組成，這些節點既儲存一個值，又儲存對清單中下一個節點的引用。 然後，清單由其第一個節點表示，並且遍歷清單是透過跟隨下一個節點直到沒有剩餘的節點來完成的。 這是一個儲存 integers 的節點的簡單實作。

```java
public record ListNode(int value, ListNode next) {}
```

我們不能透過普通方式為此建構一個 codec，因為我們會為 `next` 欄位使用什麼 codec 呢？ 我們需要一個 `Codec<ListNode>`，這正是我們正在建構的！ `Codec#recursive` 讓我們使用一個看起來很神奇的 lambda 來實現這一點：

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // codec 的名稱
  selfCodec -> {
    // 在這裡，`selfCodec` 代表 `Codec<ListNode>`，就像它已經被建構好一樣
    // 這個 lambda 應該回傳我們從一開始就想使用的 codec，
    // 這個 codec 會透過 `selfCodec` 引用自身
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // `next` 欄位將使用 self-codec 遞迴處理
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

序列化的 `ListNode` 可能看起來像這樣：

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

## 參考文獻 {#references}

- 可以在[非官方 DFU JavaDoc](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec) 中找到有關 Codec 和相關 API 的更全面的文件。
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
