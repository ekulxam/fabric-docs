---
title: 사용자 정의 위젯
description: 화면에 사용될 사용자 정의 위젯을 만드는 방법을 알아보세요.
authors:
  - IMB11
---

위젯은 일반적으로 화면에 추가되는 렌더링 컴포넌트의 컨테이너로, 키 입력, 마우스 클릭 등의 여러 이벤트를 통해 플레이어와 상호작용할 수 있습니다.

## 위젯 만들기 {#creating-a-widget}

There are multiple ways to create a widget class, such as extending `AbstractWidget`. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the `Renderable`, `GuiEventListener`, `NarrationSupplier`, and `NarratableEntry` interfaces:

- `Renderable` - for rendering - Required to register the widget to the screen via the `addRenderableWidget` method.
- `GuiEventListener` - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
- `NarrationSupplier` - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
- `NarratableEntry` - for selection - Required if you want to make your widget selectable using the <kbd>Tab</kbd> key - this also aids in accessibility.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

## 화면에 위젯 추가하기

Like all widgets, you need to add it to the screen using the `addRenderableWidget` method, which is provided by the `Screen` class. 이러한 작업은 `init` 메소드에서 수행되어야 합니다.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomScreen.java)

![화면에 표시되는 사용자 정의 위젯](/assets/develop/rendering/gui/custom-widget-example.png)

## 위젯 이벤트

You can handle events such as mouse clicks, key presses, by overriding the `mouseClicked`, `afterMouseAction`, `keyPressed`, and other methods.

For example, you can make the widget change color when it's hovered over by using the `isHovered()` method provided by the `AbstractWidget` class:

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/screens/CustomWidget.java)

![호버 이벤트 예시](/assets/develop/rendering/gui/custom-widget-events.webp)
