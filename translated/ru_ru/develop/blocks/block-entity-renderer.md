---
title: Рендеры сущностей блока
description: Узнайте, как разнообразить рендеринг с помощью блочных рендереров сущностей.
authors:
  - natri0
---

Иногда использования формата моделей Minecraft недостаточно. Если вам нужно добавить динамический рендеринг к визуальным эффектам блока, вам нужно использовать `BlockEntityRenderer`.

Например, давайте сделаем так, чтобы блок Counter из статьи [Block Entities](../blocks/block-entities) показывал кол-во кликов на своей верхней стороне.

## Создание BlockEntityRenderer {#creating-a-blockentityrenderer}

Рендеринг блоков, сущностей использует систему submit/render, в которой вы сначала отправляете данные, необходимые для рендеринга объекта на экран, а затем игра рендерит объект, используя его отправленное состояние.

При создании `BlockEntityRenderer` для `CounterBlockEntity` важно поместить класс в соответствующий набор исходных текстов, например `src/client/`, если ваш проект использует раздельные наборы исходных текстов для клиента и сервера. Обращение к классам, связанным с рендерингом, непосредственно в наборе исходных текстов `src/main/` небезопасно, поскольку эти классы могут быть загружены на сервере.

Во-первых, нам нужно создать `BlockEntityRenderState` для нашего `CounterBlockEntity`, чтобы хранить данные, которые будут использоваться для рендеринга. В этом случае нам нужно, чтобы `клики` были доступны во время рендеринга.

@[code transcludeWith=::render-state](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderState.java)

Затем мы создадим `BlockEntityRenderer` для нашего `CounterBlockEntity`.

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

Новый класс имеет конструктор с `BlockEntityRendererProvider.Context` в качестве параметра. В `Context` есть несколько полезных утилит рендеринга, например, `ItemRenderer` или `TextRenderer`.
Также, включение такого конструктора позволяет использовать конструктор в качестве функционального интерфейса `BlockEntityRendererProvider`:

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/ExampleModBlockEntityRenderer.java)

Мы переопределим несколько методов для настройки состояния рендеринга, а также метод `render`, в котором будет настроена логика рендеринга.

`createRenderState` можно использовать для инициализации состояния рендеринга.

@[code transclude={31-34}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

`extractRenderState` может быть использован для обновления состояния рендеринга с помощью данных о сущностях.

@[code transclude={36-42}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

Вы должны зарегистрировать рендеринг блочных сущностей в классе `ClientModInitializer`.

`BlockEntityRenderers` - это реестр, который сопоставляет каждый `BlockEntityType` с пользовательским кодом рендеринга с соответствующим `BlockEntityRenderer`.

## Рисование на блоках {#drawing-on-blocks}

Теперь, когда у нас есть рендерер, мы можем рисовать. Метод `render` вызывается каждый кадр, и именно в нем происходит магия рендеринга.

### Передвижение {#moving-around}

Сначала нам нужно сместить и повернуть текст так, чтобы он находился на верхней стороне блока.

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
