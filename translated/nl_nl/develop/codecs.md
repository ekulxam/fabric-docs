---
title: Codecs
description: Een duidelijke handleiding om Mojang's codec systeem te begrijpen en te gebruiken om objecten te serialiseren en te deserialiseren.
authors:
  - enjarai
  - Syst3ms
---

Codec is een systeem om makkelijk Java objecten te serialiseren, en is bij Mojang's DataFixerUpper(DFU) bibliotheek inbegrepen, die inbegrepen is bij Minecraft. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Codecs worden gebruikt in samenwerking met een andere API van DFU, `DynamicOps`. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## Codecs Gebruiken {#using-codecs}

### Serialiseren en Deserialiseren {#serializing-and-deserializing}

Het basis gebruik van een codec is om objecten van en naar een specifiek formaat te serialiseren en te deserialiseren.

Aangezien er enkele vanilla klassen al codecs hebben gedefinieerd, kunnen we deze gebruiken als voorbeeld. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3); 

// Serialiseer de BlockPos naar ren JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

Wanneer je een codec gebruikt, worden waarden geretourneerd in de vorm van een `DataResult`. This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

Dus laten we onze geserialiseerde waarde nemen en het terug in een `BlockPos` veranderen:

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

### Ingebouwde Codecs {#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

De Codec API zelf bevat ook enkele codecs voor primitieve types, zoals `Codec.INT` en `Codec.STRING`. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## Codecs Maken {#building-codecs}

Nu dat we hebben gezien hoe je een codec gebruikt, laten we eens kijken naar hoe we er zelf één kunnen bouwen. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

We kunnen een codec maken voor deze klasse door de vele kleinere codecs samen te voegen tot een grotere. In this case, we'll need one for each field:

- een `Codec<Integer>`
- een `Codec<Item>`
- een `Codec<List<BlockPos>>`

We kunnen de eerste verkrijgen van de voorafgenoemde primitieve codecs in de `Codec` klasse, specifiek `Codec.INT`. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. We hebben geen standaard codec voor `List<BlockPos>`, maar we kunnen één maken van `BlockPos.CODEC`.

### Lijsten {#lists}

`Codec#listOf` kan gebruikt worden om een lijst versie van elke codec te maken:

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

Merk op dat codecs die op deze manier gemaakt worden, altijd geserialiseerd worden tot een `ImmutableList`. If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### Codecs samenvoegen voor Record-achtige Klassen{#merging-codecs-for-record-like-classes}

Nu dat we aparte codecs hebben voor elk veld, kunnen we ze combineren in één grote codec voor onze klasse met een `RecordCodecBuilder`. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Laten we eens kijken hoe we een codec maken voor onze `CoolBeansClass`:

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

Elke regel in de groep specifieert een codec, een veldnaam, en een getter methode. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

Je kan ook `Codec#optionalFieldOf` gebruiken in deze context op een veld optioneel te maken, zoals omschrijven in de [Optionele Velden](#optional-fields) sectie.

### MapCodec, Niet te Verwarren Met Codec&lt;Map&gt;{#mapcodec}

Een aanroep van `Codec#fieldOf` zal een `Codec<T>` in een `MapCodec<T>` veranderen, wat een variant, maar geen rechtstreekse implementatie is van `Codec<T>`. `MapCodec`s, zoals hun naam ook voorstelt zijn gegarandeerd om te serialiseren als een sleutel naar waarde map, of zijn equivalent in de `DynamicOps`. Enkele functies kunnen er één nodig hebben in plaats van een normale codec.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

Maar wanneer deze in een `MapCodec<BlockPos>` wordt veranderd door middel van \`BlockPos.CODEC.fieldOf("pos"), zou het er als volgt uit kunnen zien:

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### Optionele Velden {#optional-fields}

`Codec#optionalFieldOf` kan gebruikt worden om een optionele map codec aan te maken. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### Constanten, Beperkingen en Compositie {#constants-constraints-composition}

#### Eenheid {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### Numerieke Bereiken {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. Dit geldt voor zowel serialisatie en deserialisatie.

```java
// Mag niet meer dan 2 zijn
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Paar {#pair}

`Codec.pair` voegt twee codecs, `Codec<A>` en `Codec<B>, samen tot `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec\`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
De resulterende codec zal serialiseren als een map die de velden van beide gebruikte codecs combineert.

Bijvoorbeeld, wanneer je deze code uitvoert:

```java
// Maak twee aparte ingekaderde codecs
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec(); 
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec(); 

// Vorg ze samen tot een pair codec
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec); 

// Gebruik het om data te serealiseren
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

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. See the example below:

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

### Mutually Convertible Types {#mutually-convertible-types}

#### `xmap` {#xmap}

Say we have two classes that can be converted to each other, but don't have a parent-child relationship. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` already has a codec, but let's pretend it doesn't. We can create one for it by basing it on the
codec for `Vec3d` like this:

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Convert Vec3d to BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Convert BlockPos to Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// When converting an existing class (`X` for example)
// to your own class (`Y`) this way, it may be nice to
// add `toX` and static `fromX` methods to `Y` and use
// method references in your `xmap` call.
```

#### flatComapMap, comapFlatMap, and flatXMap {#flatcomapmap-comapflatmap-flatxmap}

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

| Method                  | A -> B always valid? | B -> A always valid? |
| ----------------------- | -------------------- | -------------------- |
| `Codec<A>#xmap`         | Yes                  | Yes                  |
| `Codec<A>#comapFlatMap` | No                   | Yes                  |
| `Codec<A>#flatComapMap` | Yes                  | No                   |
| `Codec<A>#flatXMap`     | No                   | No                   |

### Registry Dispatch {#registry-dispatch}

`Codec#dispatch` let us define a registry of codecs and dispatch to a specific one based on the value of a
field in the serialized data. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

For example, say we have an abstract `Bean` interface with two implementing classes: `StringyBean` and `CountingBean`. To serialize these with a registry dispatch, we'll need a few things:

- Separate codecs for every type of bean.
- A `BeanType<T extends Bean>` class or record that represents the type of bean, and can return the codec for it.
- A function on `Bean` to retrieve its `BeanType<?>`.
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- A `Codec<BeanType<?>>` based on this registry. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

With all of this, we can create a registry dispatch codec for beans:

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// Now we can create a codec for bean types
// based on the previously created registry
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// And based on that, here's our registry dispatch codec for beans!
// The first argument is the field name for the bean type.
// When left out, it will default to "type".
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

### Recursive Codecs {#recursive-codecs}

Sometimes it is useful to have a codec that uses _itself_ to decode specific fields, for example when dealing with certain recursive data structures. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Such a codec can be constructed using `Codec#recursive`.

For example, let's try to serialize a singly-linked list. This way of representing lists consists of a bunch of nodes that hold both a value and a reference to the next node in the list. The list is then represented by its first node, and traversing the list is done by following the next node until none remain. Here is a simple implementation of nodes that store integers.

```java
public record ListNode(int value, ListNode next) {}
```

We can't construct a codec for this by ordinary means, because what codec would we use for the `next` field? We would need a `Codec<ListNode>`, which is what we are in the middle of constructing! `Codec#recursive` lets us achieve that using a magic-looking lambda:

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // a name for the codec
  selfCodec -> {
    // Here, `selfCodec` represents the `Codec<ListNode>`, as if it was already constructed
    // This lambda should return the codec we wanted to use from the start,
    // that refers to itself through `selfCodec`
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // the `next` field will be handled recursively with the self-codec
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

A serialized `ListNode` may then look like this:

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

## References {#references}

- A much more comprehensive documentation of Codecs and related APIs can be found at the
  [Unofficial DFU JavaDoc](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec).
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
