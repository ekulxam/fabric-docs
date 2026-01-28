---
title: 사용자 지정 갑옷
description: 갑옷을 추가하는 방법에 대해 알아보세요.
authors:
  - IMB11
---

갑옷은 몹과 다른 플레이어의 공격으로부터 플레이어에게 추가적인 방어력을 제공합니다.

## 갑옷 재료 클래스 만들기 {#creating-an-armor-materials-class}

기술적으로, 갑옷 재료를 위해 새로운 클래스를 필수적으로 만들어야 하는 것은 아니지만, 필요한 정적 필드를 고려했을 때 좋은 습관이라고 할 수 있습니다.

For this example, we'll create a `GuiditeArmorMaterial` class to store our static fields.

### 기본 내구도 {#base-durability}

This constant will be used in the `Item.Properties#maxDamage(int damageValue)` method when creating our armor items, it is also required as a parameter in the `ArmorMaterial` constructor when we create our `ArmorMaterial` object later.

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

균형 잡힌 내구성을 결정하는 것이 어렵다면, `ArmorMaterials` 인터페이스를 통해 찾을 수 있는 바닐라 갑옷 재료 인스턴스를 참고해 보아도 됩니다.

### 장비 어셋 레지스트리 키 {#equipment-asset-registry-key}

굳이 `ArmorMaterial`을 다른 레지스트리에 등록할 필요까진 없지만, 레지스트리 키를 상수 필드로 가지는 것은 좋은 습관입니다. 게임이 갑옷에 맞는 텍스처를 찾을 때 사용할 수 있기 때문입니다.

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

레지스트리 키는 나중에 `ArmorMaterial` 생성자에 입력할 것입니다.

### `ArmorMaterial` 인스턴스 {#armormaterial-instance}

재료를 추가하려면, 새로운 `ArmorMaterial` 레코드 인스턴스를 생성해야 합니다. 기본 내구도와 재료 레지스트리 키가 여기에 입력됩니다.

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

`ArmorMaterial` 생성자는 순서대로 다음 매개변수를 입력받습니다:

| 매개 변수                 | 설명                                                                                                                                                       |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `durability`          | 이 재료로 만들어진 갑옷의 기본 내구도. 이후 각 방어구의 총 내구도를 계산할 때 사용됩니다. 이전에 만들어 두었던 기본 내구도 상수가 입력되어야 합니다.                   |
| `defense`             | `EquipmentType` 열거형(방어구 슬롯을 지정할 때 사용합니다)과 정수 값이 서로 연결된 맵. 해당 방어구 슬롯에 갑옷을 착용했을 때 증가할 방어력을 설정할 때 사용합니다. |
| `enchantmentValue`    | 이 재료로 만들어진 갑옷 아이템의 "마법 부여성."                                                                                                             |
| `equipSound`          | 이 재료로 만들어진 갑옷을 착용했을 때 재생될 소리 이벤트의 레지스트리 항목. 소리에 대한 자세한 내용은, [사용자 지정 소리](../sounds/custom)를 참조하십시오.                       |
| `toughness`           | 갑옷 재료의 "방어 강도" 속성을 나타내는 실수 값. 사실상 갑옷이 흡수하는 피해량입니다.                                                                       |
| `knockbackResistance` | 이 재료로 만들어진 갑옷을 착용했을 때 증가할 밀치기 저항 실수 값.                                                                                                   |
| `repairIngredient`    | 모루에서 이 재료로 만들어진 갑옷을 수리할 때 사용할 수 있는 아이템의 태그.                                                                                              |
| `assetId`             | 레지스트리 키는 나중에 `ArmorMaterial` 생성자에 입력할 것입니다.                                                                                              |

We define the repair ingredient tag reference as follows:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

매개 변수의 값을 결정하는 것이 어렵다면, 위에서 언급한 것 처럼 `ArmorMaterials` 인터페이스에 있는 바닐라 `ArmorMaterial` 인스턴스를 참고할 수 있습니다.

## 갑옷 아이템 추가하기 {#creating-the-armor-items}

이제 재료를 등록했으니, `ModItems` 클래스에 갑옷 아이템을 추가할 수 있습니다:

물론, 모든 종류의 갑옷을 만들 필요는 없습니다. 부츠나 레깅스만 있어도 되죠. 거북 등딱지는 투구만 있기 때문에, 이러한 상황의 좋은 예시라고 할 수 있습니다.

`ToolMaterial`과는 다르게, `ArmorMaterial`은 아이템의 내구도에 대한 정보는 저장되지 않습니다. For this reason the base durability needs to be manually added to the armor items' `Item.Properties` when registering them.

This is achieved by passing the `BASE_DURABILITY` constant we created previously into the `maxDamage` method in the `Item.Properties` class.

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

모든 아이템이 그렇듯, 번역 키도 추가해야 합니다.

## 텍스처와 모델 {#textures-and-models}

아이템에 사용할 텍스쳐와 더불어, "인간형" 개체(플레이어, 좀비, 스켈레톤 등)가 갑옷을 착용했을 때 보여질 실제 텍스쳐도 추가해야 합니다.

### 아이템 텍스처와 모델 {#item-textures-and-model}

이 텍스처는 다른 아이템과 마찬가지로, [첫 번째 아이템 만들기](./first-item#adding-a-texture-and-model)에서 설명한 것 처럼 텍스처를 만들고, 기본 아이템 모델을 만들어야 합니다.

예제에서는, 아래 텍스처와 JSON 모델을 사용하겠습니다.

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">아이템 텍스처</DownloadEntry>

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
