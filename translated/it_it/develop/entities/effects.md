---
title: Mob Effects
description: Learn how to add custom mob effects.
authors:
  - dicedpixels
  - Friendly-Banana
  - Manchick0
  - SattesKrokodil
  - TheFireBlast
  - YanisBft
authors-nogithub:
  - siglong
  - tao0lu
---

Mob effects, also known as status effects or simply effects, are a condition that can affect an entity. Possono essere positivi, negativi o neutrali in natura. Il gioco base applica questi effetti in vari modi, come cibi, pozioni ecc.

Il comando `/effect` può essere usato per applicare effetti su un'entità.

## Custom Mob Effects {#custom-mob-effects}

In questo tutorial aggiungeremo un nuovo effetto personalizzato chiamato _Tater_ che ti darà un punto esperienza in ogni tick di gioco.

### Extend `MobEffect` {#extend-mobeffect}

Let's create a custom effect class by extending `MobEffect`, which is the base class for all effects.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/TaterEffect.java)

### Registrare il tuo Effetto Personalizzato {#registering-your-custom-effect}

Similar to block and item registration, we use `Registry.register` to register our custom effect into the
`MOB_EFFECT` registry. Questo può essere fatto nel nostro initializer.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/ExampleModEffects.java)

### Texture {#texture}

The mob effect icon is a 18x18 PNG which will appear in the player's inventory screen. Posiziona la tua icona personalizzata in:

```text:no-line-numbers
resources/assets/example-mod/textures/mob_effect/tater.png
```

<DownloadEntry visualURL="/assets/develop/tater-effect.png" downloadURL="/assets/develop/tater-effect-icon.png">Texture di Esempio</DownloadEntry>

### Traduzioni {#translations}

Come per qualsiasi altra traduzione, puoi aggiungere un'entrata con il formato ID `"effect.example-mod.effect-identifier": "Value"` al file di lingua.

```json
{
  "effect.example-mod.tater": "Tater"
}
```

### Applicare l'Effetto {#applying-the-effect}

Vale la pena di dare un'occhiata a come si aggiunge solitamente un effetto ad un'entità.

::: tip

For a quick test, it might be a better idea to use the previously mentioned `/effect` command:

```mcfunction
effect give @p example-mod:tater
```

:::

To apply an effect internally, you'd want to use the `LivingEntity#addMobEffect` method, which takes in
a `MobEffectInstance`, and returns a boolean, specifying whether the effect was successfully applied.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/ReferenceMethods.java)

| Argomento   | Tipo                | Descrizione                                                                                                                                                                                                                                                                             |
| ----------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `effect`    | `Holder<MobEffect>` | A holder that represents the effect.                                                                                                                                                                                                                                    |
| `duration`  | `int`               | La durata dell'effetto **in tick**; **non** in secondi                                                                                                                                                                                                                                  |
| `amplifier` | `int`               | L'amplificatore al livello dell'effetto. Non corrisponde al **livello** dell'effetto, ma è invece aggiunto al di sopra. Per cui un `amplifier` di `4` => livello `5`                                                                                    |
| `ambient`   | `boolean`           | Questo è un po' complesso. In pratica indica che l'effetto è stato aggiunto dall'ambiente (per esempio un **Faro**) e non ha una causa diretta. Se `true`, l'icona dell'effetto nel HUD avrà un overlay color ciano. |
| `particles` | `boolean`           | Se si mostrano le particelle.                                                                                                                                                                                                                                           |
| `icon`      | `boolean`           | Se si mostra un'icona dell'effetto nel HUD. L'effetto sarà mostrato nell'inventario indipendentemente da questo valore.                                                                                                                                 |

::: info

Per creare una pozione che usa questo effetto, per favore vedi la guida [Pozioni](../items/potions).

:::

<!---->
