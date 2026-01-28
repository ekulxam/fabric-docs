---
title: Pociones
description: Aprende a agregar pociones personalizadas para varios efectos de estado.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

Las pociones son items consumibles que le dan un efecto a una entidad. Un jugador puede crear una poción usando un Soporte para Pociones u obtenerlas como items a partir de varias mecánicas del juego.

## Pociones Personalizadas

Just like items and blocks, potions need to be registered.

### Crear la Poción

Let's start by declaring a field to hold your `Potion` instance. We will be directly using a `ModInitializer`-implementing class to
hold this. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. Aquí usamos nuestro efecto personalizado. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - La duración del efecto medido en ticks del juego.
- `int amplifier` - Un amplificador para el efecto. Por ejemplo, Prisa Minera II tendría un amplificador de 1.

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
