---
title: Alimenti
description: Impara come aggiungere una FoodComponent ad un oggetto per renderlo edibile, e come configurarlo.
authors:
  - IMB11
---

Gli alimenti sono un aspetto cruciale di sopravvivenza in Minecraft, per cui quando si creano oggetti edibili devi considerare l'uso del cibo con altri oggetti edibili.

A meno che tu non voglia creare una mod con oggetti troppo potenti, dovresti tenere in considerazione:

- Quanta fame aggiunge o toglie l'oggetto edibile?
- Quali effetti di pozione fornisce?
- È accessibile presto o tardi nel gioco?

## Aggiungere la Componente Alimento {#adding-the-food-component}

To add a food component to an item, we can pass it to the `Item.Properties` instance:

```java
new Item.Properties().food(new FoodProperties.Builder().build())
```

Per ora questo rende l'oggetto edibile, e nulla di più.

The `FoodProperties.Builder` class has some methods that allow you to modify what happens when a player eats your item:

| Metodo               | Descrizione                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------- |
| `nutrition`          | Imposta la quantità di punti fame che l'oggetto sazierà.                           |
| `saturationModifier` | Imposta la quantita di punti di saturazione che l'oggetto aggiungerà.              |
| `alwaysEdible`       | Permette al tuo oggetto di essere consumato indipendentemente dal livello di fame. |

When you've modified the builder to your liking, you can call the `build()` method to get the `FoodProperties`.

If you want to add status effects to the player when they eat your food, you will need to add a `Consumable` component alongside the `FoodProperties` component as seen in the following example:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Come nell'esempio della pagina [Creare il Tuo Primo Oggetto](./first-item), useremo la componente sopra:

@[code transcludeWith=:::poisonous_apple](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Questo rende l'oggetto:

- Sempre edibile, può essere mangiato indipendentemente dal livello di fame.
- Fornisce sempre Avvelenamento II per 6 secondi appena consumato.

<VideoPlayer src="/assets/develop/items/food_0.webm">Mangiare la Mela Avvelenata</VideoPlayer>
