---
title: Собственная броня
description: Вы узнаете как создавать собственные комплекты брони.
authors:
  - IMB11
---

Броня дает игроку повышенную защиту от атак мобов и других игроков.

## Создание класса для материалов брони {#creating-an-armor-materials-class}

Технически вам не нужен специальный класс для материала вашей брони, но в любом случае это хорошая практика, учитывая количество статических полей, которые вам понадобятся.

В этом примере мы создадим класс `GuiditeArmorMaterial` для хранения наших статических полей.

### Базовая прочность {#base-durability}

This constant will be used in the `Item.Properties#maxDamage(int damageValue)` method when creating our armor items, it is also required as a parameter in the `ArmorMaterial` constructor when we create our `ArmorMaterial` object later.

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Если вы пытаетесь определить сбалансированную базовую прочность, вы можете обратиться к экземплярам материалов ванильных доспехов, которые можно найти в интерфейсе `ArmorMaterials`.

### Ключ реестра ассетов оборудования {#equipment-asset-registry-key}

Хотя нам не нужно регистрировать наш `ArmorMaterial` в каких-либо реестрах, игра будет использовать его для поиска соответствующих текстур для нашей брони. Обычно хорошей практикой является хранить любые ключи реестра как константы.

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Мы передадим это конструктору `ArmorMaterial` позже.

### Экземпляр `ArmorMaterial` {#armormaterial-instance}

Чтобы создать наш материал, нам нужно создать новый экземпляр записи `ArmorMaterial`, здесь будут использоваться константы базового ключа реестра прочности и материала.

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Конструктор `ArmorMaterial` принимает следующие параметры в указанном порядке:

| Параметр              | Описание                                                                                                                                                                                                                                                                    |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `durability`          | Базовая прочность всех частей брони, которая используется при расчете общей прочности каждого отдельного частей брони, в котором используется этот материал. Это должна быть базовая константа прочности, которую вы создали ранее.         |
| `defense`             | Сопоставление `EquipmentType` (перечисление, представляющее каждый слот для брони) с целым значением, которое указывает защитную ценность материала при использовании в соответствующем слоте для брони.                                 |
| `enchantmentValue`    | "Зачаровываемость" предметов брони, в которых используется этот материал.                                                                                                                                                                                   |
| `equipSound`          | Запись в реестре звукового события, которое воспроизводится, когда вы надеваете броню, в которой используется этот материал. Для получения дополнительной информации о звуках посетите страницу [Пользовательские звуки](../sounds/custom). |
| `toughness`           | Плавающее значение, которое отражает "прочность" материала брони - по сути, то, насколько хорошо броня будет поглощать урон.                                                                                                                                |
| `knockbackResistance` | Плавающее значение, представляющее собой величину сопротивления удару, которую материал брони обеспечивает владельцу.                                                                                                                                       |
| `repairIngredient`    | Метка предмета, представляющая все предметы, которые могут быть использованы для ремонта доспехов из этого материала в наковальне.                                                                                                                          |
| `assetId`             | Ключ реестра `EquipmentAsset`. Это должна быть константа ключа реестра активов оборудования, созданная вами ранее.                                                                                                                          |

Мы определяем ссылку на тег ингредиента ремонта следующим образом:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Если вам трудно определить значения для любого из параметров, вы можете обратиться к экземплярам `ArmorMaterial`, которые можно найти в интерфейсе `ArmorMaterials`.

## Создание предметов брони {#creating-the-armor-items}

Теперь, когда вы зарегистрировали материал, вы можете создать предметы брони в своем классе `ModItems`:

Очевидно, что комплект брони необязательно должен включать все типы предметов, вы можете иметь комплект, состоящий только из ботинок, штанов и т.д. — классический черепаший панцирь — хороший пример комплекта брони с отсутствующими слотами.

В отличие от `ToolMaterial`, `ArmorMaterial` не хранит никакой информации о прочности предметов. For this reason the base durability needs to be manually added to the armor items' `Item.Properties` when registering them.

This is achieved by passing the `BASE_DURABILITY` constant we created previously into the `maxDamage` method in the `Item.Properties` class.

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

Как и для всех элементов, для них также следует создать ключи перевода.

## Текстуры и Модели {#textures-and-models}

Вам нужно будет создать набор текстур для предметов и набор текстур для самой брони, когда ее носят "гуманоидные" существа (игроки, зомби, скелеты и т.д.).

### Текстуры и модели предметов {#item-textures-and-model}

Эти текстуры ничем не отличаются от других предметов — вам необходимо создать текстуры и создать общую сгенерированную модель предмета, что было рассмотрено в руководстве [Создание вашего первого предмета](./first-item#adding-a-texture-and-model).

В качестве примера вы можете использовать следующие текстуры и модель JSON в качестве справочного материала.

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">Текстуры предметов</DownloadEntry>

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
