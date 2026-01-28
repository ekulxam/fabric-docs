---
title: Componenti di Dati Personalizzate
description: Impara come si aggiungono dati personalizzati ai tuoi oggetti usando il nuovo sistema di componenti di 1.20.5.
authors:
  - Romejanic
---

Più i tuoi oggetti crescono in complessità, più troverai la necessità di memorizzare dati personalizzati associati con ogni oggetto. Il gioco permette di memorizzare dati persistenti in un `ItemStack`, e a partire da 1.20.5 il modo di fare ciò è usare le **Componenti di Dati**.

Le Componenti di Dati sostituiscono i dati NBT di versioni precedenti, con tipi di dati strutturati che possono essere applicati a un `ItemStack` per memorizzare dati su quello stack. Le componenti di dati sfruttano namespace, il che significa che possiamo implementare le nostre componenti di dati per memorizzare dati personalizzati su un `ItemStack` e accederci successivamente. Una lista completa delle componenti di dati vanilla si trova su questa [pagina della Minecraft Wiki](https://minecraft.wiki/w/Data_component_format#List_of_components).

Assieme alla registrazione delle componenti personalizzate, questa pagina tratta l'uso generale dell'API delle componenti, il che si applica anche alle componenti vanilla. You can see and access the definitions of all vanilla components in the `DataComponents` class.

## Registrare una Componente {#registering-a-component}

As with anything else in your mod you will need to register your custom component using a `DataComponentType`. Questo tipo di componente accetta un parametro generico contenente il tipo del valore della tua componente. Ci concentreremo su questo più in basso quando tratteremo le componenti [basilari](#basic-data-components) e [avanzate](#advanced-data-components).

Scegli sensibilmente una classe in cui mettere ciò. Per questo esempio creeremo un nuovo package chiamato `component` e una classe che conterrà tutti i tipi delle nostre componenti chiamate `ModComponents`. Assicurati di chiamare `ModComponents.initialize()` nel tuo [inizializzatore della mod](../getting-started/project-structure#entrypoints).

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

Questo è il modello generico per registrare un tipo di componente:

```java
public static final DataComponentType<?> MY_COMPONENT_TYPE = Registry.register(
    BuiltInRegistries.DATA_COMPONENT_TYPE,
    Identifier.fromNamespaceAndPath(ExampleMod.MOD_ID, "my_component"),
    DataComponentType.<?>builder().persistent(null).build()
);
```

Ci sono un paio di cose qui da far notare. Nelle linee prima e quarta, noti un `?`. Questo sarà sostituito con il tipo del valore della tua componente. Lo compileremo presto.

Secondly, you must provide an `Identifier` containing the intended ID of your component. Questo avrà il namespace con l'ID della tua mod.

Lastly, we have a `DataComponentType.Builder` that creates the actual `DataComponentType` instance that's being registered. Questo contiene un altro dettaglio cruciale che dovremmo analizzare: il `Codec` della tua componente. Questo per ora è `null` ma presto dobbiamo scriverlo.

## Componenti di Dati Basilari {#basic-data-components}

Le componenti di dati basilari (come `minecraft:damage`) consiste di un valore di dati singolo, come un `int`, `float`, `boolean` o `String`.

Per questo esempio, creiamo un valore `Integer` che traccierà quante volte il giocatore ha cliccato con il tasto destro mente teneva il nostro oggetto. Aggiorniamo la registrazione della nostra componente alla seguente:

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

Puoi ora notare che abbiamo passato `<Integer>` come nostro tipo generico, indicando che questa componente sarà memorizzata come un valore `int` singolo. For our codec, we are using the provided `ExtraCodecs.POSITIVE_INT` codec. Possiamo cavarcela usando codec basilari per componenti semplici come questa, ma scenari più complessi potrebbero richiedere un codec personalizzato (questo sarà trattato tra poco).

Se avviassi il gioco, dovresti poter inserire un comando come questo:

![Comando /give che mostra la componente personalizzata](/assets/develop/items/custom_component_0.png)

Quando esegui il comando, dovresti ricevere l'oggetto contenente la componente. Tuttavia, non possiamo ancora sfruttare la componente per fare qualcosa di utile. Iniziamo leggendo il valore della componente in modo da poterlo vedere.

## Leggere il Valore della Componente {#reading-component-value}

Aggiungiamo un nuovo oggetto che aumenterà il contatore ogni volta che viene cliccato con il tasto destro. Dovresti leggere la pagina [Interazioni tra Oggetti Personalizzate](./custom-item-interactions) che tratterà delle tecniche usate in questa guida.

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

Ricorda come sempre di registrare l'oggetto nella tua classe `ModItems`.

```java
public static final Item COUNTER = register("counter", CounterItem::new, new Item.Properties());
```

Aggiungeremo del codice del tooltip per mostrare il valore corrente del contatore di clic quando passiamo il mouse sopra al nostro oggetto nell'inventario. Possiamo usare il metodo `get()` sul nostro `ItemStack` per ottenere il valore della nostra componente, così:

```java
int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
```

Questo restituirà il valore corrente della componente nel tipo che abbiamo definito quando abbiamo registrato la nostra componente. Possiamousare questo valore per aggiungere una voce al tooltip. Add this line to the `appendHoverText` method in the `CounterItem` class:

```java
public void appendHoverText(ItemStack stack, TooltipContext context, TooltipDisplay displayComponent, Consumer<Component> textConsumer, TooltipFlag type) {
  int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
  textConsumer.accept(Component.translatable("item.example-mod.counter.info", count).withStyle(ChatFormatting.GOLD));
}
```

Non dimenticarti di aggiornare il tuo file di lingua (`/assets/example-mod/lang/en_us.json`) e aggiungere queste due righe:

```json
{
  "item.example-mod.counter": "Counter",
  "item.example-mod.counter.info": "Used %1$s times"
}
```

Avvia il gioco e esegui questo comando per darti un nuovo oggetto Counter con un conto di 5.

```mcfunction
/give @p example-mod:counter[example-mod:click_count=5]
```

Quando passi il mouse sopra a questo oggetto nell'inventario, dovresti notare il conto nel tooltip!

![Tooltip che mostra "Used 5 times"](/assets/develop/items/custom_component_1.png)

Tuttavia, se ti dai un nuovo oggetto Counter _senza_ la componente personalizzata, il gioco crasherà quando passi il mouse sull'oggetto nel suo inventario. Dovresti vedere un errore come il seguente nel report di crash:

```log
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "net.minecraft.world.item.ItemStack.get(net.minecraft.core.component.DataComponentType)" is null
        at com.example.docs.item.custom.CounterItem.appendHoverText(LightningStick.java:45)
        at net.minecraft.world.item.ItemStack.getTooltipLines(ItemStack.java:767)
```

Come ci aspettavamo, poiché l'`ItemStack` non contiene per ora un'istanza della nostra componente personalizzata, chiamare `stack.get()` con il tipo della nostra componente restituirà `null`.

Ci sono tre soluzioni a questo problema.

### Impostare un Valore Predefinito della Componente {#setting-default-value}

When you register your item and pass a `Item.Properties` object to your item constructor, you can also provide a list of default components that are applied to all new items. Tornando alla nostra classe `ModItems`, dove registriamo il `CounterItem`, possiamo aggiungere un valore predefinito alla nostra componente. Aggiungi questo così i nuovi oggetti mostreranno un conto di `0`.

@[code transcludeWith=::\_13](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Quando un nuovo oggetto viene creato, applicherà automaticamente la nostra componente personalizzata con il valore dato.

::: warning

Using commands, it is possible to remove a default component from an `ItemStack`. You should refer to the next two sections to properly handle a scenario where the component is not present on your item.

:::

### Reading with a Default Value {#reading-default-value}

In addition, when reading the component value, we can use the `getOrDefault()` method on our `ItemStack` object to return a specified default value if the component is not present on the stack. This will safeguard against any errors resulting from a missing component. We can adjust our tooltip code like so:

```java
int count = stack.getOrDefault(ModComponents.CLICK_COUNT_COMPONENT, 0);
```

As you can see, this method takes two arguments: our component type like before, and a default value to return if the component is not present.

### Checking if a Component Exists {#checking-if-component-exists}

You can also check for the existence of a specific component on an `ItemStack` using the `has()` method. This takes the component type as an argument and returns `true` or `false` depending on whether the stack contains that component.

```java
boolean exists = stack.has(ModComponents.CLICK_COUNT_COMPONENT);
```

### Fixing the Error {#fixing-the-error}

We're going to go with the third option. So along with adding a default component value, we'll also check if the component is present on the stack and only show the tooltip if it is.

@[code transcludeWith=::3](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

Start the game again and hover over the item without the component, you should see that it displays "Used 0 times" and no longer crashes the game.

![Tooltip showing "Used 0 times"](/assets/develop/items/custom_component_2.png)

Try giving yourself a Counter with our custom component removed. You can use this command to do so:

```mcfunction
/give @p example-mod:counter[!example-mod:click_count]
```

When hovering over this item, the tooltip should be missing.

![Counter item with no tooltip](/assets/develop/items/custom_component_7.png)

## Updating Component Value {#setting-component-value}

Now let's try updating our component value. We're going to increase the click count each time we use our Counter item. To change the value of a component on an `ItemStack` we use the `set()` method like so:

```java
stack.set(ModComponents.CLICK_COUNT_COMPONENT, newValue);
```

This takes our component type and the value we want to set it to. In this case it will be our new click count. This method also returns the old value of the component (if one is present) which may be useful in some situations. For example:

```java
int oldValue = stack.set(ModComponents.CLICK_COUNT_COMPONENT, newValue);
```

Let's set up a new `use()` method to read the old click count, increase it by one, and then set the updated click count.

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

Now try starting the game and right-clicking with the Counter item in your hand. If you open up your inventory and look at the item again you should see that the usage number has gone up by the amount of times you've clicked it.

![Tooltip showing "Used 8 times"](/assets/develop/items/custom_component_3.png)

## Removing Component Value {#removing-component-value}

You can also remove a component from your `ItemStack` if it is no longer needed. This is done by using the `remove()` method, which takes in your component type.

```java
stack.remove(ModComponents.CLICK_COUNT_COMPONENT);
```

This method also returns the value of the component before being removed, so you can also use it as follows:

```java
int oldCount = stack.remove(ModComponents.CLICK_COUNT_COMPONENT);
```

## Advanced Data Components {#advanced-data-components}

You may need to store multiple attributes in a single component. As a vanilla example, the `minecraft:food` component stores several values related to food, such as `nutrition`, `saturation`, `eat_seconds` and more. In this guide we'll refer to them as "composite" components.

For composite components, you must create a `record` class to store the data. This is the type we'll register in our component type and what we'll read and write when interacting with an `ItemStack`. Start by making a new record class in the `component` package we made earlier.

```java
public record MyCustomComponent() {
}
```

Notice that there's a set of brackets after the class name. This is where we define the list of properties we want our component to have. Let's add a float and a boolean called `temperature` and `burnt` respectively.

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/component/MyCustomComponent.java)

Since we are defining a custom data structure, there won't be a pre-existing `Codec` for our use case like with the [basic component](#basic-data-components). This means we're going to have to construct our own codec. Let's define one in our record class using a `RecordCodecBuilder` which we can reference once we register the component. For more details on using a `RecordCodecBuilder` you can refer to [this section of the Codecs page](../codecs#merging-codecs-for-record-like-classes).

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/component/MyCustomComponent.java)

You can see that we are defining a list of custom fields based on the primitive `Codec` types. However, we are also telling it what our fields are called using `fieldOf()`, and then using `forGetter()` to tell the game which attribute of our record to populate.

You can also define optional fields by using `optionalFieldOf()` and passing a default value as the second argument. Any fields not marked optional will be required when setting the component using `/give` so make sure you mark any optional arguments as such when creating your codec.

Finally, we call `apply()` and pass our record's constructor. For more details on how to construct codecs and more advanced use cases, be sure to read the [Codecs](../codecs) page.

Registering a composite component is similar to before. We just pass our record class as the generic type, and our custom `Codec` to the `codec()` method.

@[code transcludeWith=::3](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

Now start the game. Using the `/give` command, try applying the component. Composite component values are passed as an object enclosed with `{}`. If you put blank curly brackets, you'll see an error telling you that the required key `temperature` is missing.

![Give command showing missing key "temperature"](/assets/develop/items/custom_component_4.png)

Add a temperature value to the object using the syntax `temperature:8.2`. You can also optionally pass a value for `burnt` using the same syntax but either `true` or `false`. You should now see that the command is valid, and can give you an item containing the component.

![Valid give command showing both properties](/assets/develop/items/custom_component_5.png)

### Getting, Setting and Removing Advanced Components {#getting-setting-removing-advanced-comps}

Using the component in code is the same as before. Using `stack.get()` will return an instance of your `record` class, which you can then use to read the values. Since records are read-only, you will need to create a new instance of your record to update the values.

```java
// read values of component
MyCustomComponent comp = stack.get(ModComponents.MY_CUSTOM_COMPONENT);
float temp = comp.temperature();
boolean burnt = comp.burnt();

// set new component values
stack.set(ModComponents.MY_CUSTOM_COMPONENT, new MyCustomComponent(8.4f, true));

// check for component
if (stack.contains(ModComponents.MY_CUSTOM_COMPONENT)) {
    // do something
}

// remove component
stack.remove(ModComponents.MY_CUSTOM_COMPONENT);
```

You can also set a default value for a composite component by passing a component object to your `Item.Properties`. For example:

```java
public static final Item COUNTER = register(
    "counter",
    CounterItem::new,
    new Item.Properties().component(ModComponents.MY_CUSTOM_COMPONENT, new MyCustomComponent(0.0f, false))
);
```

Now you can store custom data on an `ItemStack`. Use responsibly!

![Item showing tooltips for click count, temperature and burnt](/assets/develop/items/custom_component_6.png)
