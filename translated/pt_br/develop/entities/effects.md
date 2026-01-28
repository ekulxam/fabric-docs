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

Mob effects, also known as status effects or simply effects, are a condition that can affect an entity. Eles podem ser de natureza positiva, negativa ou neutra. O jogo base aplica esses efeitos de vários modos, como comidas, poções, etc.

O comando `/effect` pode ser usado para aplicar efeitos numa entidade.

## Custom Mob Effects {#custom-mob-effects}

Neste tutorial adicionaremos um novo efeito personalizado chamado _Tater_, que lhe dará um ponto de experiência a cada tick do jogo.

### Extend `MobEffect` {#extend-mobeffect}

Let's create a custom effect class by extending `MobEffect`, which is the base class for all effects.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/TaterEffect.java)

### Registrando seu Efeito Personalizado

Similar to block and item registration, we use `Registry.register` to register our custom effect into the
`MOB_EFFECT` registry. Isso pode ser feito no nosso inicializador.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/ExampleModEffects.java)

### Textura

The mob effect icon is a 18x18 PNG which will appear in the player's inventory screen. Coloque seu ícone personalizado em:

```text:no-line-numbers
resources/assets/example-mod/textures/mob_effect/tater.png
```

<DownloadEntry visualURL="/assets/develop/tater-effect.png" downloadURL="/assets/develop/tater-effect-icon.png">Example Texture</DownloadEntry>

### Traduções

Like any other translation, you can add an entry with ID format `"effect.example-mod.effect-identifier": "Value"` to the
language file.

```json
{
  "effect.example-mod.tater": "Tater"
}
```

### Aplicando o Efeito {#applying-the-effect}

Vale a pena dar uma olhada em como você normalmente aplicaria um efeito a uma entidade.

::: tip

For a quick test, it might be a better idea to use the previously mentioned `/effect` command:

```mcfunction
effect give @p example-mod:tater
```

:::

To apply an effect internally, you'd want to use the `LivingEntity#addMobEffect` method, which takes in
a `MobEffectInstance`, and returns a boolean, specifying whether the effect was successfully applied.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/ReferenceMethods.java)

| Argumento   | Tipo                | Descrição                                                                                                                                                                                                                                                                                        |
| ----------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `effect`    | `Holder<MobEffect>` | A holder that represents the effect.                                                                                                                                                                                                                                             |
| `duration`  | `int`               | A duração do efeito **em ticks**, **não** segundos                                                                                                                                                                                                                                               |
| `amplifier` | `int`               | O amplificador ao nível do efeito. Não corresponde ao **nível** do efeito, mas é adicionado em cima. Portanto, `amplifier` de `4` => nível de `5`                                                                                                                |
| `ambient`   | `boolean`           | Este é complicado. Ele basicamente especifica que o efeito foi adicionado pelo ambiente (por exemplo, um **Sinalizado**) e não tem uma causa direta. Se `true`, o ícone do efeito no HUD aparecerá com uma sobreposição aqua. |
| `particles` | `boolean`           | Se as partículas devem ser mostradas.                                                                                                                                                                                                                                            |
| `icon`      | `boolean`           | Se um ícone do efeito deve ser exibido no HUD. O efeito será exibido no inventário independentemente desta flag.                                                                                                                                                 |

::: info

::: info
Para criar uma poção que utiliza este efeito, consulte o [guia de Poções](../items/potions).

:::

<!---->
