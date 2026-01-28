---
title: カスタム防具
description: Learn how to create your own armor sets.
authors:
  - IMB11
---

防具は、Mobや他のプレイヤーからの攻撃に対する防御力を高めるアイテムです。

## 防具素材用のクラスを定義 {#creating-an-armor-materials-class}

ここでは、防具の素材(鉄やダイヤモンド)を定義するクラスを作成します。厳密には専用のクラスを作る必要はありませんが、防具素材の定義に必要なstaticフィールドが多いため、作成することをお勧めします。一般的に、処理ごとにクラスを分ける習慣はとても良いことです。

For this example, we'll create a `GuiditeArmorMaterial` class to store our static fields.

### 基礎耐久値 {#base-durability}

This constant will be used in the `Item.Properties#maxDamage(int damageValue)` method when creating our armor items, it is also required as a parameter in the `ArmorMaterial` constructor when we create our `ArmorMaterial` object later.

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

基礎耐久値の選ぶ時、バランスがいい値は何かと迷うかもしれません。その場合は、`net.minecraft.item.equipment`パッケージ内にある、`ArmorMaterials`インターフェースを確認してみてください。バニラの防具素材(鉄やダイヤモンド)の基礎耐久値を確認できます。

### 装備中のテクスチャ定義に必要なレジストリキー {#equipment-asset-registry-key}

一般的に、MODで使用する様々なレジストリキーは、定数として定義しておくことが推奨されます。下のコードは、防具の装備中のテクスチャ情報をレジストリに登録するための、レジストリキーを定義しています。

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

このレジストリキーも、`ArmorMaterial`のコンストラクタに引数として渡します。

### `ArmorMaterial`インスタンスの定義 {#armormaterial-instance}

`ArmorMaterial`は、レコードで定義されています。`ArmorMaterial`のインスタンスを作成して、新しい防具素材を定義することができます。(もし、`REPAIRS_GUIDITE_ARMOR`の値が分からなければ、`TagKey.of(RegistryKeys.ITEM,Identifier.of(MODの名前.MOD_ID, "guidite"))`と入れてみてください。

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

`ArmorMaterial`のコンストラクタは、以下のパラメータをこの順番に従って受け付けます。

| Parameter                      | Description                                                                                                                                                                 |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `耐久値 durability`               | これは、各防具タイプに共通した、素材の基本耐久値です。これは、この素材を使用する各防具の全耐久値(素材の基本耐久値×各防具タイプの耐久値)を計算するときに使用されます。 この例では、staticフィールドに定義したBASE_DURABILITYの値を設定します。 |
| `防御力 defense`                  | 各防具タイプ (Enumで定義された`EquipmentType`内の定数)と、各防具の防御力を対応させたMapオブジェクトです。体力バーの上の灰色のチェストプレートで示されている値です。この素材で作られた各防具が、どれだけこの防御力を持つかを定義します。                         |
| `エンチャント可能性 enchantmentValue`   | エンチャント時に、良いエンチャントが付くかどうかに影響する値です。                                                                                                                                           |
| `装備音 equipSound`               | この素材で作られた防具を装備した時に再生されるサウンドです。 サウンドの詳細については、[カスタムサウンド](../sounds/custom) を参照してください。                                                                                         |
| `強靭性 toughness`                | この素材の“強靭さ”を表すfloat値です。クリーパーの爆発などの大きなダメージを受けた時のダメージ計算に影響を与える数値です。この値が大きければ、大きなダメージを受けた時のダメージ吸収率が大きくなります。                                                                     |
| `ノックバック耐久 knockbackResistance` | この素材で作られた防具を来たプレイヤー(Mobも含む)の、ノックバックに影響を与えるfloat値です。                                                                                                      |
| `修理素材 repairIngredient`        | この素材で作られた防具を金床で修理する時に使用できるアイテムを指定するアイテムタグです。                                                                                                                                |
| `装備中情報 assetId`                | 装備中のテクスチャ定義に用いるレジストリキーです。                                                                                                                                                   |

We define the repair ingredient tag reference as follows:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

バランスがいい値は何かと迷う時は、`ArmorMaterials`インターフェイスにある、バニラの防具素材の`ArmorMaterial`インスタンスを参考にしてください。

## 防具アイテムの定義 {#creating-the-armor-items}

防具素材を定義できたら、その素材を使った防具アイテムを作成することができるようになります。この例では、防具アイテムを`ModItems`クラスの中で作成します。

防具アイテムは、全ての防具タイプを揃える必要はありません。ブーツだけやレギンスだけでも大丈夫です。バニラの ”カメの甲羅” でも、防具は頭部のアイテムだけが定義されています。

`ArmorMaterial`は素材情報であり、アイテムの耐久性に関する情報を自動で提供しません。 For this reason the base durability needs to be manually added to the armor items' `Item.Properties` when registering them.

This is achieved by passing the `BASE_DURABILITY` constant we created previously into the `maxDamage` method in the `Item.Properties` class.

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

また、防具アイテムに言語別の名前を追加する場合は、言語ファイルへ各防具アイテムの翻訳キーペアを登録してください。

## テクスチャとモデルの定義 {#textures-and-models}

各防具アイテムのテクスチャを定義します。防具アイテムでは、装備中に表示されるテクスチャについても、定義する必要があります。今回では、”人型(humanoid)” エンティティ（プレイヤー、ゾンビ、スケルトンなど）が装備する例で説明します。

### テクスチャとモデル {#item-textures-and-model}

防具アイテムのテクスチャの登録方法は、[初めてのアイテム作成](./first-item#adding-a-texture-and-model)と同様に、通常のアイテムと同じ方法で定義できます。テクスチャファイルを用意して、親モデルを`item/generated`に設定するような方法です。

この例では、以下のテクスチャとモデルJSONファイルを使用します。

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">Item Textures</DownloadEntry>

::: info

You will need model JSON files for all the items, not just the helmet, it's the same principle as other item models.

:::

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/item/guidite_helmet.json)

As you can see, in-game the armor items should have suitable models:

![Armor item models](/assets/develop/items/armor_1.png)

### Armor Textures {#armor-textures}

When an entity wears your armor, nothing will be shown. This is because you're missing textures and the equipment model definitions.

![Broken armor model on player](/assets/develop/items/armor_2.png)

There are two layers for the armor texture, both must be present.

Previously, we created a `ResourceKey<EquipmentAsset>` constant called `GUIDITE_ARMOR_MATERIAL_KEY` which we passed into our `ArmorMaterial` constructor. It's recommended to name the texture similarly, so in our case, `guidite.png`

- `assets/example-mod/textures/entity/equipment/humanoid/guidite.png` - Contains upper body and boot textures.
- `assets/example-mod/textures/entity/equipment/humanoid_leggings/guidite.png` - Contains legging textures.

<DownloadEntry downloadURL="/assets/develop/items/example_armor_layer_textures.zip">Guidite Armor Model Textures</DownloadEntry>

::: tip

If you're updating to 1.21.11 from an older version of the game, the `humanoid` folder is where your `layer0.png` armor texture goes, and the `humanoid_leggings` folder is where your `layer1.png` armor texture goes.

:::

Next, you'll need to create an associated equipment model definition. These go in the `/assets/example-mod/equipment/` folder.

The `ResourceKey<EquipmentAsset>` constant we created earlier will determine the name of the JSON file. In this case, it'll be `guidite.json`.

Since we only plan to add "humanoid" (helmet, chestplate, leggings, boots etc.) armor pieces, our equipment model definition will look like this:

@[code](@/reference/latest/src/main/resources/assets/example-mod/equipment/guidite.json)

With the textures and equipment model definition present, you should be able to see your armor on entities that wear it:

![Working armor model on player](/assets/develop/items/armor_3.png)

<!-- TODO: A guide on creating equipment for dyeable armor could prove useful. -->
