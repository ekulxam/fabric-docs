---
title: Armadura Personalizada
description: Aprender a criar seus próprios conjuntos de armadura.
authors:
  - IMB11
---

Armaduras fornecem ao jogador defesa aumentada contra ataques de criaturas e outros jogadores.

## Criando uma Classe de Materiais de Armadura {#creating-an-armor-materials-class}

Tecnicamente, você não precisa de uma calsse dedicada para o seu material de armadura, mas é uma boa prática de qualquer forma, considerando o número de campos estáticos que você precisará.

For this example, we'll create a `GuiditeArmorMaterial` class to store our static fields.

### Durabilidade Base {#base-durability}

This constant will be used in the `Item.Properties#maxDamage(int damageValue)` method when creating our armor items, it is also required as a parameter in the `ArmorMaterial` constructor when we create our `ArmorMaterial` object later.

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Se você estiver com dificuldades para determinar uma durabilidade base balanceada, você pode consultar as instâncias de materiais de armadura originais encontradas na interface `ArmorMaterials`.

### Chave de Registro de Recurso de Equipamento {#equipment-asset-registry-key}

Embora não precisemos registrar nosso `ArmorMaterial` em nenhum registro, geralmente é uma boa prática armazenar as chaves de registro como constantes, pois o jogo usará isso para encontrar as texturas relevantes para a nossa armadura.

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Passaremos isso para o construtor de `ArmorMaterial` mais tarde.

### Instância de `ArmorMaterial` {#armormaterial-instance}

Para criar nosso meterial, precisamos criar uma nova instância do record `ArmorMaterial`, as constantes de durabilidade base e chave de registro do material serão usadas aqui.

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

O construtor de `ArmorMaterial` aceita os seguintes parâmetros, nesta ordem específica:

| Parâmetro             | Descrição                                                                                                                                                                                                                                                |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `durability`          | A durabilidade base de todas as peças da armadura, é usada para calcular a durabilidade total de cada peça individual que usa este material. Esta deve ser a constate de durabilidade base que você criou anteriormente. |
| `defense`             | Um mapa de `EquipmentType` (um enum que representa cada slot de armadura) para um valor inteiro, que indica o valor de defesa do material quando usado no slot de armadura correspondente.                            |
| `enchantmentValue`    | O valor de "encantabilidade" de itens de armadura que usam este material.                                                                                                                                                                |
| `equipSound`          | Uma entrada de registro de um evento de som que é reproduzido quando você equipa uma peça de armadura que usa este material. Para mais informações sobre sons, confira a página [Sons Personalizados](../sounds/custom). |
| `toughness`           | Um valor float que representa o atributo de resistência do material de armadura — essencialmente o quão bem a armadura irá absorver dano.                                                                                                |
| `knockbackResistance` | Um valor float que representa a quantidade de redução da repulsão que o material da armadura concede ao usuário.                                                                                                                         |
| `repairIngredient`    | Uma tag de item que representa todos os itens que podem ser usados para consertar os itens de armadura deste material em uma bigorna.                                                                                                    |
| `assetId`             | Uma chave de registro de `EquipmentAsset`, esta deve ser a constante de chave de registro de recurso de equipamento que você criou anteriormente.                                                                                        |

We define the repair ingredient tag reference as follows:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Se você estiver com dificuldades para determinar valores para qualquer um dos parâmetros, você pode consultar as instâncias `ArmorMaterial` vanilla que podem ser encontradas na interface `ArmorMaterials`.

## Criando os Itens de Armadura {#creating-the-armor-items}

Agora que você registrou o material, pode criar os itens de armadura em sua classe `ModItems`:

Obviamente, um conjunto de armadura não precisa ter todos os tipos de peças, você pode ter um conjunto com apenas botas ou calças, etc. — o capacete de casco de tartaruga do jogo base é um bom exemplo de um conjunto de armadura com slots ausentes.

Diferente do `ToolMaterial`, o `ArmorMaterial` não armazena nenhuma informação sobre a durabilidade dos itens. For this reason the base durability needs to be manually added to the armor items' `Item.Properties` when registering them.

This is achieved by passing the `BASE_DURABILITY` constant we created previously into the `maxDamage` method in the `Item.Properties` class.

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

Assim como todos os itens, você deve criar chaves de tradução para eles também.

## Texturas e Modelos {#textures-and-models}

Você precisará criar um conjunto de texturas para os itens, e um conjunto de texturas para a armadura em si quando ela for vestida por uma entidade "humanoide" (jogadores, zumbis, esqueletos, etc).

### Texturas e Modelo dos Itens {#item-textures-and-model}

Essas texturas não diferem das de outros itens — você deve criar as texturas e um modelo de item gerado genérico, o que foi abordado no guia [Criando Seu Primeiro Item](./first-item#adding-a-texture-and-model).

Para fins de exemplo, você pode usar as seguintes texturas e o JSON de modelo como referência.

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">Texturas dos Itens</DownloadEntry>

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
