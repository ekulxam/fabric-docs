---
title: Estados do bloco
description: Saiba o porque os estados dos blocos são uma boa maneira de adicionar funcionalidade visual para seus blocos.
authors:
  - IMB11
---

Um estado do bloco é um pedaço de dado anexado a um bloco singular no mundo do Minecraft contendo a informação na forma de propriedades — alguns exemplos de propriedades vanilla guardado nos estados:

- Rotação: Usado principalmente em troncos ou outros blocos naturais.
- Ativado: Fortemente usado em componentes de redstone, e em blocos como a fornalha ou o defumador.
- Idade (Age): usado em plantações, plantas, mudas, algas etc.

Você provavelmente pode ver porque eles são úteis — eles evitam que você armazene dados NBT em um bloco-entidade — reduzindo o tamanho do mundo e prevenindo problemas de TPS!

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## Exemplo: Pilar {#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

Minecraft já tem algumas classes personalizadas que permitem você criar certos tipos de blocos — este exemplo vai pela criação do bloco com a propriedade `axis` por criar um bloco como o "Tronco de carvalho condensado".

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

Blocos de pilar tem duas texturas, topo e lado — eles usam o modelo `block/cube_column`.

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">Texturas</DownloadEntry>

Como o bloco de pilar tem duas posições, horizontal e vertical, devemos fazer dois arquivos de modelos separados:

- `condensed_oak_log_horizontal.json` que estende o modelo `block/cube_column_horizontal`.
- `condensed_oak_log.json` que estende o modelo `block/cube_column`.

Um exemplo do arquivo `condensed_oak_log_horizontal.json`:

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

Para uma busca mais profunda nos modificadores disponíveis no arquivo de estados do bloco, veja a página [Minecraft Wiki - Models (Block States)] (https://minecraft.wiki/w/Tutorials/Models#Block_states).

:::

A seguir, precisamos criar um Estado de Bloco, aonde a mágica acontece. Pilares tem três eixos, então nós utilizamos modelos específicos para as seguintes situações:

- `axis=x` - quando o bloco é colocado junto ao axis X, vamos rotacionar o modelo para virar a direção X positiva.
- `axis=y` - Quando o bloco é colocado junto ao axis Y, usaremos o modelo vertical normal.
- `axis=z` - Quando o bloco é colocado junto ao axis Z, iremos rotacionar o modelo para virar a direção X positiva.

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

Como sempre, você deverá criar uma tradução para seu bloco, o modelo de um item que os parentes são qualquer um dos dois modelos.

![Exemplo de um bloco de pilar dentro do jogo](/assets/develop/blocks/blockstates_1.png)

## Estados do bloco personalizado {#custom-block-states}

Customizar Estado de Bloco é bom se seu bloco tem propriedades únicas — às vezes você pode encontrar um modo do bloco reutilizar propriedades do vanilla.

Esse exemplo criará uma única propriedade booliana chamada `activated` quando um jogador clica com o botão direito no bloco, ele irá de `activated=false` para `activated=ture` — mudando sua estrutura de acordo.

### Criando a Propriedade {#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. Você precisará substituir o método para acessar a construção:

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Você também terá que escolher um estado padrão para a propriedade `activated` na construção do seu bloco personalizado.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Usando a Propriedade {#using-the-property}

Esse exemplo inverte a propriedade booliana `activated` quando o jogador interage com o bloco. We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Visualizando a Propriedade {#visualizing-the-property}

Antes de criar o arquivo de Estado de Bloco, você precisará prover textura para ambos estados do bloco ativo e desativo, também como o modelo do bloco.

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">Texturas</DownloadEntry>

Usando seu conhecimento de modelos de blocos para criar dois modelos para o bloco: um quando estiver estado ativado e outro para o estado desativado. Uma vez feito, você pode começar criando o arquivo de Estado de Bloco.

Desde que você criou uma propriedade, você irá precisar atualizar o arquivo de Estado de Bloco para o bloco contar com aquela propriedade.

Se você tem múltiplas propriedades no bloco, você precisará contar com todas as possíveis combinações. Por exemplo, `activated` e `axis` levaria a 6 possíveis combinações (dois possíveis valores para `activated` e três possíveis valores para `axis`).

Desde que esse bloco tenha duas diferentes variantes, só é possível uma propriedade (`activated`), o Estado de Bloco JSON irá parecer algo como:

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

Desde que o bloco de exemplo é uma lâmpada, nós também precisamos fazer que ele emita luz quando a propriedade `activated` é verdadeira. Isso pode ser feito através das configurações de bloco passadas a construção quando registramos o bloco.

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

Assim que você completou tudo, o resultado final deve se parecer algo como:

<VideoPlayer src="/assets/develop/blocks/blockstates_3.webm">Bloco de Lanterna do Mar em jogo</VideoPlayer>
