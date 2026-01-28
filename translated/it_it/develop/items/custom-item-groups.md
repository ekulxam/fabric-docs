---
title: Custom Creative Tabs
description: Learn how to create your own creative tab and add items to it.
authors:
  - CelDaemon
  - IMB11
---

Creative Tabs, also known as Item Groups, are the tabs in the creative inventory that store items. You can create your own creative tab to store your items in a separate tab. Questo è piuttosto utile se la tua mod aggiunge molti oggetti e vuoi tenerli organizzati in una sola posizione per facilitarne l'accesso per i giocatori.

## Creating the Creative Tab {#creating-the-creative-tab}

Adding a creative tab is pretty simple. Simply create a new static final field in your items class to store the creative tab and a resource key for it. You can then use `FabricItemGroup.builder` to create the tab and add items to it:

@[code transcludeWith=:::9](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

@[code transcludeWith=:::\_12](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You should see a new tab is now in the creative inventory menu. Tuttavia, è rimasto senza traduzione - devi aggiungere una chiave al tuo file di traduzioni - come quando hai tradotto il tuo primo oggetto.

![Creative Tab without translation in creative menu](/assets/develop/items/itemgroups_0.png)

## Aggiungere una Chiave di Traduzione {#adding-a-translation-key}

If you used `Component.translatable` for the `title` method of the creative tab builder, you will need to add the translation to your language file.

```json
{
  "itemGroup.example-mod": "Example Mod"
}
```

Now, as you can see, the creative tab should be correctly named:

![Fully completed creative tab with translation and items](/assets/develop/items/itemgroups_1.png)
