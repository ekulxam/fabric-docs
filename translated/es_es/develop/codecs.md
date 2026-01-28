---
title: Codecs
description: Una guía completa para entender y usar el sistema de codecs de Mojang para la serialización y deserialización de objetos.
authors:
  - enjarai
  - Syst3ms
---

Los Codecs es un sistema para la fácil serialización de objetos de Java, el cual viene incluido en la librería de DataFixerUpper (DFU) de Mojang, el cual es incluido en Minecraft. In a modding context they can be used as an alternative
to GSON and Jankson when reading and writing custom JSON files, though they're starting to become
more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Los codecs son usandos en conjunto con otro API de DFU, llamado `DynamicOps` (Operaciones Dinámicas). A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as JSON or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

## Usando Codecs

### Serializando y Deserializando

La manera básica de usar un codec es para serializar y deserializar objetos desde y a un formato en específico.

Ya que algunas clases vanila ya tienen codecs definidos, podemos usarlos como ejemplo. Mojang has also provided us with two dynamic ops classes by default, `JsonOps` and `NbtOps`, which tend to cover most use cases.

Now, let's say we want to serialize a `BlockPos` to JSON and back. We can do this using the codec statically stored at `BlockPos.CODEC` with the `Codec#encodeStart` and `Codec#parse` methods, respectively.

```java
BlockPos pos = new BlockPos(1, 2, 3);

// Serializamos el BlockPos a un JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
```

Cuando usamos codecs, los valores retornados vienen en la forma de un `DataResult` ("Resultado de Dato"). This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, `DataResult#result` will simply return an `Optional` containing our value, while `DataResult#resultOrPartial` also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

Ahora podemos obtener nuestro valor serializado y convertirlo de vuelta a un `BlockPos`:

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

### Codecs Incluidos

As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to `BlockPos`, `BlockState`, `ItemStack`, `Identifier`, `Component`, and regex `Pattern`s. Codecs for Mojang's own classes are usually found as static fields named `CODEC` on the class itself, while most others are kept in the `Codecs` class. It should also be noted that all vanilla registries contain a method to get a `Codec`, for example, you can use `BuiltInRegistries.BLOCK.byNameCodec()` to get a `Codec<Block>` which serializes to the block id and back and a `holderByNameCodec()` to get a `Codec<Holder<Block>>`.

La API de Codec también tiene codecs para tipos de variable primitivos, como `Codec.INT` y `Codec.STRING`. These are available as statics on the `Codec` class, and are usually used as the base for more complex codecs, as explained below.

## Construyendo Codecs

Ahora que hemos visto como usar codecs, veamos como podemos hacer nuestros propios codecs. Suppose we have the following class, and we want to deserialize instances of it from JSON files:

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

Podemos hacer un codec para esta clase integrando varios codecs más pequeños en uno más grande. In this case, we'll need one for each field:

- un `Codec<Integer>`
- un `Codec<Item>`
- un `Codec<List<BlockPos>>`

Podemos obtener el primero usando los codecs primitivos ya mencionados, mediante la clase `Codec`, específicamente `Codec.INT`. While the second one can be obtained from the `BuiltInRegistries.ITEM` registry, which has a `byNameCodec()` method that returns a `Codec<Item>`. Mientras que el segundo se puede obtener mediante el registro `Registries.ITEM`, el cuando tiene un método `getCodec()`, el cual retorna un `Codec<Item>`.

### Listas

`Codec#listOf` puede ser usado para crear una versión lista de cualquier codec:

```java
Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
```

Debe ser recalcado que los codecs creados de esta manera siempre se deserializarán a un `ImmutableList` (lista inmutable o no modificable). If you need a mutable list instead, you can make use of [xmap](#mutually-convertible-types) to convert to one during
deserialization.

### Combinar Codecs para Clases como Records

Ahora que tenemos codecs separados para cada miembro, podemos combinarlos en un solo codec usando un `RecordCodecBuilder`. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Veamos como podemos crear un codec para nuestra clase `CoolBeansClass`:

```java
public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    BuiltInRegistries.ITEM.byNameCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
```

Cada línea en el grupo especifica un codec, un nombre para el miembro, y el método adquiridor. The `Codec#fieldOf` call is used to convert the codec into a [map codec](#mapcodec), and the `forGetter` call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the `apply` call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

También puedes usar `Codec#optionalFieldOf` en este contexto para hacer un miembro opcional, como explicado en la sección de [Miembros Opcionales](#optional-fields).

### MapCodec, sin ser confundido con Codec&lt;Map&gt; {#mapcodec}

Llamar `Codec#fieldOf` convertirá un `Codec<T>` a un `MapCodec<T>`, el cual es una implementación variante, pero no directa de `Codec<T>`. Como su nombre lo indica, los `MapCodec`s garantizan la serialización a una asociación (map) llave a valor, o su equivalente en el `DynamicOps` usado. Algunas funciones pueden requerir una, en vez de un codec regular.

This particular way of creating a `MapCodec` essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a `Codec<BlockPos>` when serialized into JSON would look like this:

```json
[1, 2, 3]
```

Pero cuando es convertido a un `MapCodec<BlockPos>` usando `BlockPos.CODEC.fieldOf("pos")`, se verá así:

```json
{
  "pos": [1, 2, 3]
}
```

While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the [Merging Codecs for Record-like Classes](#merging-codecs-for-record-like-classes) section above, they can also be turned back into regular codecs using `MapCodec#codec`, which will retain the same behavior of
boxing their input value.

#### Optional Fields {#optional-fields}

`Codec#optionalFieldOf` puede ser usado para crear un map codec opcional. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty `Optional` or a specified default value.

```java
// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ZERO);
```

Do note that if the field is present, but the value is invalid, the field fails to deserialize at all if the field value is invalid.

### Constantes, Restricciones, y Composición

#### Unidad

`MapCodec.unitCodec` can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.

```java
Codec<Integer> theMeaningOfCodec = MapCodec.unitCodec(42);
```

#### Rangos Numéricos

`Codec.intRange` and its pals, `Codec.floatRange` and `Codec.doubleRange` can be used to create a codec that only accepts number values within a specified **inclusive** range. Esto se aplica tanto en la serialización y deserialización.

```java
// No puede ser mayor a 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
```

#### Pares

`Codec.pair` combina dos codecs, `Codec<A>` y `Codec<B>`, a un `Codec<Pair<A, B>>`. Keep in mind it only works properly with codecs that serialize to a specific field, such as [converted `MapCodec`s](#mapcodec) or
[record codecs](#merging-codecs-for-record-like-classes).
Ten en cuenta que solo funciona bien con codecs que serializan a un miembro específico, como un [`MapCodec`s convertidos](#mapcodec) o
[codecs de record](#merging-codecs-for-record-like-classes).

Por ejemplo, al correr este código:

```java
// Crea dos codecs empaquetados
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// Combinalos a un codec par
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// Usalo para serializar datos
DataResult<JsonElement> result = pairCodec.encodeStart(JsonOps.INSTANCE, Pair.of(23, true));
```

Will output this JSON:

```json
{
  "i_am_number": 23,
  "this_statement_is_false": true
}
```

#### Cualquiera (Either)

`Codec.either` combina dos codecs, `Codec<A>` y `Codec<B>`, a un `Codec<Either<A, B>>`. The resulting codec will, during deserialization, attempt to use the first codec, and _only if that fails_, attempt to use the second one.
Si el segundo también falla, el error del _segundo_ codec será retornado.

#### Asociaciones (Maps)

Para procesar asociaciones con llaves arbitrarias, como `HashMap`s, se puede usar `Codec.unboundedMap`. Esto nos dá un `Codec<Map<K, V>>`, para un `Codec<K>` y un `Codec<V>` dado. The resulting codec will serialize to a JSON object or the equivalent for the current dynamic ops.

Due to limitations of JSON and NBT, the key codec used _must_ serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as `Identifier.CODEC`. Vea el ejemplo a continuación:

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

### Tipos Mutuamente Convertibles y Tú

#### xmap

Digamos que tenemos dos clases que pueden ser convertidas una a la otra, pero no tienen una relación pariente-hijo. For example, a vanilla `BlockPos` and `Vec3d`. If we have a codec for one, we can use `Codec#xmap` to create a codec for the other by specifying a conversion function for each direction.

`BlockPos` ya tiene un codec, pero pretendamos que no lo tiene. Podemos crear uno para él basándolo en el codec de `Vec3d` así:

```java
Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Convierte Vec3d a Blockpos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Convierte BlockPos a Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// Cuando convertimos una clase ya existente (`X` por ejemplo)
// a tu propia clase (`Y`) de esta manera, puede ser útil tener
// métodos `toX` y un método estático `fromX` en la clase `Y` y usar
// referencias a métodos en tu llamado a `xmap`.
```

#### flatComapMap, comapFlatMap, and flatXMap

`Codec#flatComapMap`, `Codec#comapFlatMap` and `flatXMap` are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla `Identifier`s. While all Identifiers can be turned into strings, not all strings are valid Identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails.
Debido a esto, su codec incluído es en realidad un `comapFlatMap` en `Codec.STRING`, ilustrando como se pueden usar:

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

| Método                  | ¿A -> B siempre válido? | ¿B -> A siempre válido? |
| ----------------------- | ----------------------- | ----------------------- |
| `Codec<A>#xmap`         | Yes                     | Yes                     |
| `Codec<A>#comapFlatMap` | No                      | Yes                     |
| `Codec<A>#flatComapMap` | Yes                     | No                      |
| `Codec<A>#flatXMap`     | No                      | No                      |

### Despachador de Registros

`Codec#dispatch` nos permite definir un registro de codecs y despachar a uno en específico basado en el valor de un miembro en los datos serializados. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

Por ejemplo, digamos que tenemos una interfaz abstracta `Bean`, con dos clases implementadoras: `StringyBean` y `CountingBean`. To serialize these with a registry dispatch, we'll need a few things:

- Codecs separados para cada tipo de bean.
- Una clase `BeanType<T extends Bean>` o un record que represente el tipo de bean, y que pueda retornar un codec para él.
- Una función en `Bean` para obtener su `BeanType<?>`.
- A map or registry to map `Identifier`s to `BeanType<?>`s.
- Un `Codec<BeanType<?>>` basado en este registro. If you use a `net.minecraft.core.Registry`, one can be easily made
  using `Registry#getCodec`.

Con todo esto, puedes crear un despacho de registros para beans:

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/Bean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanType.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/StringyBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/CountingBean.java)

@[code transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/codec/BeanTypes.java)

```java
// Ahora podemos crear un codec para los tipos de bean
// basado en el registro previamente creado
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// ¡Y basado en esto, aquí esta nuestro codec de despacho de registros para beans!
// El primer argumento es el nombre del miembro para el tipo de bean.
// Cuando no se especifica, se usa el valor por defecto "type".
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

### Codecs Recursivos

A veces es útil tener un codec que se use a _sí mismo_ para decodificar ciertos miembros, por ejemplo cuando estamos lidiando con ciertas estructuras de datos recursivas. In vanilla code, this is used for `Component` objects, which may store other `Component`s as children. Tal codec puede ser construido usando `Codec#recursive`.

Por ejemplo, tratemos de serializar una lista enlazada. Esta manera de representar listas consiste en varios nodos que contienen un valor y una referencia al siguiente nodo en la lista. La lista es entonces representada mediante el primer nodo, y para caminar por la lista se sigue el siguiente nodo hasta que no quede ninguno. Aquí está una implementación simple de los nodos que guardan números enteros.

```java
public record ListNode(int value, ListNode next) {}
```

No podemos construir un codec para esto mediante métodos ordinarios, porque ¿qué codec usaríamos para el miembro de `next`? ¡Tendríamos que usar un `Codec<ListNode>`, que es lo que estamos construyendo actualmente! `Codec#recursive` nos permite lograr eso con una expresión lambda mágica:

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

Un `ListNode` serializado se podría ver algo así:

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

## References {#references}

- Puedes encontrar una documentación más completa sobre codecs y los APIs relacionados en los [Javadocs de DFU no oficiales](https://kvverti.github.io/Documented-DataFixerUpper/snapshot/com/mojang/serialization/Codec).
- The general structure of this guide was heavily inspired by the
  [Forge Community Wiki's page on Codecs](https://forge.gemwire.uk/wiki/Codecs), a more Forge-specific take on the same topic.
