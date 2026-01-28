---
title: Пользовательские компоненты данных
description: Узнайте, как добавлять пользовательские данные к вашим товарам, используя новую систему компонентов 1.20.5.
authors:
  - Romejanic
---

По мере того как ваши элементы становятся более сложными, вам может потребоваться хранить специальные данные, связанные с каждым элементом. Игра позволяет хранить постоянные данные в `ItemStack`, и начиная с версии 1.20.5 мы делаем это с помощью **Компонентов данных**.

Компоненты данных заменяют данные NBT из предыдущих версий структурированными типами данных, которые можно применять к `ItemStack` для хранения постоянных данных об этом стеке. Компоненты данных имеют пространство имен, что означает, что мы можем реализовать собственные компоненты данных для хранения пользовательских данных о `ItemStack` и доступа к ним позже. Полный список компонентов данных vanilla можно найти на этой [странице вики Minecraft](https://minecraft.wiki/w/Data_component_format#List_of_components).

Наряду с регистрацией пользовательских компонентов на этой странице рассматривается общее использование API компонентов, которое также применимо к ванильным компонентам. You can see and access the definitions of all vanilla components in the `DataComponents` class.

## Регистрация компонента {#registering-a-component}

As with anything else in your mod you will need to register your custom component using a `DataComponentType`. Этот тип компонента принимает универсальный аргумент, содержащий тип значения вашего компонента. Мы более подробно рассмотрим это далее, когда будем рассматривать [базовые](#basic-data-components) и [расширенные](#advanced-data-components) компоненты.

Выберите подходящий класс для размещения этого кода. В этом примере мы создадим новый пакет с именем `component` и класс, содержащий все типы наших компонентов, с именем `ModComponents`. Убедитесь, что вы вызвали `ModComponents.initialize()` в инициализаторе вашего [мода](../getting-started/project-structure#entrypoints).

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

Это базовый шаблон для регистрации типа компонента:

```java
public static final DataComponentType<?> MY_COMPONENT_TYPE = Registry.register(
    BuiltInRegistries.DATA_COMPONENT_TYPE,
    Identifier.fromNamespaceAndPath(ExampleMod.MOD_ID, "my_component"),
    DataComponentType.<?>builder().persistent(null).build()
);
```

Здесь есть несколько вещей, на которые стоит обратить внимание. В первой и четвертой строках вы можете увидеть `?`. Он будет заменен типом значения вашего компонента. Мы скоро это заполним.

Secondly, you must provide an `Identifier` containing the intended ID of your component. Это пространство имен с идентификатором вашего мода.

Lastly, we have a `DataComponentType.Builder` that creates the actual `DataComponentType` instance that's being registered. Здесь содержится еще одна важная деталь, которую нам нужно будет обсудить: `Codec` вашего компонента. В настоящее время это поле пустое, но мы скоро его заполним.

## Базовые компоненты данных {#basic-data-components}

Базовые компоненты данных (например, `minecraft:damage`) состоят из одного значения данных, например `int`, `float`, `boolean` или `String`.

В качестве примера давайте создадим значение `Integer`, которое будет отслеживать, сколько раз игрок щелкнул правой кнопкой мыши, удерживая наш предмет. Давайте обновим регистрацию нашего компонента следующим образом:

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

Вы можете видеть, что теперь мы передаем `<Integer>` в качестве нашего универсального типа, указывая, что этот компонент будет сохранен как одно значение `int`. For our codec, we are using the provided `ExtraCodecs.POSITIVE_INT` codec. Для таких простых компонентов, как этот, можно обойтись использованием базовых кодеков, но для более сложных сценариев может потребоваться специальный кодек (об этом мы кратко поговорим позже).

Если вы запустите игру, вы сможете ввести такую ​​команду:

![/give command showing the custom component](/assets/develop/items/custom_component_0.png)

При выполнении команды вы должны получить элемент, содержащий компонент. Однако в настоящее время мы не используем наш компонент для каких-либо полезных целей. Давайте начнем с прочтения значения компонента таким образом, чтобы мы могли его увидеть.

## Значение компонента чтения {#reading-component-value}

Давайте добавим новый элемент, который будет увеличивать счетчик каждый раз, когда по нему щелкают правой кнопкой мыши. Вам следует прочитать страницу [Взаимодействие с пользовательскими элементами](./custom-item-interactions), где описаны методы, которые мы будем использовать в этом руководстве.

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

Не забудьте, как обычно, зарегистрировать элемент в классе `ModItems`.

```java
public static final Item COUNTER = register("counter", CounterItem::new, new Item.Properties());
```

Мы добавим код подсказки, чтобы отображать текущее значение количества кликов при наведении курсора на наш предмет в инвентаре. Мы можем использовать метод `get()` в нашем `ItemStack`, чтобы получить значение нашего компонента следующим образом:

```java
int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
```

Это вернет текущее значение компонента как тип, который мы определили при регистрации нашего компонента. Затем мы можем использовать это значение для добавления записи всплывающей подсказки. Add this line to the `appendHoverText` method in the `CounterItem` class:

```java
public void appendHoverText(ItemStack stack, TooltipContext context, TooltipDisplay displayComponent, Consumer<Component> textConsumer, TooltipFlag type) {
  int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
  textConsumer.accept(Component.translatable("item.example-mod.counter.info", count).withStyle(ChatFormatting.GOLD));
}
```

Не забудьте обновить ваш lang-файл (`/assets/example-mod/lang/en_us.json`) и добавить эти две строки:

```json
{
  "item.example-mod.counter": "Counter",
  "item.example-mod.counter.info": "Used %1$s times"
}
```

Запустите игру и выполните эту команду, чтобы получить новый предмет счетчик со значением 5.

```mcfunction
/give @p example-mod:counter[example-mod:click_count=5]
```

При наведении курсора на этот предмет в инвентаре вы увидите количество, отображаемое во всплывающей подсказке!

![Подсказка с надписью «Использовано 5 раз»](/assets/develop/items/custom_component_1.png)

Однако если вы дадите себе новый предмет Counter _без_ пользовательского компонента, игра вылетит при наведении курсора на предмет в инвентаре. В отчете о сбое вы должны увидеть такую ​​ошибку:

```log
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "net.minecraft.world.item.ItemStack.get(net.minecraft.core.component.DataComponentType)" is null
        at com.example.docs.item.custom.CounterItem.appendHoverText(LightningStick.java:45)
        at net.minecraft.world.item.ItemStack.getTooltipLines(ItemStack.java:767)
```

As expected, since the `ItemStack` doesn't currently contain an instance of our custom component, calling `stack.get()` with our component type will return `null`.

Для решения этой проблемы мы можем использовать три решения.

### Установка значения компонента по умолчанию {#setting-default-value}

When you register your item and pass a `Item.Properties` object to your item constructor, you can also provide a list of default components that are applied to all new items. Если вернуться к нашему классу `ModItems`, где мы регистрируем `CounterItem`, мы можем добавить значение по умолчанию для нашего пользовательского компонента. Добавьте это, чтобы для новых элементов отображалось количество «0».

@[code transcludeWith=::\_13](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

При создании нового элемента он автоматически применит наш пользовательский компонент с заданным значением.

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
