---
title: 方塊狀態
description: 學習為什麼方塊狀態是一個向你的方塊添加可視化功能的好方法。
authors:
  - IMB11
---

方塊狀態是附加到 Minecraft 世界中的單個方塊上的一段數據，包含屬性形式的方塊塊信息——原版存儲在方塊狀態中的屬性的一些示例：

- Rotation：主要用於原木方塊和其他自然方塊中。
- Activated：主要用於紅石裝置方塊和類似於熔爐、煙薰爐的方塊中。
- Age：用於農作物、植物、樹苗、海帶等方塊中使用。

你可能看出了為什麼方塊狀態有用——避免了在方塊實體中存儲 NBT 數據的需要——這既減小了世界大小，也防止產生 TPS 問題！

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## 示例：柱方塊{#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

Minecraft 已經有些自定義的類，允許你快速創建特定類型的方塊——這個例子會通過創建“Condensed Oak Log”方塊來帶你創建帶有 `axis` 屬性的方塊。

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

柱方塊有兩個紋理，頂部（`top`）和側面（`side`），使用 `block/cube_column` 模型。

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">紋理</DownloadEntry>

由於柱方塊有兩個位置，水平和垂直，我們需要創建兩個單獨的模型文件：

- `condensed_oak_log_horizontal.json`，繼承 `block/cube_column_horizontal` 模型。
- `condensed_oak_log.json`，繼承 `block/cube_column` 模型。

`condensed_oak_log_horizontal.json` 文件的示例：

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

更加深入瞭解方塊狀態文件中可用的所有修飾器，可看看 [Minecraft Wiki - 模型（方塊狀態）](https://zh.minecraft.wiki/w/Tutorial:模型/方塊狀態)頁面。

:::

接下來，我們需要創建一個方塊狀態文件，這就是神奇的事情發生的地方。 柱型方塊有三個軸，因此我們將針對以下情況使用特定模型：

- `axis=x` - 方塊沿 X 軸放置時，旋轉模型以朝向正 X 方向。
- `axis=y` - 方塊沿 Y 軸旋轉時，使用正常的垂直模型。
- `axis=z` - 方塊沿Z 軸放置時，旋轉模型以朝向正 X 方向。

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

同樣，需要為你的方塊創建翻譯，以及繼承了這兩個模型中的任意一個的物品模型。

![遊戲內的柱方塊的示例](/assets/develop/blocks/blockstates_1.png)

## 自定義方塊狀態{#custom-block-states}

如果你的方塊有獨特的屬性，那麼自定義方塊狀態會非常不錯——有時你會發現你的方塊可以復用原版的屬性。

這個例子會創建一個叫做 `activated` 的獨特屬性——玩家右鍵單擊方塊時，方塊會由 `activated=false` 變成 `activated-true` 並相應改變紋理。

### 創建屬性{#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. 需要覆蓋此方法以訪問 builder：

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

你還需要在你的自定義方塊的構造函數中，設置 `activated` 屬性的默認狀態。

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### 使用屬性{#using-the-property}

這個例子會在玩家與方塊交互時，翻轉 `activated` 屬性的布爾值。 We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### 視覺呈現屬性{#visualizing-the-property}

創建方塊狀態前，我們需要為方塊的激活的和未激活的狀態都提供紋理，以及方塊模型。

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">紋理</DownloadEntry>

用你的方塊模型知識，創建方塊的兩個模型：一個用於激活的狀態，一個用於未激活的狀態。 完成後，就可以開始創建方塊狀態文件了。

因為創建了新的屬性，所以需要為方塊更新方塊狀態文件以使用那個屬性。

如果方塊有多個屬性，那麼會需要包含所有可能的組合。 例如，`activated` 和 `axis` 可能就會導致 6 個組合（`activated` 有兩個可能的值，`axis` 有三個可能的值）。

因為方塊只有一個屬性（`activated`），只有兩個變種，所以方塊狀態 JSON 看起來應該像這樣：

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

因為這個示例方塊是燈，所以還需要讓它在 `activated` 屬性為 true 時發光。 可以通過在註冊方塊時傳入構造器的 block settings 來完成。

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

一切完成後，最終的結果應該看起來像這樣：

<VideoPlayer src="/assets/develop/blocks/blockstates_3.webm">遊戲內海晶燈方塊</VideoPlayer>
