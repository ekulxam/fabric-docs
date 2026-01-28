---
title: Mob Effects
description: Learn how to add custom mob effects.
authors:
  - dicedpixels
  - Friendly-Banana
  - Manchick0
  - SattesKrokodil
  - FireBlast
  - YanisBft
authors-nogithub:
  - siglong
  - tao0lu
---

Mob effects, also known as status effects or simply effects, are a condition that can affect an entity. Pueden ser positivos, negativos o neutrales en naturaleza. El juego base aplica estos efectos de varias maneras como comida, pociones, etc.

El comando `/effect` puede ser usado para aplicar efectos en una entidad.

## Custom Mob Effects {#custom-mob-effects}

En este tutorial añadiremos un nuevo efecto de estado personalizado llamado _Tater_, el cual te da un punto de experiencia por cada tick del juego.

### Extend `MobEffect` {#extend-mobeffect}

Let's create a custom effect class by extending `MobEffect`, which is the base class for all effects.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/TaterEffect.java)

### Registrar tu Efecto Personalizado

Similar to block and item registration, we use `Registry.register` to register our custom effect into the
`MOB_EFFECT` registry. Esto se puede hacer en nuestro inicializador.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/ExampleModEffects.java)

### Textura

The mob effect icon is a 18x18 PNG which will appear in the player's inventory screen. Coloca tu ícono personalizado en:

```text:no-line-numbers
resources/assets/example-mod/textures/mob_effect/tater.png
```

<DownloadEntry visualURL="/assets/develop/tater-effect.png" downloadURL="/assets/develop/tater-effect-icon.png">Example Texture</DownloadEntry>

### Traducciones

Like any other translation, you can add an entry with ID format `"effect.example-mod.effect-identifier": "Value"` to the
language file.

```json
{
  "effect.example-mod.tater": "Tater"
}
```

### Applying The Effect {#applying-the-effect}

It's worth taking a look at how you'd typically apply an effect to an entity.

::: tip

For a quick test, it might be a better idea to use the previously mentioned `/effect` command:

```mcfunction
effect give @p example-mod:tater
```

:::

To apply an effect internally, you'd want to use the `LivingEntity#addMobEffect` method, which takes in
a `MobEffectInstance`, and returns a boolean, specifying whether the effect was successfully applied.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/ReferenceMethods.java)

| Argument          | Type                | Description                                                                                                                                                                                                                                                                                                                      |
| ----------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Efectos de Estado | `Holder<MobEffect>` | A holder that represents the effect.                                                                                                                                                                                                                                                                             |
| `duration`        | `int`               | The duration of the effect **in ticks**; **not** seconds                                                                                                                                                                                                                                                                         |
| `amplifier`       | `int`               | The amplifier to the level of the effect. It doesn't correspond to the **level** of the effect, but is rather added on top. Hence, `amplifier` of `4` => level of `5`                                                                                                                            |
| `ambient`         | `boolean`           | This is a tricky one. It basically specifies that the effect was added by the environment (e.g. a **Beacon**) and doesn't have a direct cause. If `true`, the icon of the effect in the HUD will appear with an aqua overlay. |
| `particles`       | `boolean`           | Whether to show particles.                                                                                                                                                                                                                                                                                       |
| `icon`            | `boolean`           | Whether to display an icon of the effect in the HUD. The effect will be displayed in the inventory regardless of this flag.                                                                                                                                                                      |

::: info

::: info
Para crear una poción que use este efecto, por favor visita la guía sobre [Pociones](../items/potions).

:::

<!---->
