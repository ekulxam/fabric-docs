---
title: 블록 엔티티 렌더러
description: 블록 엔티티 렌더러를 사용해서 렌더링을 맵시있게 하는 방법을 알아보세요.
authors:
  - natri0
---

이따금씩 Minecraft의 기본 모델 포맷만으로는 충분하지 않을 때가 있습니다. 블록의 외형에 동적인 렌더링(움직이거나 변하는 모습)을 추가하고 싶다면,
`BlockEntityRenderer`를 사용해야 합니다.

예를 들면, 우리가 [Block Entities](../blocks/block-entities)파트에서 만들었던 Counter Block 위에 클릭한 횟수를 표시해봅시다.

## BlockEntityRenderer 만들기 {#creating-a-blockentityrenderer}

Block entity rendering uses a submit/render system where you first submit the data required to render an object to the screen, the game then renders the object using it's submitted state.

이 렌더러 클래스를 만들 때 `src/client/` 폴더에 놓는 게 중요합니다.
프로젝트가 클라이언트와 서버 코드를 분리하고 있다면 꼭 그래야 합니다. 만약 렌더링 관련 클래스를 실수로 `src/main/`에 만들면, 서버가 실행될 때 렌더링 클래스를 로드하려고 해서 크래시(충돌) 가 날 수 있습니다.

First, we need to create a `BlockEntityRenderState` for our `CounterBlockEntity` to hold the data that will be used for rendering. In this case, we will need the `clicks` to be available during rendering.

@[code transcludeWith=::render-state](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderState.java)

Then we create a `BlockEntityRenderer` for our `CounterBlockEntity`.

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

The new class has a constructor with `BlockEntityRendererProvider.Context` as a parameter. 이 `Context` 안에는 다양한 렌더링 도구들(예: `ItemRenderer`, `TextRenderer` 등)이 들어있습니다.
Also, by including a constructor like this, it becomes possible to use the constructor as the `BlockEntityRendererProvider` functional interface itself:

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/ExampleModBlockEntityRenderer.java)

We will override a few methods to set up the render state along with the `render` method where the rendering logic will be set up.

`createRenderState` can be used to initialize the render state.

@[code transclude={31-34}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

`extractRenderState` can be used to update the render state with entity data.

@[code transclude={36-42}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

우리가 만든 `BlockEntityRenderer`는 `ClientModInitializer` 클래스 안에서 등록해야 합니다.

`BlockEntityRenderers` is a registry that maps each `BlockEntityType` with custom rendering code to its respective `BlockEntityRenderer`.

## 블록 위에 그리기 {#drawing-on-blocks}

이제 렌더러를 만들었으니까, 진짜로 그림을 그릴 수 있습니다. `render` 메서드는 매 프레임마다 호출되고, 이 메서드 안에서 이를테면 '렌더링 마법'이 일어납니다.

### 위치 이동하기 {#moving-around}

먼저, 텍스트를 블록의 위쪽에 오도록 위치를 이동(translate)시키고, 회전(rotation)도 시켜야 합니다.

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
