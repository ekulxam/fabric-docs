---
title: 自定义盔甲
description: 学习如何创建自己的盔甲集。
authors:
  - IMB11
---

盔甲增强玩家的防御，抵御来自生物和其他玩家的攻击。

## 创建盔甲材质类 {#creating-an-armor-materials-class}

从技术上讲，你不需要为你的盔甲材质设立专门的类别，但无论如何，对于你需要的静态场数量来说，这都是很好的做法。

For this example, we'll create a `GuiditeArmorMaterial` class to store our static fields.

### 基础耐久度 {#base-durability}

在创建我们的盔甲物品时，这个常量将在 `Item.Properties#maxDamage(int damageValue)` 方法中使用，当我们稍后创建 `ArmorMaterial` 对象时，它也是 `ArmorMaterial` 构造函数中的参数。

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

如果你难以确定平衡的基础耐久度，你可以参考 `ArmorMaterials` 界面中的原始盔甲材质实例。

### 装备资产注册表项 {#equipment-asset-registry-key}

尽管我们不必将我们的 `ArmorMaterial` 注册到任何注册表中，但将任何注册表项存储为常量通常是一种很好的做法，因为游戏将使用它来找到我们盔甲的相关纹理。

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

我们稍后会将其传递给 `ArmorMaterial` 构造函数。

### `ArmorMaterial` 实例 {#armormaterial-instance}

为了创建我们的材质，我们需要创建一个新的 `ArmorMaterial` 记录实例，这里将使用基础耐久度和材质注册表项常量。

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

`ArmorMaterial` 构造函数按照特定顺序接受以下参数：

| 参数                    | 描述                                                                   |
| --------------------- | -------------------------------------------------------------------- |
| `durability`          | 所有盔甲的基础耐久度，用于计算使用该材质的每个盔甲的总耐久度。 这应该是你之前创建的基础耐久度常量。                   |
| `defense`             | `EquipmentType`（代表每个护甲槽位的枚举）到整数值的映射，表示材质在相应护甲槽位中使用时的防御值。             |
| `enchantmentValue`    | 使用此材料制作的盔甲的“可附魔性”。                                                   |
| `equipSound`          | 当你装备使用该材质的盔甲时播放的声音事件的注册表项。 有关声音的更多信息，请参阅[自定义声音](../sounds/custom)页面。 |
| `toughness`           | 一个浮点值，代表盔甲材质的“韧性”属性——本质上代表盔甲吸收伤害的能力。                                 |
| `knockbackResistance` | 一个浮点值，代表盔甲材质赋予穿戴者的击退抗性。                                              |
| `repairIngredient`    | 一个物品标签，代表所有能够用在铁砧中修复此材质的盔甲物品的物品。                                     |
| `assetId`             | 一个 `EquipmentAsset` 注册表项，这应该是你之前创建的设备资产注册表项常量。                       |

我们对修复原材料标签引用定义如下：

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

如果你难以确定任何参数的值，你可以查阅在 `ArmorMaterials` 界面中可以找到的原始 `ArmorMaterial` 实例。

## 创建盔甲物品 {#creating-the-armor-items}

现在材料已经注册了，就可以在你的 `ModItems` 类中创建盔甲物品。

显然，盔甲集并不需要满足每种类型，可以让你的集只有靴或护腿等——原版的海龟壳头盔就是个例子，盔甲集缺了部分槽位。

不像 `ToolMaterial`，`ArmorMaterial` 并不储存物品的耐久度信息。 因此，在注册盔甲物品时需要手动将基础耐久度添加到盔甲物品的 `Item.Properties` 中。

这是通过将我们之前创建的 `BASE_DURABILITY` 常量传递到 `Item.Properties` 类中的 `maxDamage` 方法来实现的。

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

就像所有物品一样，要为这些物品创建翻译键。

## 纹理与模型 {#textures-and-models}

你需要为物品创建一组纹理，并为“人形/humanoid”实体（玩家、僵尸、骷髅等）穿戴的实际盔甲创建一组纹理。

### 物品纹理和模型 {#item-textures-and-model}

这些纹理和其他物品没有区别——必须创建纹理、创建一般的 generated 的物品模型，这在[创建你的第一个物品](./first-item#adding-a-texture-and-model)指南中有讲到。

例如，你可以使用下面的纹理和模型 JSON 作为参考。

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">物品纹理</DownloadEntry>

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
