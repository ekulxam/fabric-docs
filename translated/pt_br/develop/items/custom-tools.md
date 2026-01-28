---
title: Ferramentas e Armas
description: Aprenda a criar suas próprias ferramentas e a configurar suas propriedades.
authors:
  - IMB11
---

Ferramentas são essenciais para a sobrevivência e progressão, permitindo que os jogadores coletem recursos, construam edifícios e se defendam.

## Criando um Material de Ferramenta {#creating-a-tool-material}

Você pode criar um material de ferramenta instanciando um novo objeto `ToolMaterial` e armazenando-o em um campo que poderá ser usada para criar os itens de ferramenta que usam este material.

@[code transcludeWith=:::guidite_tool_material](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

O construtor de `ToolMaterial` aceita os seguintes parâmetros, nesta ordem específica:

| Parâmetro                 | Descrição                                                                                                                                                                                |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `incorrectBlocksForDrops` | Se um bloco estiver na tag incorrectBlocksForDrops, isso significa que ao usar uma ferramenta feita com este `ToolMaterial` neste bloco, ele não irá dropar nenhum item. |
| \`durability              | A durabilidade de todas as ferramentas que são deste `ToolMaterial`.                                                                                                     |
| `speed`                   | A velocidade de mineração das ferramentas que são deste `ToolMaterial`.                                                                                                  |
| `attackDamageBonus`       | O dano de ataque adicional que as ferramentas deste `ToolMaterial` terão.                                                                                                |
| `enchantmentValue`        | O valor de "encantablidade" das ferramentas que são deste `ToolMaterial`.                                                                                                |
| `repairItems`             | Quaisquer itens dentro desta tag podem ser usadas para consertar ferramentas deste `ToolMaterial` em uma bigorna.                                                        |

For this example, we will use the same repair item we will be using for armor. We define the tag reference as follows:

@[code transcludeWith=:::repair_tag](@/reference/latest/src/main/java/com/example/docs/item/armor/GuiditeArmorMaterial.java)

Se você estiver com dificuldade para determinar valores balanceados para qualquer um dos parâmetros numéricos, considere observar as constantes de material de ferramentas do jogo vanilla, como `ToolMaterial.STONE` ou `ToolMaterial.DIAMOND`.

## Criando Itens de Ferramenta {#creating-tool-items}

Usando a mesma funcão utilitária do guia [Criando Seu Primeiro Item](./first-item), você pode criar seus itens de ferramenta:

@[code transcludeWith=:::7](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Os dois valores float (`1f, 1f`) referem-se, respectivamente, ao dano de ataque da ferramenta e à velocidade de ataque da ferramenta.

Remember to add them to a creative tab if you want to access them from the creative inventory!

@[code transcludeWith=:::8](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Você também terá que adicionar uma textura, uma tradução de item e um modelo de item. No entanto, para o modelo de item, você vai querer usar o modelo `item/handheld` como seu pai, em vez do usual `item/generated`.

Para este exemplo, estarei usando o seguinte modelo e textura para o item "Espada de Guidite¨:

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/item/guidite_sword.json)

<DownloadEntry visualURL="/assets/develop/items/tools_0.png" downloadURL="/assets/develop/items/tools_0_small.png">Textura</DownloadEntry>

É basicamente isso! Se você entrar no jogo, deverá ver seus itens de ferramenta na aba de ferramentas do menu de inventário criativo.

![Itens prontos no inventário](/assets/develop/items/tools_1.png)
