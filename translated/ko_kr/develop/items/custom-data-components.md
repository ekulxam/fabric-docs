---
title: 사용자 지정 데이터 구성 요소
description: 1.20.5의 새로운 구성 요소 시스템에 사용자 지정 데이터를 추가하는 방법을 알아보세요.
authors:
  - Romejanic
---

아이템이 더 복잡해질수록, 각 아이템마다 데이터를 저장해야 하는 경우도 생길 수 있습니다. 게임에서는 `ItemStack`에 영구 데이터를 저장하는 방법을 제공하고 있으며, 1.20.5 이후의 방법으로는 `DataComponent`를 사용함으로써 이를 수행할 수 있습니다.

데이터 구성 요소는 기존의 NBT 데이터를 대체하며, `ItemStack`에 데이터를 구조화된 형태로 영구 저장할 수 있도록 합니다. 또, 데이터 구성 요소는 각각 고유한 네임스페이스를 가지고 있어, 새로운 데이터 구성 요소를 구현하고 나중에 접근할 수 있습니다. 바닐라 데이터 구성 요소의 전체 목록은 [Minecraft Wiki](https://minecraft.wiki/w/Data_component_format#List_of_components)에서 확인할 수 있습니다.

사용자 지정 구성 요소를 등록하는 것과 더불어, 바닐라 구성 요소에서도 사용되는 구성 요소 API의 일반적인 사용 방법도 알아볼 것입니다. You can see and access the definitions of all vanilla components in the `DataComponents` class.

## 구성 요소 등록 {#registering-a-component}

As with anything else in your mod you will need to register your custom component using a `DataComponentType`. 구성 요소 형태에는 구성 요소의 값의 형태와 같이 기본적인 매개 변수가 입력됩니다. 구성 요소 형태에 대한 자세한 내용은 [기본 데이터 구성 요소](#basic-data-components)와 [고급 데이터 구성 요소](#advanced-data-components)를 다루며 알아볼 것입니다.

먼저 데이터 구성 요소를 구현할 클래스를 결정합니다. 예제에서는 `component` 패키지를 생성한 후, `ModComponents` 클래스를 추가하여 모든 구성 요소 형태를 추가할 것입니다. Make sure you call `ModComponents.initialize()` in your [mod's initializer](../getting-started/project-structure#entrypoints).

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

구성 요소 형태를 등록하는 기본적인 코드는 다음과 같습니다:

```java
public static final DataComponentType<?> MY_COMPONENT_TYPE = Registry.register(
    BuiltInRegistries.DATA_COMPONENT_TYPE,
    Identifier.fromNamespaceAndPath(ExampleMod.MOD_ID, "my_component"),
    DataComponentType.<?>builder().persistent(null).build()
);
```

주의해야 할 부분이 있습니다. 첫 번째와 네 번째 줄을 보면, `?`를 찾을 수 있습니다. 해당 위치에는 구성 요소의 값의 형태가 입력되어야 합니다. 곧 입력할 것입니다.

Secondly, you must provide an `Identifier` containing the intended ID of your component. 모드의 아이디로 네임스페이스하면 됩니다.

Lastly, we have a `DataComponentType.Builder` that creates the actual `DataComponentType` instance that's being registered. 여기에는 고려해야 하는 또 다른 결정적인 세부 사항이 포함되어 있는데, 바로 구성 요소의 '코덱'입니다. 지금은 `null`이지만 이것도 곧 입력할 것입니다.

## 기본적인 데이터 구성 요소 {#basic-data-components}

기본적인 데이터 구성 요소는(`minecraft:damage` 등) `int`, `float`, `boolean`이나 `String`처럼 단일 값으로 구성됩니다.

예시로, 플레이어가 아이템을 들고 얼마나 오른쪽 마우스를 클릭했는지 `Integer`로 저장하는 구성 요소를 만들어봅시다. 다음과 같이 구성 요소 등록을 변경합니다:

@[code transcludeWith=::2](@/reference/latest/src/main/java/com/example/docs/component/ModComponents.java)

이제 일반 형태로 `<Integer>`을 사용하는 것을 확인할 수 있습니다. 이 구성 요소에 단일 `int` 값이 저장될 것임을 의미합니다. For our codec, we are using the provided `ExtraCodecs.POSITIVE_INT` codec. 이런 간단한 구성 요소는 기본 코덱만으로 충분이 구현할 수 있지만, (아래에서 다루게 될) 복잡한 시나리오에서는 사용자 지정 코덱이 필요할 수 있습니다.

이제 게임을 시작하면, 다음과 같은 명령어를 실행할 수 있을 것입니다:

![사용자 지정 구성 요소가 입력된 /give 명령어](/assets/develop/items/custom_component_0.png)

명령어를 실행하면, 방금 추가한 구성 요소가 포함된 아이템을 획득할 수 있을 것입니다. 하지만, 이 구성 요소는 아직 아무런 용도로도 사용되지 않습니다. 구성 요소의 값을 읽을 수 있게 하는 것으로 시작해봅시다.

## 컴포넌트 데이터 읽기 {#reading-component-value}

오른쪽 마우스로 클릭하면 카운터가 증가하는 새로운 아이템을 추가해 봅시다. 이 설명서는 [사용자 지정 아이템 상호작용](./custom-item-interactions)에서 설명한 일부 기술을 사용합니다.

@[code transcludeWith=::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CounterItem.java)

`ModItems` 클래스에서 아이템을 등록해야 한다는 것을 잊지 마세요.

```java
public static final Item COUNTER = register("counter", CounterItem::new, new Item.Properties());
```

보관함에서 아이템에 마우스를 올렸을 때 카운터의 현재 값을 표시하는 도구 설명을 추가해 봅시다. 다음과 같이 `ItemStack#get()` 메소드를 사용하여 컴포넌트 값을 불러올 수 있습니다:

```java
int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
```

이렇게 하면 컴포넌트를 등록했을 때 지정한 자료형으로 현재 컴포넌트의 값을 반환받을 수 있습니다. 이제 이 값을 도구 설명에 사용할 수 있습니다. Add this line to the `appendHoverText` method in the `CounterItem` class:

```java
public void appendHoverText(ItemStack stack, TooltipContext context, TooltipDisplay displayComponent, Consumer<Component> textConsumer, TooltipFlag type) {
  int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
  textConsumer.accept(Component.translatable("item.example-mod.counter.info", count).withStyle(ChatFormatting.GOLD));
}
```

Don't forget to update your lang file (`/assets/example-mod/lang/en_us.json`) and add these two lines:

```json
{
  "item.example-mod.counter": "Counter",
  "item.example-mod.counter.info": "Used %1$s times"
}
```

게임을 실행한 다음, 다음 명령어를 입력해서 카운터 아이템(count: 5)을 획득하세요.

```mcfunction
/give @p example-mod:counter[example-mod:click_count=5]
```

인벤토리에서 이 아이템에 마우스를 올리면, 툴팁에 개수가 표시됩니다!

![툴팁에 "Used 5 times"라고 표시됩니다.](/assets/develop/items/custom_component_1.png)

하지만 커스텀 컴포넌트가 없는 새로운 카운터 아이템을 직접 지급하면, 인벤토리에서 아이템에 마우스를 올릴 때 게임이 크래시됩니다. 크래시 보고서에서 다음과 같은 오류를 확인할 수 있습니다:

```log
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "net.minecraft.world.item.ItemStack.get(net.minecraft.core.component.DataComponentType)" is null
        at com.example.docs.item.custom.CounterItem.appendHoverText(LightningStick.java:45)
        at net.minecraft.world.item.ItemStack.getTooltipLines(ItemStack.java:767)
```

예상한 대로, 현재 `ItemStack`에 커스텀 컴포넌트 인스턴스가 없기 때문에, `stack.get()`에 해당 컴포넌트 타입을 호출하면 `null`을 반환합니다.

이 문제를 해결하기 위해 사용할 수 있는 세 가지 방법이 있습니다.

### 기본 컴포넌트 값 설정 {#setting-default-value}

When you register your item and pass a `Item.Properties` object to your item constructor, you can also provide a list of default components that are applied to all new items. `CounterItem`을 등록하는 `ModItems` 클래스에서, 커스텀 컴포넌트의 기본값을 추가할 수 있습니다. 새 아이템이 `0`의 개수를 표시하도록 다음 코드를 추가하세요.

@[code transcludeWith=::\_13](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

새 아이템이 생성될 때, 지정한 값과 함께 커스텀 컴포넌트가 자동으로 적용됩니다.

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
