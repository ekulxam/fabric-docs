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

Mob effects, also known as status effects or simply effects, are a condition that can affect an entity. ステータス効果はそれぞれ、ポジティブ・ネガティブ・中立のいずれかのタイプを持ちます。 ステータス効果を付与する方法は、食べ物やポーションなど、様々な方法があります。

コマンドでステータス効果を付与する場合は、`/effect`コマンドを使用します。

## Custom Mob Effects {#custom-mob-effects}

このチュートリアルでは、`Tater`という新たなカスタムステータス効果を追加します。これは、ゲームチック毎に経験値を 1 付与する効果を持ちます。

### Extend `MobEffect` {#extend-mobeffect}

Let's create a custom effect class by extending `MobEffect`, which is the base class for all effects.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/TaterEffect.java)

### カスタムステータス効果の登録 {#registering-your-custom-effect}

Similar to block and item registration, we use `Registry.register` to register our custom effect into the
`MOB_EFFECT` registry. (下の例では、ModInitializerを直接実装していますが、その他の方法と同じく、TaterEffectクラスにinitializeメソッドを作成し、onInitializeメソッドから呼び出して初期化すれば大丈夫です。)

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/effect/ExampleModEffects.java)

### テクスチャ {#texture}

The mob effect icon is a 18x18 PNG which will appear in the player's inventory screen. アイコン画像は以下で示したフォルダに、ファイル名を新しいステータス効果のidと同じくして格納します。

```text:no-line-numbers
resources/assets/example-mod/textures/mob_effect/tater.png
```

<DownloadEntry visualURL="/assets/develop/tater-effect.png" downloadURL="/assets/develop/tater-effect-icon.png">Example Texture</DownloadEntry>

### 言語別の名前 {#translations}

Like any other translation, you can add an entry with ID format `"effect.example-mod.effect-identifier": "Value"` to the
language file.

```json
{
  "effect.example-mod.tater": "Tater"
}
```

### ステータス効果の適用 {#applying-the-effect}

ここでは、エンティティにステータス効果を適用する方法について説明します。

::: tip

For a quick test, it might be a better idea to use the previously mentioned `/effect` command:

```mcfunction
effect give @p example-mod:tater
```

:::

To apply an effect internally, you'd want to use the `LivingEntity#addMobEffect` method, which takes in
a `MobEffectInstance`, and returns a boolean, specifying whether the effect was successfully applied.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/ReferenceMethods.java)

| Argument    | Type                | Description                                                                                                                                                    |
| ----------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `effect`    | `Holder<MobEffect>` | A holder that represents the effect.                                                                                                           |
| `duration`  | `int`               | ステータス効果の継続時間（秒ではなくゲームティック単位）                                                                                                                                   |
| `amplifier` | `int`               | ステータス効果のレベルに関する値。 効果レベルは、初期値にこのamplifierが足された値になります。 例えば、amplifierが4の場合、効果レベルは5になります。                                                                          |
| `ambient`   | `boolean`           | 少しややこしい概念です。 これは基本的に、その効果が環境要因（ビーコンなど）によって付与されるステータス効果であり、食べ物やポーションなどの直接的な原因で付与されるものではないことを指定します。 trueに設定した場合、HUD(プレイヤーの右上)のアイコンが水色っぽくなります。 |
| `particles` | `boolean`           | パーティクルを表示するか否かを指定します。                                                                                                                                          |
| `icon`      | `boolean`           | HUDにステータス効果のアイコンを表示するか否かを設定します。 このフラグに関係なく、インベントリを開いた時にはアイコンが表示されます。                                                                                           |

::: info

ここで作成したステータス効果を設定したポーションを作成する例は、[ポーション](../items/potions) を参照ください。

:::

<!---->
