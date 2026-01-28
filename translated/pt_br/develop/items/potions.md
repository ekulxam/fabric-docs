---
title: Poções
description: Aprenda a adicionar poções customizadas para vários efeitos de estado.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

Poções são consumíveis que concedem efeitos a uma entidade. Um jogador pode preparar poções usando um Suporte de Poções ou obtê-las como itens através de várias outras mecânicas do jogo.

## Poções Personalizadas

Just like items and blocks, potions need to be registered.

### Criando a Poção

Let's start by declaring a field to hold your `Potion` instance. We will be directly using a `ModInitializer`-implementing class to
hold this. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. Usamos nosso efeito personalizado aqui. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - Duração do efeito em ticks do jogo.
- `int amplifier` - Um amplificador para o efeito. Por exemplo, Pressa II teria um amplificador de 1.

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
