---
title: ブロック状態
description: Learn why blockstates are a great way to add visual functionality to your blocks.
authors:
  - IMB11
---

ブロック状態とは、ブロックに紐づいたブロックの状態を表すプロパティです。バニラのブロック状態を示すプロパティには、以下の例があります。

- Rotation(回転): 柱ブロックが良い例です。その他のブロックにも設定されます。
- Activated(活性化)： レッドストーン系のブロックに多用されます。かまど・燻製器にも設定されます。
- Age(経過・成長度)：作物・植物・苗木・昆布などに設定されます。

ブロック状態を用いると、NBTデータをブロックエンティティに保存する必要がないため、ワールドサイズを縮小し、TPSの問題を防ぐことができます！

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## ブロック状態の例：柱ブロック {#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

この例では、`axis`というブロック状態を持つ ”Condensed Oak Log” ブロックを作成します。マインクラフトでは、特定の型のブロックを素早く作成できるように、いくつかクラスが定義されており、今回は`PillarBlock(柱ブロック)`を使用します。

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

柱ブロックは、上部と側面の2つのテクスチャを持ちます。このブロックの親モデルとして定義するモデル情報は、`block/cube_column`を選択します。

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">Textures</DownloadEntry>

柱ブロックには水平と垂直の2つの配置状態があり、別々にモデル情報を定義します。

- `condensed_oak_log_horizontal.json`: これは、親モデルを`block/cube_column_horizontal`としたモデルです。
- `condensed_oak_log.json`: これは、親モデルを`block/cube_column`モデルとしたモデルです。

`condensed_oak_log_horizontal.json`ファイルの書式です。

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

このファイルで定義できる情報について詳しく知るためには、[Minecraft Wiki - Models (Block States)](https://minecraft.wiki/w/Tutorials/Models#Block_states)を参照してください。

:::

次に、ブロック状態を定義するファイルを作成します。このファイルが重要なファイルです。 柱ブロックには3つの軸に沿った置き方があるので、それらの違いに応じた設定します。

- `axis=x`: ブロックがX軸に沿って配置された場合、水平に傾けたモデルをXの正方向を向くように回転させます。
- `axis=y`: ブロックがY軸に沿って配置された場合、垂直のモデルであるcondensed_oak_logをそのまま使用します。
- `axis=z`: ブロックがZ軸に沿って配置された場合、水平に傾けたモデルをXの正方向を向くように回転させます。

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

他のブロック同様、各言語別の翻訳キーペアを作成し、2つのモデルのどちらかを親としたアイテムモデルを作成する必要があります。

![Example of Pillar block in-game](/assets/develop/blocks/blockstates_1.png)

## カスタムブロック状態 {#custom-block-states}

ブロックにもっとユニークな特徴を設けたい場合、自作のブロック状態を作成することが可能です。この時、バニラのプロパティを参考にすると効果的です。

この例では、`activated`というプロパティを作成します。これは、プレイヤーがブロックに向かって右クリックすると、ブロック状態が `activated=false`から `activated=true`になり、それに応じてテクスチャが変更されるような`boolean`型のプロパティです。

### プロパティの作成 {#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. この`appendProperties`メソッドは、`Block`クラスのものをオーバーライドしています。

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

また、作成した`activated`プロパティのデフォルトの状態を定義します。この定義は、`PrismarineLampBlock`ブロックのコンストラクタで行います。

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### プロパティの使用 {#using-the-property}

今回作成するブロックは、プレイヤーが右クリックを押すという動作に反応して、`activated`プロパティのbooleanが反転します。 We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### プロパティの可視化 {#visualizing-the-property}

ブロック状態のJSONファイルを作成する前に、`activated`と`deactivated`の両方の見た目を表すテクスチャと、モデル情報を用意しましょう。

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">Textures</DownloadEntry>

モデル情報・テクスチャ共に、condensed_dirtの方法と同じです。 完了出来たらブロック状態の定義に入ります。

新しいプロパティを作成したので、そのプロパティを適用するために、ブロック状態ファイルを更新します。

1つのブロックに複数のプロパティを定義する場合は、その組み合わせを考慮する必要があります。 例えば、`activated`と`axis`を持つ場合、6つの組み合わせに対して定義します。（`activated`は2つ、`axis`は3つの情報を持つため）。

この例では、`activated`だけの2通りなので、JSONファイルの書式は次のようになります。

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

このブロックはランプなので、`activated`が`true`の時に発光させる必要があります。 これは、ブロックを登録するときにコンストラクタに渡されるブロック設定(`AbstractBlock.Settings`)によって設定できます。

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

全て完了すると、最終的には以下のようになります。

<0>Prismarine Lamp Block in-game</0>
