---
title: 药水
description: 学习如何加入多种状态效果的自定义药水。
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

药水是能为实体提供效果的消耗品。 玩家可以使用酿造台酿造药水，或者从其他游戏机制中以物品形式获取。

## 自定义药水{#custom-potions}

和物品和方块一样，药水需要注册。

### 创建物品{#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. 我们将直接使用 `ModInitializer`——实现这个类来持有这个字段。 Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

我们传入一个 `MobEffectInstance` 实例，它的构造方法接收以下 3 个参数：

- `Holder<MobEffect> type` - An effect, represented as a holder. 我们在这里使用我们的自定义效果。 你也可以通过原版的 `MobEffects` 类访问原版效果。
- `int duration` - 状态效果的持续时间（以刻计算）。
- `int amplifier` - 状态效果的增幅。 比如 急迫 II 的增幅是 1。

::: info

To create your own potion effect, please see the [Effects](../entities/effects) guide.

:::

### Registering the Potion {#registering-the-potion}

In our initializer, we will use the `FabricBrewingRecipeRegistryBuilder.BUILD` event to register our potion using the `BrewingRecipeRegistry.registerPotionRecipe` method.

@[code lang=java transclude={29-40}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

`registerPotionRecipe` takes 3 parameters:

- `Holder<Potion> input` - The starting potion, represented by a holder. Usually this can be a Water Bottle or an Awkward Potion.
- `Item item` - The item which is the main ingredient of the potion.
- `Holder<Potion> output` - The resultant potion, represented by a holder.

Once registered, you can brew a Tater potion using a potato.

![Effect in player inventory](/assets/develop/tater-potion.png)
