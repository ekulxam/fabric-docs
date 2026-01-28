---
title: Зелья
description: Узнайте, как добавить собственное зелье для различных статусных эффектов.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

Зелья — это расходные материалы, которые дают сущности определенный эффект. Игрок может варить зелья, используя Варочную Стойку, или получать их как предметы с помощью различных других игровых механик.

## Пользовательские зелья {#custom-potions}

Так же, как предметы и блоки, зелья необходимо регистрировать.

### Создание зелья {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. Мы будем напрямую использовать `ModInitializer`-реализующий класс для хранения этого. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. Здесь мы используем наш собственный эффект. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - Длительность эффекта в игровых тиках.
- `int amplifier` - Усилитель эффекта. Например, Haste II будет иметь усилитель 1.

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
