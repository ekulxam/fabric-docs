---
title: Armature Personalizzate
description: Impara come creare i tuoi set di armature personalizzati.
authors:
  - IMB11
---

Un'armatura fornisce al giocatore una difesa migliore contro attacchi di mob e di altri giocatori.

## Creare una Classe per un Materiale delle Armature {#creating-an-armor-materials-class}

Tecnicamente, non serve una classe apposita per il materiale della tua armatura, ma comunque è buona pratica data la quantità di attributi statici di cui avrai bisogno.

For this example, we'll create a `GuiditeArmorMaterial` class to store our static fields.

### Durabilità di Base {#base-durability}

This constant will be used in the `Item.Properties#maxDamage(int damageValue)` method when creating our armor items, it is also required as a parameter in the `ArmorMaterial` constructor when we create our `ArmorMaterial` object later.

@[code transcludeWith=:::base_durability](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Se fai fatica a determinare una durabilità di base bilanciata, puoi far riferimento alle istanze dei materiali di armature vanilla che trovi nell'interfaccia `ArmorMaterials`.

### Chiave di Registry per gli Asset Indossati {#equipment-asset-registry-key}

Anche se non dobbiamo registrare il nostro `ArmorMaterial` in alcuna registry, è in genere buona pratica memorizzare le chiavi di registry come costanti, poiché il gioco userà queste per trovare le texture adatte alla nostra armatura.

@[code transcludeWith=:::material_key](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Dopo passeremo questo al costruttore di `ArmorMaterial`.

### Istanza di `ArmorMaterial` {#armormaterial-instance}

Per creare il nostro materiale, dobbiamo creare una nuova istanza del record `ArmorMaterial`, in cui useremo le costanti di durabilità di base e chiave di registry del materiale.

@[code transcludeWith=:::guidite_armor_material](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Il costruttore di `ArmorMaterial` accetta i parametri seguenti, in questo ordine:

| Parametro             | Descrizione                                                                                                                                                                                                                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `durability`          | La durabilità di base di tutti i pezzi d'armatura, questa viene usata nel calcolare la durabilità totale di ciascun pezzo individuale di armatura che usa questo materiale. Questa dovrebbe essere la costante di durabilità di base che hai creato pocanzi. |
| `defense`             | Una mappa da `EquipmentType` (un enum che rappresenti ogni casella d'armatura) a un valore intero, che indica il valore di difesa del materiale se usato nella casella d'armaturà corrispondente.                                                         |
| `enchantmentValue`    | L'"incantabilità" degli oggetti d'armatura che usano questo materiale.                                                                                                                                                                                                       |
| `equipSound`          | Una voce di registry di un evento sonoro da riprodurre appena indossato un pezzo d'armatura che usa questo materiale. Per maggiori informazioni sui suoni, dai un'occhiata alla pagina [Suoni Personalizzati](../sounds/custom).                             |
| `toughness`           | Un valore float che rappresenti l'attributo "tenacità" del materiale d'armatura - in altre parole quanto l'armatura assorba bene il danno.                                                                                                                                   |
| `knockbackResistance` | Un valore float che rappresenti la quantità di resistenza al contraccolpo che il materiale d'armatura offre a chi la indossa.                                                                                                                                                |
| `repairIngredient`    | Un tag di oggetti che rappresenta tutti gli oggetti che possono essere usati per riparare gli oggetti dell'armatura di questo materiale in un'incudine.                                                                                                                      |
| `assetId`             | Una chiave di registry `EquipmentAsset`, questa dovrebbe essere la costante di chiave di registry per l'asset indossato creata in precedenza.                                                                                                                                |

Definiamo il riferimento di tag dell'ingrediente di riparo come segue:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Se fai fatica a determinare valori adatti per ciascuno di questi parametri, puoi consultare le istanze dei `ArmorMaterial` vanilla che trovi nell'interfaccia `ArmorMaterials`.

## Creare gli Oggetti dell'Armatura {#creating-the-armor-items}

Ora che hai registrato il materiale, puoi creare gli oggetti dell'armatura nella tua classe `ModItems`:

Ovviamente, un set di armatura non deve per forza essere completo, puoi avere un set con solo stivali, o solo gambiere... - il carapace di tartaruga vanilla è un buon esempio di un set di armatura con elementi mancanti.

A differenza di `ToolMaterial`, `ArmorMaterial` non memorizza alcuna informazione riguardo alla durabilità degli oggetti. For this reason the base durability needs to be manually added to the armor items' `Item.Properties` when registering them.

This is achieved by passing the `BASE_DURABILITY` constant we created previously into the `maxDamage` method in the `Item.Properties` class.

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You will also need to **add the items to a creative tab** if you want them to be accessible from the creative inventory.

Come per tutti gli oggetti, dovresti creare chiavi di traduzione anche per questi.

## Texture e Modelli {#textures-and-models}

Dovremo creare un insieme di texture per gli oggetti, e un insieme di texture per l'armatura effettiva quando indossata da un'entità "umanoide" (giocatori, zombie, scheletri...).

### Texture e Modello dell'Oggetto {#item-textures-and-model}

Queste texture non differiscono da quelle di altri oggetti - devi creare le texture, e creare un modello di oggetto generico generato - tutto questo è coperto dalla guida [Creare il Tuo Primo Oggetto](./first-item#adding-a-texture-and-model).

Come esempio, puoi usare le seguenti texture e modelli JSON come riferimento.

<DownloadEntry visualURL="/assets/develop/items/armor_0.png" downloadURL="/assets/develop/items/example_armor_item_textures.zip">Texture degli Oggetti</DownloadEntry>

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
