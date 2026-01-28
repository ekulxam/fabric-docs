---
title: 自定义数据组件
description: 学习如何使用 1.20.5 新的组件系统为你的物品添加自定义数据
authors:
  - Romejanic
---

物品越来越复杂，你会发现自己需要存储与每个物品关联的自定义数据。 游戏需要你将持久的数据存储在 `ItemStack`（物品堆）中，在 1.20.5 中，方法就是使用**数据组件**。

数据组件替代了之前版本中的 NBT 数据，替换成能应用在 `ItemStack` 的结构化的数据，从而存储物品堆的持久数据。 数据组件是有命名空间的，也就是说，我们可以实现自己的数据组件，存储 `ItemStack` 的自定义数据，并稍后再访问。 所有原版可用的数据组件可以见于此 [Minecraft wiki 页面](https://zh.minecraft.wiki/w/%E7%89%A9%E5%93%81%E5%A0%86%E5%8F%A0%E7%BB%84%E4%BB%B6#%E7%BB%84%E4%BB%B6%E6%A0%BC%E5%BC%8F)。

除了注册自定义组件外，本页还介绍了组件 API 的一般用法，这些也可用于原版的组件。 你可以在 `DataComponents` 类中查看并访问所有原版组件的定义。

## 注册组件{#registering-a-component}

就你模组中的其他东西一样，你需要使用 `DataComponentType` 注册自定义的组件。 这个组件类型接受一个泛型参数，包含你的组件的值的类型。 之后在讲[基本](#basic-data-components)和[高级](#advanced-data-components)组件时会更深入研究。

把这个组件放到一个合理的类中。 对于这个例子，我们创建一个新的包，叫做 `compoennt`，以及一个类，叫做 `ModComponents`，包含我们所有的组件类型。 确保在你的[模组的初始化器](../getting-started/project-structure#entrypoints)中调用 `ModComponents.initialize()`。

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

这是注册一个组件类型的基本模板：

```java
public static final DataComponentType<?> MY_COMPONENT_TYPE = Registry.register(
    BuiltInRegistries.DATA_COMPONENT_TYPE,
    Identifier.fromNamespaceAndPath(ExampleMod.MOD_ID, "my_component"),
    DataComponentType.<?>builder().persistent(null).build()
);
```

有几点需要注意。 在第一行，你看到了一个 `?`， 这将被替换成你的组件的值的类型， 我们稍后完成。

Secondly, you must provide an `Identifier` containing the intended ID of your component. 其命名空间就是你模组的 ID。

最后，我们有一个 `DataComponentType.Builder`，创建一个需要注册的实际 `DataComponentType` 实例。 这包含我们会需要讨论的另一个重要细节：你的组件的 `Codec`。 现在还是 `null`，但我们也会稍后完成。

## 基本数据组件{#basic-data-components}

基本数据组件（例如 `minecraft:damagae`）包含单个数据值，例如 `int`、`float`、`boolean` 或 `String`。

例如，我们创建一个 `Integer` 值，追踪玩家手持我们的物品右键点击了多少次。 参照下面的代码，更新刚刚注册组件的代码。

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

可以提到，我们这里将 `<Integer>` 传入作为我们的泛型类型，表示这个组件会存储为单个的 `int` 值。 对于我们的 codec，直接使用提供的 `ExtraCodecs.POSITIVE_INT` codec 就可以了。 对于这样简单的组件，可以使用基本的 codec，但是更加复杂的情形可能需要自定义的 codec（后面就会讲到）。

如果开始游戏，就可以输入像这样的命令：

![/give 命令显示自定义的组件](/assets/develop/items/custom_component_0.png)

运行命令时，你应该能收到一个包含组件的物品。 但是，现在还没使用这个组件做些有用的事情。 先开始以我们能看到的方式读取组件的值吧。

## 读取组件的值{#reading-component-value}

添加新物品，每次右键点击时都会增加计数器。 可以阅读[自定义物品交互](./custom-item-interactions)页面以了解我们在这个教程中使用的技巧。

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

记得要和平时一样，在 `ModItems` 类中注册物品。

```java
public static final Item COUNTER = register("counter", CounterItem::new, new Item.Properties());
```

我们会添加一些物品提示，在物品栏中鼠标悬浮在物品上时，显示点击次数的当前值。 可以使用 `ItemStack` 的 `get()` 方法获取组件的值：

```java
int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
```

这会返回组件的当前值，其类型为我们注册组件时定义的类型。 可以将这个值添加到物品提示中。 在 `CounterItem` 类中，把这一行添加到 `appendHoverText` 方法：

```java
public void appendHoverText(ItemStack stack, TooltipContext context, TooltipDisplay displayComponent, Consumer<Component> textConsumer, TooltipFlag type) {
  int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
  textConsumer.accept(Component.translatable("item.example-mod.counter.info", count).withStyle(ChatFormatting.GOLD));
}
```

不要忘记更新你的语言文件（`/assets/example-mod/lang/en_us.json` 和 `/assets/example-mod/lang/zh_cn.json`），并添加这两行：

```json
{
  "item.example-mod.counter": "Counter",
  "item.example-mod.counter.info": "Used %1$s times"
}
```

启动游戏，运行这个命令，给自己一个计数为 5 的新的计数器物品。

```mcfunction
/give @p example-mod:counter[example-mod:click_count=5]
```

在物品栏内鼠标悬停在这个物品上时，你可以看到物品提示中显示了计数！

![显示“使用了 5 次”的物品提示](/assets/develop/items/custom_component_1.png)

但是，如果给自己一个_没有_自定义组件的新的计数器物品，当你在物品栏内选中物品时游戏会崩溃。 崩溃报告中，会提示这样的信息：

```log
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "net.minecraft.world.item.ItemStack.get(net.minecraft.core.component.DataComponentType)" is null
        at com.example.docs.item.custom.CounterItem.appendHoverText(LightningStick.java:45)
        at net.minecraft.world.item.ItemStack.getTooltipLines(ItemStack.java:767)
```

这是因为 `ItemStack` 当前没有包含我们的自定义组件的实例，因此对于我们的组件类型，调用 `stack.get()` 会返回 `null`。

解决这个问题有三种方法。

### 设置默认组件值{#setting-default-value}

当你注册你的物品并传递 `Item.Properties` 对象到你的物品构造函数中，你还可以提供应用于所有新物品的默认组件的列表。 如果回到我们的 `ModItems` 类，注册 `CounterItem` 的地方，就可以为我们的自定义组件添加默认值。 添加这个，这样新物品会显示计数为 `0`。

@[code transcludeWith=::\_13](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

创建了新物品后，就会自动为我们的自定义组件应用给定的值。

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
