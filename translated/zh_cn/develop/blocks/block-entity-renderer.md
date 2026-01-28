---
title: 方块实体渲染器
description: 了解如何使用方块实体渲染器为渲染增色。
authors:
  - natri0
---

有的时候，用 Minecraft 自带的模型格式和渲染器并不足够。 如果需要为你的方块的视觉效果添加动态渲染，则需要使用 `BlockEntityRenderer`。

举个例子，让我们来制作一个在 [方块实体](../blocks/block-entities) 文章中出现的 Counter Block，这个方块会在方块顶部显示点击次数。

## 创建一个 BlockEntityRenderer {#creating-a-blockentityrenderer}

方块实体渲染使用提交/渲染系统，首先将渲染对象所需的数据提交到屏幕，然后游戏使用其提交状态渲染对象。

在为 `CounterBlockEntity` 创建 `BlockEntityRenderer` 时，如果您的项目对客户端和服务器端使用了不同的源代码集，则需要确保将该渲染器类放置于对应的源代码集中，例如客户端相关的类应放在 `src/client/` 目录下。 直接访问 `src/main/` 源代码集中与渲染相关的类并不安全，因为这些类可能已在服务器上加载。

首先，我们需要为 `CounterBlockEntity` 创建一个 `BlockEntityRenderState` 来保存将用于渲染的数据。 在这种情况下，我们需要 `clicks` 在渲染期间可用。

@[code transcludeWith=::render-state](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderState.java)

然后我们为 `CounterBlockEntity` 创建一个 `BlockEntityRenderer`。

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

新类有一个构造函数，以 `BlockEntityRendererProvider.Context` 作为参数。 `Context` 有几个非常有用的渲染辅助工具，比如 `ItemRenderer` 或 `TextRenderer`。
此外，通过包含这样一个构造函数，就可以将该构造函数用作 `BlockEntityRendererProvider` 函数式接口本身：

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/ExampleModBlockEntityRenderer.java)

我们将重写一些方法来设置渲染状态以及设置渲染逻辑的 `render` 方法。

`createRenderState` 可以用于初始化渲染状态。

@[code transclude={31-34}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

`extractRenderState` 可以用于使用实体数据更新渲染状态。

@[code transclude={36-42}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

你应该在 `ClientModInitializer` 类中注册你的方块实体渲染器。

`BlockEntityRenderers` 是一个注册表，它将每个具有自定义渲染代码的 `BlockEntityType` 映射到其各自的 `BlockEntityRenderer`。

## 在方块上绘制 {#drawing-on-blocks}

现在我们有了渲染器，就可以开始绘制了。 `render` 方法在每一帧都会被调用，这就是渲染魔法发生的地方。

### 四处移动 {#moving-around}

首先，我们需要偏移和旋转文本，使其位于方块的顶部。

::: info

As the name suggests, the `PoseStack` is a _stack_, meaning that you can push and pop transformations.
A good rule-of-thumb is to push a new one at the beginning of the `render` method and pop it at the end, so that the rendering of one block doesn't affect others.

More information about the `PoseStack` can be found in the [Basic Rendering Concepts article](../rendering/basic-concepts).

:::

To make the translations and rotations needed easier to understand, let's visualize them. In this picture, the green block is where the text would be drawn, by default in the furthest bottom-left point of the block:

![Default rendering position](/assets/develop/blocks/block_entity_renderer_1.png)

So first we need to move the text halfway across the block on the X and Z axes, and then move it up to the top of the block on the Y axis:

![Green block in the topmost center point](/assets/develop/blocks/block_entity_renderer_2.png)

This is done with a single `translate` call:

```java
matrices.translate(0.5, 1, 0.5);
```

That's the _translation_ done, _rotation_ and _scale_ remain.

By default, the text is drawn on the XY plane, so we need to rotate it 90 degrees around the X axis to make it face upwards on the XZ plane:

![Green block in the topmost center point, facing upwards](/assets/develop/blocks/block_entity_renderer_3.png)

The `PoseStack` does not have a `rotate` function, instead we need to use `multiply` and `Axis.XP`:

```java
matrices.multiply(Axis.XP.rotationDegrees(90));
```

Now the text is in the correct position, but it's too large. The `BlockEntityRenderer` maps the whole block to a `[-0.5, 0.5]` cube, while the `TextRenderer` uses Y coordinates of `[0, 9]`. As such, we need to scale it down by a factor of 18:

```java
matrices.scale(1/18f, 1/18f, 1/18f);
```

Now, the whole transformation looks like this:

@[code transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

### Drawing Text {#drawing-text}

As mentioned earlier, the `Context` passed into the constructor of our renderer has a `TextRenderer` that we can use to measure text (`width`), which is useful for centering.

To draw the text, we will be submitting the necessary data to the render queue. Since we're drawing some text, we can use the `submitText` method provided through the `OrderedRenderCommandQueue` instance passed into the `render` method.

@[code transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

The `submitText` method takes a lot of parameters, but the most important ones are:

- the `FormattedCharSequence` to draw;
- its `x` and `y` coordinates;
- the RGB `color` value;
- the `Matrix4f` describing how it should be transformed (to get one from a `PoseStack`, we can use `.last().pose()` to get the `Matrix4f` for the topmost entry).

And after all this work, here's the result:

![Counter Block with a number on top](/assets/develop/blocks/block_entity_renderer_4.png)
