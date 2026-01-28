---
title: Custom Creative Tabs
description: Learn how to create your own creative tab and add items to it.
authors:
  - CelDaemon
  - IMB11
---

Creative Tabs, also known as Item Groups, are the tabs in the creative inventory that store items. You can create your own creative tab to store your items in a separate tab. Это будет полезно, если ваш мод добавляет много предметов, и вы хотите хранить их в одном месте, чтобы игрок мог легко получить к ним доступ.

## Creating the Creative Tab {#creating-the-creative-tab}

Adding a creative tab is pretty simple. Simply create a new static final field in your items class to store the creative tab and a resource key for it. You can then use `FabricItemGroup.builder` to create the tab and add items to it:

@[code transcludeWith=:::9](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

@[code transcludeWith=:::\_12](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You should see a new tab is now in the creative inventory menu. Однако, она не переведена — вам необходимо добавить ключ перевода в файл с переводами как при переводе вашего первого предмета.

![Creative Tab without translation in creative menu](/assets/develop/items/itemgroups_0.png)

## Добавление ключа перевода {#adding-a-translation-key}

If you used `Component.translatable` for the `title` method of the creative tab builder, you will need to add the translation to your language file.

```json
{
  "itemGroup.example-mod": "Example Mod"
}
```

Now, as you can see, the creative tab should be correctly named:

![Fully completed creative tab with translation and items](/assets/develop/items/itemgroups_1.png)
