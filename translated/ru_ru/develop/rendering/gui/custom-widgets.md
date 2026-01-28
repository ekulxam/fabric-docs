---
title: Пользовательские виджеты
description: Узнайте, как создавать пользовательские виджеты для ваших экранов.
authors:
  - IMB11
---

Виджеты — это контейнеризированные компоненты рендеринга, которые можно добавить на экран, где игрок взаимодействует через различные события, например щелчки мыши или нажатия клавиш.

## Создание виджета {#creating-a-widget}

There are multiple ways to create a widget class, such as extending `AbstractWidget`. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the `Renderable`, `GuiEventListener`, `NarrationSupplier`, and `NarratableEntry` interfaces:

- `Renderable` - for rendering - Required to register the widget to the screen via the `addRenderableWidget` method.
- `GuiEventListener` - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
- `NarrationSupplier` - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
- `NarratableEntry` - for selection - Required if you want to make your widget selectable using the <kbd>Tab</kbd> key - this also aids in accessibility.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

## Добавление виджета на экран {#adding-the-widget-to-the-screen}

Like all widgets, you need to add it to the screen using the `addRenderableWidget` method, which is provided by the `Screen` class. Обязательно сделайте это в методе `init`.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomScreen.java)

![Пользовательский виджет на экране](/assets/develop/rendering/gui/custom-widget-example.png)

## События виджета {#widget-events}

You can handle events such as mouse clicks, key presses, by overriding the `mouseClicked`, `afterMouseAction`, `keyPressed`, and other methods.

For example, you can make the widget change color when it's hovered over by using the `isHovered()` method provided by the `AbstractWidget` class:

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

![Пример события наведения](/assets/develop/rendering/gui/custom-widget-events.webp)
