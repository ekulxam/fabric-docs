---
title: États de Bloc
description: Découvrez pourquoi les états de bloc sont un excellent moyen d'ajouter des fonctionnalités visuelles à vos blocs.
authors:
  - IMB11
---

Un état de bloc est une donnée attachée à un bloc unique dans le monde de Minecraft, contenant des informations sur le bloc sous forme de propriétés. Voici quelques exemples de propriétés que Minecraft vanilla stocke dans les états de bloc :

- Rotation : Principalement utilisée pour les rondins et autres blocs naturels.
- Activé : Utilisé intensivement dans les dispositifs redstone et les blocs comme le four ou le fumoir.
- Âge : Utilisé pour les cultures, les plantes, les jeunes arbres, le varech, etc.

Vous pouvez probablement voir pourquoi ils sont utiles : ils évitent le besoin de stocker des données NBT dans une entité de bloc, réduisant ainsi la taille du monde et prévenant les problèmes de TPS !

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## Exemple : Bloc de Pilier {#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

Minecraft possède déjà certaines classes personnalisées qui permettent de créer rapidement certains types de blocs. Cet exemple explique la création d'un bloc avec la propriété axis en créant un bloc "Rondin de Chêne Condensé".

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

Les blocs de pilier ont deux textures, le dessus et le côté, et utilisent le modèle 'block/cube_column'.

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">Textures</DownloadEntry>

Puisque le bloc de pilier a deux positions, horizontale et verticale, nous devrons créer deux fichiers de modèle séparés :

- 'condensed_oak_log_horizontal.json' qui étend le modèle 'block/cube_column_horizontal'.
- 'condensed_oak_log.json' qui étend le modèle 'block/cube_column'.

Un exemple du fichier 'condensed_oak_log_horizontal.json' :

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

Pour une vue plus approfondie de tous les modificateurs disponibles dans les fichiers d'état de bloc, consultez la page [Minecraft Wiki - Modèles (États de Bloc)](https://fr.minecraft.wiki/w/Mod%C3%A8le) page.

:::

Ensuite, nous avons besoin de créer un fichier de status de bloc, c'est là où la magie opère. Les blocs de piliers ont trois axes, donc nous allons utiliser un modèle spécifique pour chaque situation ci-dessous:

- 'axis=x' : Lorsque le bloc est placé le long de l'axe X, nous ferons pivoter le modèle pour qu'il soit orienté dans la direction positive de l'axe X.
- 'axis=y' : Lorsque le bloc est placé le long de l'axe Y, nous utiliserons le modèle vertical normal.
- 'axis=z' : Lorsque le bloc est placé le long de l'axe Z, nous ferons pivoter le modèle pour qu'il soit orienté dans la direction positive de l'axe X.

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

Comme toujours, vous devrez créer une traduction pour votre bloc, ainsi qu'un modèle d'objet qui hérite de l'un des deux modèles.

![Exemple de bloc de pilier en jeu](/assets/develop/blocks/blockstates_1.png)

## États de Bloc Personnalisés {#custom-block-states}

Les états de bloc personnalisés sont idéaux si votre bloc possède des propriétés uniques. Parfois, vous pourriez découvrir que votre bloc peut réutiliser des propriétés vanilla.

Cet exemple va créer une propriété booléenne unique appelée 'activated'. Lorsqu'un joueur fait un clic droit sur le bloc, celui-ci passera de 'activated=false' à 'activated=true', changeant ainsi sa texture en conséquence.

### Création de la Propriété {#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. Vous devrez remplacer la méthode pour accéder au constructeur :

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Vous devrez également définir un état par défaut pour la propriété 'activated' dans le constructeur de votre bloc personnalisé.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Utilisation de la Propriété {#using-the-property}

Cet exemple inverse la propriété booléenne 'activated' lorsque le joueur interagit avec le bloc. We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Visualisation de la Propriété {#visualizing-the-property}

Avant de créer le fichier d'état de bloc, vous devrez fournir des textures pour les deux états du bloc, activé et désactivé, ainsi que le modèle de bloc.

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">Textures</DownloadEntry>

Utilisez vos connaissances sur les modèles de bloc pour créer deux modèles pour le bloc : un pour l'état activé et un pour l'état désactivé. Une fois que vous avez fait cela, vous pouvez commencer à créer le fichier d'état de bloc.

Puisque vous avez créé une nouvelle propriété, vous devrez mettre à jour le fichier d'état de bloc pour le bloc afin de prendre en compte cette propriété.

Si vous avez plusieurs propriétés sur un bloc, vous devrez prendre en compte toutes les combinaisons possibles. Par exemple, 'activated' et 'axis' conduiraient à 6 combinaisons (deux valeurs possibles pour 'activated' et trois valeurs possibles pour 'axis').

Puisque ce bloc n'a que deux variantes possibles, car il n'a qu'une seule propriété ('activated'), le JSON de l'état de bloc ressemblera à ceci :

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

Puisque le bloc exemple est une lampe, nous devons également le faire émettre de la lumière lorsque la propriété 'activated' est vraie. Cela peut être fait via les paramètres de bloc passés au constructeur lors de l'enregistrement du bloc.

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

Une fois que vous avez terminé, le résultat final devrait ressembler à ceci :

<VideoPlayer src="/assets/develop/blocks/blockstates_3.webm">Lampe de Prismarine en jeu</VideoPlayer>
