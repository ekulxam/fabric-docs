---
title: Codec
description: Una guida esaustiva per la comprensione e l'uso del sistema di codec di Mojang per serializzare e deserializzare gli oggetti.
authors:
  - enjarai
  - Syst3ms
---

Un codec è un sistema per serializzare facilmente oggetti Java, ed è incluso nella libreria DataFixerUpper (DFU) di Mojang, che è inclusa in Minecraft. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

I Codec vengono usati assieme a un'altra API da DFU, `DynamicOps`. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## Usare i Codec {#using-codecs}

### Serializzazione e Deserializzazione {#serializing-and-deserializing}

L'uso principale di un codec è serializzare e deserializzare oggetti da e a un formato specifico.

Poiché alcune classi vanilla hanno già dei codec definiti, possiamo usare quelli come un esempio. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// Serializza il BlockPos a un JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

Quando si usa un codec, i valori sono restituiti come un `DataResult`. This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

Quindi prendiamo il nostro valore serializzato e ritrasformiamolo nuovamente in un `BlockPos`:

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

### Codec Predefiniti {#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

L'API stessa dei Codec contiene anche alcuni codec per tipi primitivi, come `Codec.INT` e `Codec.STRING`. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## Costruire Codec {#building-codecs}

Ora che abbiamo visto come usare i codec, vediamo come possiamo costruircene di nostri. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

Possiamo creare un codec per questa classe mettendo insieme tanti codec più piccoli per formarne uno più grande. In this case, we'll need one for each field:

- un `Codec<Integer>`
- un `Codec<Item>`
- un `Codec<List<BlockPos>>`

Possiamo ottenere il primo dal codec primitivo nella classe `Codec` menzionato in precedenza, nello specifico `Codec.INT`. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. Non abbiamo un codec predefinito per `List<BlockPos>`, ma possiamo crearne uno a partire da `BlockPos.CODEC`.

### Liste {#lists}

`Codec#listOf` può essere usato per creare una versione lista di qualsiasi codec:

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

Bisogna sottolineare che i codec creati così verranno sempre deserializzati a un'`ImmutableList`. If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### Unire i Codec per Classi Simili ai Record {#merging-codecs-for-record-like-classes}

Ora che abbiamo codec separati per ciascun attributo, possiamo combinarli a formare un singolo codec per la nostra classe usando un `RecordCodecBuilder`. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Diamo un'occhiata a come creare un codec per la nostra `CoolBeansClass`:

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

Ogni linea nel gruppo specifica un codec, il nome di un attributo, e un metodo getter. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

Puoi anche usare `Codec#optionalFieldOf` in questo contesto per rendere un attributo opzionale, come spiegato nella sezione [Attributi Opzionali](#attributi-opzionali).

### MapCodec, Da Non Confondere Con Codec&lt;Map&gt; {#mapcodec}

La chiamata a `Codec#fieldOf` convertirà un `Codec<T>` in un `MapCodec<T>`, che è una variante, ma non una diretta implementazione di `Codec<T>`. I `MapCodec`, come suggerisce il loro nome garantiscono la serializzazione a una mappa chiave-valore, o al suo equivalente nella `DynamicOps` usata. Alcune funzioni ne potrebbero richiedere uno invece di un codec normale.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

Ma quando viene convertito in un `MapCodec<BlockPos>` usando `BlockPos.CODEC.fieldOf("pos")`, esso ha il seguente aspetto:

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### Attributi Opzionali {#optional-fields}

`Codec#optionalFieldOf` può essere usato per create una mappa codec opzionale. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### Costanti, Vincoli, e Composizione {#constants-constraints-composition}

#### Unità {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### Intervalli Numerici {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. Questo si applica sia alla serializzazione sia alla deserializzazione.

```java
// Non può essere superiore a 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Coppia {#pair}

`Codec.pair` unisce due codec, `Codec<A>` e `Codec<B>`, in un `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
Il codec risultante serializzerà a una mappa contenente gli attributi di entrambi i codec usati.

Per esempio, eseguire questo codice:

```java
// Crea due codec incapsulati separati
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// Uniscili in un codec coppia
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// Usalo per serializzare i dati
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

`Codec.either` unisce due codec, `Codec<A>` e `Codec<B>`, in un `Codec<Either<A, B>>`. The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
Se anche il secondo fallisse, l'errore del _secondo_ codec verrà restituito.

#### Mappe {#maps}

Per gestire mappe con chiavi arbitrarie, come `HashMap`, `Codec.unboundedMap` può essere usato. Questo restituisce un `Codec<Map<K, V>>` per un dato `Codec<K>` e `Codec<V>`. The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. Vedi l'esempio sotto:

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

### Tipi Convertibili Mutualmente {#mutually-convertible-types}

#### `xmap` {#xmap}

Immagina di avere due classi che possono essere convertite l'una nell'altra e viceversa, ma che non hanno un legame gerarchico genitore-figlio. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` ha già un codec, ma facciamo finta che non ce l'abbia. Possiamo creargliene uno basandolo sul codec per `Vec3d` così:

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Converti Vec3d a BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Converti BlockPos a Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// Quando converti una classe esistente (per esempio `X`)
// alla tua classe personalizzata (`Y`) in questo modo,
// potrebbe essere comodo aggiungere i metodi `toX` e
// `fromX` statico ad `Y` e usare riferimenti ai metodi
// nella tua chiamata ad `xmap`.
```

#### flatComapMap, comapFlatMap, e flatXMap {#flatcomapmap-comapflatmap-flatxmap}

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
Per questo, il suo codec predefinito è in realtà una `comapFlatMap` su `Codec.STRING`, che illustra bene come usarla:

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

| Metodo                  | A -> B è sempre valido? | B -> A è sempre valido? |
| ----------------------- | ----------------------- | ----------------------- |
| `Codec<A>#xmap`         | Yes                     | Yes                     |
| `Codec<A>#comapFlatMap` | No                      | Yes                     |
| `Codec<A>#flatComapMap` | Yes                     | No                      |
| `Codec<A>#flatXMap`     | No                      | No                      |

### Dispatch della Registry {#registry-dispatch}

`Codec#dispatch` ci permette di definire una registry di codec e di fare dispatch ad uno di essi in base al valore di un attributo nei dati serializzati. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

Per esempio, immaginiamo di avere un'interfaccia astratta `Bean` con due classi che la implementano: `StringyBean` e `CountingBean`. To serialize these with a registry dispatch, we'll need a few things:

- Codec separati per ogni tipo di fagiolo.
- Una classe o un record `BeanType<T extends Bean>` che rappresenta il tipo di fagiolo, e che può restituire il codec per esso.
- Una funzione in `Bean` per ottenere il suo `BeanType<?>`.
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- Un `Codec<BeanType<?>>` basato su questa registry. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

Con tutto questo, possiamo creare un codec di dispatch di registry per i fagioli:

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// Ora possiamo creare un codec per i tipi di fagioli
// in base alla registry creata in precedenza
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// E in base a quello, ecco il nostro codec di dispatch della registry per i fagioli!
// Il primo parametro e il nome dell'attributo per il tipo di fagiolo.
// Se lasciato vuoto, assumerà "type" come valore predefinito.
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

### Codec Ricorsivi {#recursive-codecs}

A volte è utile avere un codec che usa _sé stesso_ per decodificare attributi specifici, per esempio quando si gestiscono certe strutture dati ricorsive. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Un codec del genere può essere costruito usando `Codec#recursive`.

Per esempio, proviamo a serializzare una lista concatenata singolarmente. Questo metodo di rappresentare le liste consiste di una serie di nodi che contengono sia un valore sia un riferimento al nodo successivo nella lista. La lista è poi rappresentata dal suo primo nodo, e per attraversare la lista si segue il prossimo nodo finché non ce ne sono più. Ecco una semplice implementazione di nodi che contengono interi.

```java
public record ListNode(int value, ListNode next) {}
```

Non possiamo costruire un codec per questo come si fa di solito: quale codec useremmo per l'attributo `next`? Avremmo bisogno di un `Codec<ListNode>`, che è ciò che stiamo costruendo proprio ora! `Codec#recursive` ci permette di fare ciò usando una lambda che sembra magia:

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // un nome per il codec
  selfCodec -> {
    // Qui, `selfCodec` rappresenta il `Codec<ListNode>`, come se fosse già costruito
    // Questa lambda dovrebbe restituire il codec che volevamo usare dall'inizio,
    // che punta a sé stesso attraverso `selfCodec`
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // l'attributo `next` sarà gestito ricorsivamente con il self-codec
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

Un `ListNode` serializzato potrebbe avere questo aspetto:

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

## Riferimenti {#references}

- Una documentazione molto più dettagliata sui Codec e sulle relative API può essere trovata presso la [JavaDoc non Ufficiale di DFU](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec).
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
