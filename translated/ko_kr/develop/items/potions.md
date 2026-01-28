---
title: 물약
description: 여러 상태 효과를 부여하는 사용자 정의 물약을 만드는 방법을 알아보세요.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

물약은 엔티티에게 효과를 주는 소모품입니다. 플레이어는 양조대에서 물약을 제조하거나 여러 게임 메커니즘을 통해 아이템을 얻을 수 있습니다.

## 사용자 정의 물약 {#custom-potions}

아이템과 블록처럼, 물약도 등록이 필요합니다.

### 물약 만들기 {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. 이를 보관하기 위해 `ModInitializer` 구현 클래스를 직접 사용하겠습니다. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. 여기에선 사용자 정의 효과를 사용해볼 것입니다. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - 효과의 지속 시간(틱).
- `int amplifier` - 효과의 세기. 예를 들어, 성급함 II는 1을 세기로 가지게 됩니다.

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
