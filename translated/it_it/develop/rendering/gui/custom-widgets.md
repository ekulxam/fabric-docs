---
title: Widget Personalizzati
description: Impara come creare widget personalizzati per la tua schermata.
authors:
  - IMB11
---

I Widget sono essenzialmente componenti di rendering containerizzate che possono essere aggiunti a una schermata, e i giocatori possono interagirci attraverso vari eventi come clic del mouse, pressione di tasti, e altro.

## Creare un Widget {#creating-a-widget}

There are multiple ways to create a widget class, such as extending `AbstractWidget`. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the `Renderable`, `GuiEventListener`, `NarrationSupplier`, and `NarratableEntry` interfaces:

- `Renderable` - for rendering - Required to register the widget to the screen via the `addRenderableWidget` method.
- `GuiEventListener` - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
- `NarrationSupplier` - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
- `NarratableEntry` - for selection - Required if you want to make your widget selectable using the <kbd>Tab</kbd> key - this also aids in accessibility.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

## Aggiungere il Widget alla Schermata {#adding-the-widget-to-the-screen}

Like all widgets, you need to add it to the screen using the `addRenderableWidget` method, which is provided by the `Screen` class. Assicurati di farlo nel metodo `init`.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomScreen.java)

![Widget personalizzato sullo schermo](/assets/develop/rendering/gui/custom-widget-example.png)

## Eventi di Widget {#widget-events}

You can handle events such as mouse clicks, key presses, by overriding the `mouseClicked`, `afterMouseAction`, `keyPressed`, and other methods.

For example, you can make the widget change color when it's hovered over by using the `isHovered()` method provided by the `AbstractWidget` class:

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

![Esempio Evento Hovering](/assets/develop/rendering/gui/custom-widget-events.webp)
