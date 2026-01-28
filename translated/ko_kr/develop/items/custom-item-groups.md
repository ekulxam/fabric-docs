---
title: Custom Creative Tabs
description: Learn how to create your own creative tab and add items to it.
authors:
  - CelDaemon
  - IMB11
---

Creative Tabs, also known as Item Groups, are the tabs in the creative inventory that store items. You can create your own creative tab to store your items in a separate tab. 모드에 수 많은 아이템이 추가되어 있고, 플레이어가 쉽게 접근할 수 있도록 정렬하고 싶다면 유용하게 사용할 수 있습니다.

## Creating the Creative Tab {#creating-the-creative-tab}

Adding a creative tab is pretty simple. Simply create a new static final field in your items class to store the creative tab and a resource key for it. You can then use `FabricItemGroup.builder` to create the tab and add items to it:

@[code transcludeWith=:::9](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

@[code transcludeWith=:::\_12](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

You should see a new tab is now in the creative inventory menu. 하지만, 아직 번역되지 않았기 떄문에, 첫 아이템을 번역했던 것처럼 언어 파일에 키를 추가해야 합니다.

![Creative Tab without translation in creative menu](/assets/develop/items/itemgroups_0.png)

## 번역 키 추가하기 {#adding-a-translation-key}

If you used `Component.translatable` for the `title` method of the creative tab builder, you will need to add the translation to your language file.

```json
{
  "itemGroup.example-mod": "Example Mod"
}
```

Now, as you can see, the creative tab should be correctly named:

![Fully completed creative tab with translation and items](/assets/develop/items/itemgroups_1.png)
