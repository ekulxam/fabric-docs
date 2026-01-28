---
title: 藥水
description: 了解如何新增自訂藥水以實現各種狀態效果。
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

藥水是一種消耗品，能賦予實體一種效果。 玩家可以使用釀造台釀造藥水，或通過各種遊戲機制製作成物品。

## Custom Potions {#custom-potions}

Just like items and blocks, potions need to be registered.

### Creating the Potion {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. We will be directly using a `ModInitializer`-implementing class to
hold this. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. We use our custom effect here. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - Duration of the effect in game ticks.
- `int amplifier` - An amplifier for the effect. For example, Haste II would have an amplifier of 1.

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
