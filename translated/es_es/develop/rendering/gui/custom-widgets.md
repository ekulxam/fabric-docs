---
title: Widgets Personalizados
description: Aprende a crear widgets personalizados para tus pantallas o menús.
authors:
  - IMB11
---

Los widgets son esencialmente componentes renderizados que pueden ser agregados a una pantalla, y que pueden ser usados por el jugador mediante varios eventos como un click del mouse, presionar una tecla, y más.

## Crear un Widget

There are multiple ways to create a widget class, such as extending `AbstractWidget`. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the `Renderable`, `GuiEventListener`, `NarrationSupplier`, and `NarratableEntry` interfaces:

- `Renderable` - for rendering - Required to register the widget to the screen via the `addRenderableWidget` method.
- `GuiEventListener` - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
- `NarrationSupplier` - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
- `NarratableEntry` - for selection - Required if you want to make your widget selectable using the <kbd>Tab</kbd> key - this also aids in accessibility.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

## Agregar el Widget a La Pantalla

Like all widgets, you need to add it to the screen using the `addRenderableWidget` method, which is provided by the `Screen` class. Asegúrate de hacerlo en el método `init`.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomScreen.java)

![Widget personalizado en la pantalla](/assets/develop/rendering/gui/custom-widget-example.png)

## Eventos de Widget

You can handle events such as mouse clicks, key presses, by overriding the `mouseClicked`, `afterMouseAction`, `keyPressed`, and other methods.

For example, you can make the widget change color when it's hovered over by using the `isHovered()` method provided by the `AbstractWidget` class:

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

![Ejemplo de Evento de Mouse Flotando](/assets/develop/rendering/gui/custom-widget-events.webp)
