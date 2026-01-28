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

Mob effects, also known as status effects or simply effects, are a condition that can affect an entity. Они могут сказываться положительно, отрицательно или нейтрально на сущности. В обычном случае в игре эти эффекты применяются несколькими способами, такими как поедание еды, распитие зелий и так далее.

Можно использовать команду `/effect` для применения эффектов к сущности.

## Custom Mob Effects {#custom-mob-effects}

В этом руководстве мы добавим новый эффект под названием _Tater_, который даёт игроку одно очко опыта каждый игровой такт.

### Extend `MobEffect` {#extend-mobeffect}

Let's create a custom effect class by extending `MobEffect`, which is the base class for all effects.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/TaterEffect.java)

### Регистрация своего эффекта {#registering-your-custom-effect}

Similar to block and item registration, we use `Registry.register` to register our custom effect into the
`MOB_EFFECT` registry. Это можно сделать в нашем инициализаторе.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/ExampleModEffects.java)

### Текстура

The mob effect icon is a 18x18 PNG which will appear in the player's inventory screen. Поместите свою иконку в папку:

```text:no-line-numbers
resources/assets/example-mod/textures/mob_effect/tater.png
```

<DownloadEntry visualURL="/assets/develop/tater-effect.png" downloadURL="/assets/develop/tater-effect-icon.png">Пример текстуры</DownloadEntry>

### Переводы {#translations}

Как и любой другой перевод, вы можете добавить запись с ID формата `"effect.example-mod.effect-identifier": "Значение"` в
языковой файл.

```json
{
  "effect.example-mod.tater": "Tater"
}
```

### Применение эффекта {#applying-the-effect}

Стоит взглянуть на то, как вы обычно применяете эффект к объекту.

::: tip

For a quick test, it might be a better idea to use the previously mentioned `/effect` command:

```mcfunction
effect give @p example-mod:tater
```

:::

To apply an effect internally, you'd want to use the `LivingEntity#addMobEffect` method, which takes in
a `MobEffectInstance`, and returns a boolean, specifying whether the effect was successfully applied.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/ReferenceMethods.java)

| Аргумент    | Тип                 | Описание                                                                                                                                                                                                                                                                                                                    |
| ----------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `effect`    | `Holder<MobEffect>` | A holder that represents the effect.                                                                                                                                                                                                                                                                        |
| `duration`  | `int`               | Продолжительность эффекта **в тиках**; **не** секундах                                                                                                                                                                                                                                                                      |
| `amplifier` | `int`               | Усилитель соответствует уровню эффекта. Это не соответствует **уровню** эффекта, а скорее добавляется сверху. Следовательно, `усилитель` на уровне `4` => уровень `5`                                                                                                                       |
| `ambient`   | `boolean`           | Это очень сложный вопрос. Это в основном указывает на то, что эффект был добавлен окружающей средой (например, **Маяком**) и не имеет прямой причины. Если установлено значение `true`, то на экране появится значок эффекта с аквамариновым наложением. |
| `particles` | `boolean`           | Показывать ли частицы.                                                                                                                                                                                                                                                                                      |
| `icon`      | `boolean`           | Отображать ли значок эффекта в HUD. Эффект будет отображаться в инвентаре независимо от этого флага.                                                                                                                                                                                        |

::: info

::: info
Чтобы узнать, как создать зелье, накладывающее этот эффект, ознакомьтесь с руководством по [зельям](../items/potions).

:::

<!---->
