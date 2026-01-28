---
title: カスタムツール (道具・武器)
description: Learn how to create your own tools and configure their properties.
authors:
  - IMB11
---

道具や武器(以下ツール)は、ゲーム内を生き延び、冒険していくには必要不可欠なものです。これらは、資源の収集・建造物の建設・外敵から身を守ることに対して重要なアイテムです。(一部、カスタム防具を終えていないと進めない箇所があります)

## ツール素材の定義 {#creating-a-tool-material}

`ToolMaterial`オブジェクトを用いて、ツールの作成に用いる素材を定義することができます。`ToolMaterial`オブジェクトは、ツール素材定義用のクラスを新たに作成して、その中の`static`フィールドに定義すると扱いやすいです。

@[code transcludeWith=:::guidite_tool_material](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

`ToolMaterial`クラスのコンストラクタは、以下のパラメータをこの順番に従って受け取ります。

| Parameter                 | Description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| `incorrectBlocksForDrops` | この素材で作られたツールを、このタグの中に定義されたブロックに対して使用すると、そのブロックは破壊された時にドロップしなくなります。 |
| `durability`              | この素材を用いて作られたツールの耐久値。                                               |
| `speed`                   | この素材を用いて作られたツールの採掘速度。                                              |
| `attackDamageBonus`       | この素材が持つ追加の攻撃力。ツール自体の攻撃力に加算される。                                     |
| `enchantmentValue`        | エンチャント時に、良いエンチャントが付くかどうかに影響する値。                                    |
| `repairItems`             | この素材で作られたツールを金床で修理する時に使用できるアイテムを指定するアイテムタグ。                        |

For this example, we will use the same repair item we will be using for armor. We define the tag reference as follows:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

バランスがいい値は何かと迷う時は、`net.minecraft.item`パッケージの`ToolMaterial`レコードにある、`ToolMaterial.STONE`や`ToolMaterial.DIAMOND`のような、バニラのツール素材の情報を参考にしてください。

## ツールアイテムの作成 {#creating-tool-items}

[初めてのアイテム作成](./first-item) と同じregisterメソッドで、ツールアイテムを作成できます。

@[code transcludeWith=:::7](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

2つのfloat値（`1f, 1f`）はそれぞれ、ツール自体の攻撃力と攻撃速度を表しています。

Remember to add them to a creative tab if you want to access them from the creative inventory!

@[code transcludeWith=:::8](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

また、テクスチャ・言語別の名前、モデル情報もそれぞれ設定する必要があります。 ここで、モデル情報については、通常の`item/generated`ではなく、`item/handheld`親モデルとして使うようにします。

この例では、以下のモデルとテクスチャを使って、”Guidite Sword” アイテムを作成します。

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/item/guidite_sword.json)

<0>Texture</0>

以上です！ クリエイティブモードのインベントリの道具タブに、先ほど追加した”Guidite Sword” が表示されているはずです。

![Finished tools in inventory](/assets/develop/items/tools_1.png)
