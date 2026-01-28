---
title: Codecs
description: Un guide complet pour comprendre et utiliser le système de codecs de Mojang pour la sérialisation et désérialisation d'objets.
authors:
  - enjarai
  - Syst3ms
---

Les codecs sont un système de sérialisation facile d'objets Java, et est inclus dans la librairie DataFixerUpper (DFU) de Mojang, qui vient avec Minecraft. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Les codecs sont utilisés en tandem avec une autre API de DFU, `DynamicOps`. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## Utilisation des codecs {#using-codecs}

### Sérialisation et désérialisation {#serializing-and-deserializing}

En premier lieu, un codec est utilisé pour sérialiser et désérialiser des objets vers et à partir d'un format donné.

Puisque quelques classes vanilla définissent déjà des codecs, on peut prendre ceux-là en exemple. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// Serialisation de la BlockPos en JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

Quand on manipule un codec, les valeurs renvoyées prennent la forme d'un `DataResult` (litt. 'résultat de données'). This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

Prenons donc notre valeur sérialisée et transformons-la en `BlockPos` :

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

### Codecs intégrés {#built-in-codecs}

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

L'API des codecs contient déjà des codecs pour des types primitifs, comme `Codec.INT` et `Codec.STRING`. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## Construction de codecs {#building-codecs}

Maintenant qu'on sait utiliser les codecs, regardons comment construire le nôtre. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

On peut créer un codec pour cette classe par assemblage de codecs plus simples. In this case, we'll need one for each field:

- un `Codec<Integer>`
- un `Codec<Item>`
- un `Codec<List<BlockPos>>`

Le premier est un des codecs primitifs de la classe `Codec` mentionnés plus haut, plus précisément `Codec.INT`. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. Il n'y a pas de codec par défaut pour `List<BlockPos>`, mais on peut en créer un à partir de `BlockPos.CODEC`.

### Listes {#lists}

`Codec#listOf` crée une version liste de n'importe quel codec :

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

À noter que les codecs créés ainsi se désérialisent toujours en `ImmutableList`. If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### Fusion de codecs pour les classes similaires à des records {#merging-codecs-for-record-like-classes}

Avec les codecs pour chaque champ à notre disposition, on peut les combiner en un codec pour la classe avec un `RecordCodecBuilder`. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Voyons voir comment créer un codec pour notre `CoolBeansClass` :

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

Chaque argument à la méthode `group` spécifie un codec, un nom de champ, et une méthode getter. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

On peut également utiliser `Codec#optionalFieldOf` dans ce contexte pour rendre un champ facultatif, comme expliqué dans la section [Champs facultatifs](#optional-fields).

### MapCodec, et non pas Codec&lt;Map&gt; {#mapcodec}

`Codec#fieldOf` transforme un `Codec<T>` en `MapCodec<T>` qui est une variante de `Codec<T>`, sans en être une implémentation directe. Comme leur nom peut le suggérer, les codecs map sérialisent leurs valeurs dans en maps clés-valeurs, ou plutôt leur équivalent dans les `DynamicOps` utilisées. Certaines fonctions peuvent en nécessiter un au lieu d'un codec normal.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

Mais quand transformé en `MapCodec<BlockPos>` via `BlockPos.CODEC.fieldOf("pos")`, donnerait ceci :

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### Champs facultatifs {#optional-fields}

`Codec#optionalFieldOf` permet de créer un codec map facultatif. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### Constantes, contraintes et composition {#constants-constraints-composition}

#### Constante {#unit}

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### Intervalles numériques {#numeric-ranges}

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. Cela vaut et pour la sérialisation, et pour la désérialisation.

```java
// Ne peut excéder 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Paire {#pair}

`Codec.pair` fusionne deux codecs `Codec<A>` et `Codec<B>` en un `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
Il faut garder à l'esprit que cela ne marche correctement qu'avec des codecs qui sérialisent un champ précis, comme [des codec maps convertis](#mapcodec) ou [des codecs records](#merging-codecs-for-record-like-classes).

Par exemple, l'exécution de ce code :

```java
// Création de deux codecs encapsulés distincts
Codec<Integer> firstCodec = Codec.INT.fieldOf("un_nombre").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("cette_phrase_est_fausse").codec();

// Fusion en un codec paire
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// Utilisation pour sérialiser des données
DataResult<JsonElement> result = pairCodec.encodeStart(JsonOps.INSTANCE, Pair.of(23, true));
```

Will output this JSON:

```json
{
  "un_nombre": 23,
  "cette_phrase_est_fausse": true
}
```

#### Deux cas {#either}

`Codec.either` fusionne deux codecs `Codec<A>` et `Codec<B>` en un `Codec<Either<A,B>>`. The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
Si le second échoue à son tour, l'erreur du _second_ codec sera renvoyée.

#### Collections {#maps}

Pour gérer des `Map`s avec des clés arbitraires commes des `HashMap`s, `Codec.unboundedMap` peut être utilisé. Celle-ci renvoie un `Codec<Map<K, V>>`, étant donnés un `Codec<K>` et un `Codec<V>`. The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. Voir l'exemple suivant :

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
  "example:nombre": 23,
  "example:le_nombre_plus_cool": 42
}
```

As you can see, this works because `Identifier.CODEC` serializes directly to a string value. A similar effect can be achieved for simple objects that don't serialize to strings by using [xmap & friends](#mutually-convertible-types) to convert them.

### Les types interconvertibles {#mutually-convertible-types}

#### `xmap` {#xmap}

Supposons qu'on ait deux classes qui peuvent être converties entre elles, mais sans relation parent-enfant. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` possède déjà un codec, mais imaginons que non. On peut en créer un à partir du codec de `Vec3d` comme ceci :

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Conversion de Vec3d en BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Conversion de BlockPos en Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// Si vous convertissez une classe pré-existante (`X` par exemple)
// en une classe à vous (`Y`) ainsi, il peut être pratique
// d'ajouter des méthodes `toX` and `fromX` (statique) à `Y`
// et d'utiliser des références de méthodes dans l'appel à `xmap`.
```

#### flatComapMap, comapFlatMap, and flatXMap

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
Par conséquent, son codec intégré est en réalité un `comapFlatMap` sur `Codec.STRING`, ce qui illustre bien son utilisation :

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

| Méthode                 | A -> B toujours valide ? | B -> A toujours valide ? |
| ----------------------- | ------------------------ | ------------------------ |
| `Codec<A>#xmap`         | Yes                      | Yes                      |
| `Codec<A>#comapFlatMap` | Non                      | Yes                      |
| `Codec<A>#flatComapMap` | Yes                      | Non                      |
| `Codec<A>#flatXMap`     | Non                      | Non                      |

### Registry Dispatch {#registry-dispatch}

Si on définit un registre de codecs, `Codec#dispatch` permet d'utiliser l'un des codecs selon la valeur d'un champ dans les données sérialisées. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

Par exemple, imaginons une interface abstraite `Bean` avec deux classes qui l'implémentent : `StringyBean` et `CountingBean`. To serialize these with a registry dispatch, we'll need a few things:

- Des codecs distincts pour chaque type de `Bean`.
- Une classe/record `BeanType<T extends Bean>` qui représente le type de blob, et peut fournir le codec associé.
- Une méthode de `Bean` qui renvoie son `BeanType<?>`.
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- Un `Codec<TypeBlob<?>>` à partir de ce registre. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

Une fois tout ceci fait, on peut créer un codec de répartition par registre pour les beans :

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// On peut créer un codec pour les types de bean
// grâce au registre précédemment créé
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// Et à partir de ça, un codec de répartition par registre pour beans !
// Le premier argument est le nom du champ correspondant au type.
// Si omis, il sera égal à "type".
Codec<Bean> beanCodec = beanTypeCodec.dispatch("type", Bean::getType, BeanType::getCodec);
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

### Codecs récursifs {#recursive-codecs}

Il est parfois utile d'avoir un codec qui s'utilise _soi-même_ pour décoder certains champs, par exemple avec certaines structures de données récursives. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Un tel codec peut être construit grâce à `Codec#recursive`.

À titre d'exemple, essayons de sérialiser une liste simplement chaînée. Cette manière de représenter une liste consiste en des nœuds qui contiennent et une valeur, et une référence au prochain nœud de la liste. La liste est alors représentée par son premier nœud, et pour la parcourir, il suffit de continuer à regarder le nœud suivant juste qu'à ce qu'il n'en existe plus. Voici une implémentation simple de nœuds qui stockent des entiers.

```java
public record ListNode(int value, ListNode next) {}
```

Il est impossible de construire un codec comme d'habitude, puisque quel codec utiliserait-on pour le champ `next` ? Il faudrait un `Codec<ListNode>`, ce qui est précisément ce qu'on veut obtenir ! `Codec#recursive` permet de le faire au moyen d'un lambda magique en apparence :

```java
Codec<ListNode> codec = Codec.recursive(
  "ListNode", // un nom pour le codec
  selfCodec -> {
    // Ici, `selfCodec` représente le `Codec<ListNode>`, comme s'il était déjà construit
    // Ce lambda doit renvoyer le codec qu'on aurait voulu utiliser depuis le départ,
    // qui se réfère à lui-même via `selfCodec`
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // le champ `next` sera récursivement traité grâce à l'auto-codec
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
);
```

Un `ListNode` sérialisé pourrait alors ressembler à ceci :

```json
{
  "value": 2,
  "next": {
    "value": 3,
    "next" : {
      "value": 5
    }
  }
}
```

## Références {#references}

- Il y a une documentation bien plus exhaustive des codecs et des APIs attenantes dans la [JavaDoc DFU non-officielle](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec).
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
