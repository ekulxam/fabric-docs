---
title: Alimentos
description: Aprenda como adicionar um FoodComponent a um item para torná-lo comestível, e como configurá-lo.
authors:
  - IMB11
---

A comida é um aspecto central no modo de sobrevivência do Minecraft, então, ao criar itens comestíveis, você precisa considerar o uso da comida com outros itens comestíveis.

A menos que você esteja criando um mod com itens superpoderosos, você deve considerar:

- Quanta fome seu item comestível adiciona ou remove.
- Quais efeitos de pocão ele concede?
- É acessível no início do jogo ou no final do jogo?

## Adicionando o Componente de Comida {#adding-the-food-component}

To add a food component to an item, we can pass it to the `Item.Properties` instance:

```java
new Item.Properties().food(new FoodProperties.Builder().build())
```

Por enquanto, isso apenas torna o item comestível e nada mais.

The `FoodProperties.Builder` class has some methods that allow you to modify what happens when a player eats your item:

| Método               | Descrição                                                                            |
| -------------------- | ------------------------------------------------------------------------------------ |
| `nutrition`          | Define a quantidade de pontos de fome que seu item restaurará.       |
| `saturationModifier` | Define a quantidade de pontos de saturação que seu item adicionará.  |
| `alwaysEdible`       | Permite que seu item seja comido independentemente do nível de fome. |

When you've modified the builder to your liking, you can call the `build()` method to get the `FoodProperties`.

If you want to add status effects to the player when they eat your food, you will need to add a `Consumable` component alongside the `FoodProperties` component as seen in the following example:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Similar ao exemplo na página [Criando Seu Primeiro Item](./first-item), usarei o componente acima:

@[code transcludeWith=:::poisonous_apple](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Isso torna o item:

- Sempre comestível, podendo ser consumido independentemente do nível de fome.
- Sempre causa Veneno II por 6 segundos ao ser comido.

<VideoPlayer src="/assets/develop/items/food_0.webm">Comendo a Maçã Envenenada</VideoPlayer>
