---
title: 绘制到 GUI
description: 学习如何使用 GuiGraphics 类来渲染各种形状、文本和纹理。
authors:
  - IMB11
---

This page assumes you've taken a look at the [Basic Rendering Concepts](./basic-concepts) page.

`GuiGraphics` 类是用于在游戏内渲染的主类。 它用于渲染形状、文本和纹理，并且如前所述，用于操作 `PoseStack` 和使用 `BufferBuilder`。

## 绘制形状 {#drawing-shapes}

使用 `GuiGraphics` 绘制**矩形的**形状十分容易。 如果想绘制三角形或其他非矩形的图形，需要使用 `BufferBuilder`。

### 绘制矩形 {#drawing-rectangles}

可以使用 `GuiGraphics.fill(...)` 方法绘制填充矩形。

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![矩形](/assets/develop/rendering/draw-context-rectangle.png)

### 绘制轮廓/边框 {#drawing-outlines-borders}

假设我们想勾勒出刚刚绘制的矩形的轮廓。 We can use the `GuiGraphics.renderOutline(...)` method to draw an outline.

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![带边框的矩形](/assets/develop/rendering/draw-context-rectangle-border.png)

### 绘制独立线条 {#drawing-individual-lines}

我们可以使用 `GuiGraphics.hLine(...)` 和 `DrawContext.vLine(...)` 方法来绘制线条。

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![线条](/assets/develop/rendering/draw-context-lines.png)

## 裁剪管理器 {#the-scissor-manager}

`GuiGraphics` 类内置了裁剪管理器。 可以用来把渲染裁剪为特定区域。 这个功能在绘制某些元素时十分有用，比如悬浮提示，或者其他不应该超出指定渲染区域的界面元素。

### 使用裁剪管理器 {#using-the-scissor-manager}

::: tip

Scissor regions can be nested! But make sure that you disable the scissor manager the same amount of times as you enabled it.

:::

To enable the scissor manager, simply use the `GuiGraphics.enableScissor(...)` method. Likewise, to disable the scissor manager, use the `GuiGraphics.disableScissor()` method.

@[code lang=java transcludeWith=:::4](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![Scissor region in action](/assets/develop/rendering/draw-context-scissor.png)

As you can see, even though we tell the game to render the gradient across the entire screen, it only renders within the scissor region.

## Drawing Textures {#drawing-textures}

There is no one "correct" way to draw textures onto a screen, as the `blit(...)` method has many different overloads. This section will go over the most common use cases.

### Drawing an Entire Texture {#drawing-an-entire-texture}

Generally, it's recommended that you use the overload that specifies the `textureWidth` and `textureHeight` parameters. This is because the `GuiGraphics` class will assume these values if you don't provide them, which can sometimes be wrong.

You will also need to specify which render pipeline which your texture will use. For basic textures, this will usually always be `RenderPipelines.GUI_TEXTURED`.

@[code lang=java transcludeWith=:::5](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![Drawing whole texture example](/assets/develop/rendering/draw-context-whole-texture.png)

### Drawing a Portion of a Texture {#drawing-a-portion-of-a-texture}

This is where `u` and `v` come in. These parameters specify the top-left corner of the texture to draw, and the `regionWidth` and `regionHeight` parameters specify the size of the portion of the texture to draw.

Let's take this texture as an example.

![Recipe Book Texture](/assets/develop/rendering/draw-context-recipe-book-background.png)

If we want to only draw a region that contains the magnifying glass, we can use the following `u`, `v`, `regionWidth` and `regionHeight` values:

@[code lang=java transcludeWith=:::6](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![Region Texture](/assets/develop/rendering/draw-context-region-texture.png)

## Drawing Text {#drawing-text}

The `GuiGraphics` class has various self-explanatory text rendering methods - for the sake of brevity, they will not be covered here.

Let's say we want to draw "Hello World" onto the screen. We can use the `GuiGraphics.drawString(...)` method to do this.

::: info

Minecraft 1.21.6 and above changes text color to be ARGB instead of RGB. Passing RGB values will cause your text to render transparent. Helper methods like `ARGB.opaque(...)` can be used to change RGB to ARGB while porting.

:::

@[code lang=java transcludeWith=:::7](@/reference/latest/src/client/java/com/example/docs/rendering/DrawContextExampleScreen.java)

![Drawing text](/assets/develop/rendering/draw-context-text.png)
