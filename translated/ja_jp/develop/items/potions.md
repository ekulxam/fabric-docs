---
title: ポーション
description: Learn how to add a custom potion for various status effects.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

ポーションは、エンティティにステータス効果を与えるアイテムです。 プレイヤーは、醸造台での醸造などの様々な場所でポーションを入手できます。

## カスタムポーション {#custom-potions}

カスタムポーションもアイテムやブロックと同様に、レジストリに登録します。

### カスタムポーションの作成 {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. (下の例では、`ModInitializer`を直接実装していますが、[初めてのアイテム作成](./first-item) などと同様に、`Potion`登録用のクラスを作成して、initializeメソッドで初期化すれば大丈夫です。) Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. ステータス効果。ここではカスタムステータス効果を使用しています。 Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - ステータス効果の継続時間（秒ではなくゲームティック単位）。
- `int amplifier` - ステータス効果のレベルに関する値。 例えば、採掘速度上昇の効果のレベルがIIの場合、この値は1に設定されています。

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
