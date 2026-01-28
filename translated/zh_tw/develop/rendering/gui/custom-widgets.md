---
title: 自訂畫面元件
description: 了解如何為你的畫面建立自訂畫面元件。
authors:
  - IMB11
---

畫面元件就是包裝過的圖形，他們可以被加到畫面上，也可以讀取來自玩家的互動（例如：點選，按鍵等）。

## 新增一個畫面元件 {#creating-a-widget}

There are multiple ways to create a widget class, such as extending `AbstractWidget`. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the `Renderable`, `GuiEventListener`, `NarrationSupplier`, and `NarratableEntry` interfaces:

- `Renderable` - for rendering - Required to register the widget to the screen via the `addRenderableWidget` method.
- `GuiEventListener` - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
- `NarrationSupplier` - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
- `NarratableEntry` - for selection - Required if you want to make your widget selectable using the <kbd>Tab</kbd> key - this also aids in accessibility.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

## 讓元件顯示在畫面上 {#adding-the-widget-to-the-screen}

Like all widgets, you need to add it to the screen using the `addRenderableWidget` method, which is provided by the `Screen` class. 請確保這是在 `init` 函式裡執行的。

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomScreen.java)

![顯示自訂元件在畫面上](/assets/develop/rendering/gui/custom-widget-example.png)

## 畫面元件事件 {#widget-events}

You can handle events such as mouse clicks, key presses, by overriding the `mouseClicked`, `afterMouseAction`, `keyPressed`, and other methods.

For example, you can make the widget change color when it's hovered over by using the `isHovered()` method provided by the `AbstractWidget` class:

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

![滑鼠停留事件範例](/assets/develop/rendering/gui/custom-widget-events.webp)
