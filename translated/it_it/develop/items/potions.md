---
title: Pozioni
description: Impara come aggiungere pozioni personalizzate per vari effetti di stato.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

Le pozioni sono oggetti consumabili che conferiscono un effetto a un'entità. Un giocatore può preparare delle pozioni usando l'Alambicco oppure ottenerle come oggetti attraverso varie meccaniche di gioco.

## Pozioni Personalizzate {#custom-potions}

Proprio come gli oggetti e i blocchi, le pozioni devono essere registrate.

### Creare la Pozione {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. Useremo direttamente una classe che implementi `ModInitializer` per conservarla. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. Qui usiamo il nostro effetto personalizzato. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - Durata dell'effetto espressa in tick di gioco.
- `int amplifier` - Un amplificatore per l'effetto. Per esempio, Sollecitudine II avrebbe un amplificatore di 1.

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
