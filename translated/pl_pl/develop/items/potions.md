---
title: Mikstury
description: Dowiedz się, jak stworzyć własne mikstury dla różnych efektów.
authors:
  - cassiancc
  - dicedpixels
  - Drakonkinst
  - JaaiDead
  - PandoricaVi
---

Mikstury, to materiały możliwe do spożycia, które przyznają istocie efekt. Gracz może warzyć mikstury używając statywu alchemicznego, lub otrzymując je jako przedmioty (itemy) z innych mechanik w grze.

## Własne Mikstury {#custom-potions}

Tak samo jak przedmioty i bloki, mikstury muszą być zarejestrowane.

### Tworzenie Własnej Mikstury {#creating-the-potion}

Let's start by declaring a field to hold your `Potion` instance. Będziemy bezpośrednio używać klasy implementującej 'ModInitializer' do przechowywania owej mikstury. Note the use of `Registry.registerForHolder`, as, like mob effects, most vanilla methods that use potions prefer them as holders.

@[code lang=java transclude={18-27}](@/reference/latest/src/main/java/com/example/docs/potion/ExampleModPotions.java)

We pass an instance of `MobEffectInstance`, which takes 3 parameters:

- `Holder<MobEffect> type` - An effect, represented as a holder. Używamy naszego własnego efektu tutaj. Alternatively you can access vanilla effects
  through vanilla's `MobEffects` class.
- `int duration` - Czas trwania efektu w tickach.
- `int amplifier` - Mnożnik dla efektu. Na przykład, Pośpiech II będzie mieć mnożnik o wartości 1.

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
