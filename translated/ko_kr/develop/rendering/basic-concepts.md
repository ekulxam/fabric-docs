---
title: 기본 렌더링 개념
description: Minecraft의 렌더링 엔진에 사용되는 기본적인 렌더링의 개념을 알아보세요.
authors:
  - "0x3C50"
  - IMB11
  - MildestToucan
---

<!---->

::: warning

Although Minecraft is built using OpenGL, as of version 1.17+ you cannot use legacy OpenGL methods to render your own things. Instead, you must use the new `BufferBuilder` system, which formats rendering data and uploads it to OpenGL to draw.

간단히 요약하자면, Minecraft의 렌더링 시스템을 사용하거나, `GL.glDrawElements()`를 활용하는 자체 렌더링 시스템을 구축하여야 합니다.

:::

:::warning IMPORTANT UPDATE

Starting from 1.21.6, large changes are being implemented to the rendering pipeline, such as moving towards `RenderType`s and `RenderPipeline`s and more importantly, `RenderState`s, with the ultimate goal of being able to prepare the next frame while drawing the current frame. In the "preparation" phase, all game data used for rendering is extracted to `RenderState`s, so another thread can work on drawing that frame while the next frame is being extracted.

For example, in 1.21.8 GUI rendering adopted this model, and `GuiGraphics` methods simply add to the render state. The actual uploading to the `BufferBuilder` happens at the end of the preparation phase, after all elements have been added to the `RenderState`. See `GuiRenderer#prepare`.

This article covers the basics of rendering and, while still somewhat relevant, most times there are higher levels of abstractions for better performance and compatibility. For more information, see [Rendering in the World](./world).

:::

이 튜토리얼에서는 새로운 렌더링 시스템을 만들며 렌더링의 기본 사항과 주요 용어, 개념에 대해 알아볼 것입니다.

Although much of rendering in Minecraft is abstracted through the various `GuiGraphics` methods, and you'll likely not need to touch anything mentioned here, it's still important to understand the basics of how rendering works.

## The `Tesselator` {#the-tesselator}

The `Tesselator` is the main class used to render things in Minecraft. 이는 싱글톤이므로, 게임에 오직 하나의 인스턴스만 존재할 수 있습니다. You can get the instance using `Tesselator.getInstance()`.

## `BufferBuilder`

`BufferBuilder`는 OpenGL에 렌더링을 포맷하고 업로드하는 클래스입니다. 이는 화면에 게임을 그리기 위해 OpenGL에 업로드되는 Buffer를 생성합니다.

The `Tesselator` is used to create a `BufferBuilder`, which is used to format and upload rendering data to OpenGL.

### `BufferBuilder` 초기화 하기

`BufferBuilder`에 무엇이든 쓰기 전에, 먼저 초기화해야 합니다. This is done using `Tesselator#begin(...)` method, which takes in a `VertexFormat` and a draw mode and returns a `BufferBuilder`.

#### 꼭짓점 포맷 {#vertex-formats}

`VertexFormat`은 데이터 버퍼에 포함할 요소를 정의하고 어떻게 물체가 OpenGL에서 처리되어야 하는지에 대한 개요를 만듭니다.

The following default `VertexFormat` elements are available at `DefaultVertexFormat`:

| 요소                            | 포맷                                                                                      |
| ----------------------------- | --------------------------------------------------------------------------------------- |
| `EMPTY`                       | `{ }`                                                                                   |
| `BLOCK`                       | `{ position, color, texture uv, texture light (2 shorts), texture normal (3 sbytes) }`  |
| `NEW_ENTITY`                  | `{ position, color, texture uv, overlay (2 shorts), texture light, normal (3 sbytes) }` |
| `PARTICLE`                    | `{ position, texture uv, color, texture light }`                                        |
| `POSITION`                    | `{ position }`                                                                          |
| `POSITION_COLOR`              | `{ position, color }`                                                                   |
| `POSITION_COLOR_NORMAL`       | `{ position, color, normal }`                                                           |
| `POSITION_COLOR_LIGHTMAP`     | `{ position, color, light }`                                                            |
| `POSITION_TEX`                | `{ position, uv }`                                                                      |
| `POSITION_TEX_COLOR`          | `{ position, uv, color }`                                                               |
| `POSITION_COLOR_TEX_LIGHTMAP` | `{ position, color, uv, light }`                                                        |
| `POSITION_TEX_LIGHTMAP_COLOR` | `{ position, uv, light, color }`                                                        |
| `POSITION_TEX_COLOR_NORMAL`   | `{ position, uv, color, normal }`                                                       |

#### 그리기 모드

그리기 모드는 데이터가 그려지는 방법을 결정합니다. The following draw modes are available at `VertexFormat.Mode`:

| 그리기 모드             | 설명                                                                                                    |
| ------------------ | ----------------------------------------------------------------------------------------------------- |
| `LINES`            | 각 요소는 2개의 꼭짓점으로 구성되며 단일 선으로 표시됩니다.                                                    |
| `LINE_STRIP`       | 첫 번째 선만 2개의 꼭짓점을 가집니다. 이후 꼭짓점은 기존에 있던 꼭짓점과 연결되어, 연속적인 줄을 만듭니다.        |
| `DEBUG_LINES`      | Similar to `Mode.LINES`, but the line is always exactly one pixel wide on the screen. |
| `DEBUG_LINE_STRIP` | Same as `Mode.LINE_STRIP`, but lines are always one pixel wide.                       |
| `TRIANGLES`        | 각 요소가 3개의 꼭짓점으로 만들어져, 삼각형이 구성합니다.                                                     |
| `TRIANGLE_STRIP`   | 첫 삼각형만 세 개의 꼭짓점을 가집니다. 이후 추가된 꼭짓점은 기존에 있던 두 꼭짓점으로 새 삼각형을 구성하게 됩니다.    |
| `TRIANGLE_FAN`     | 첫 삼각형만 세 개의 꼭짓점을 가집니다. 첫 삼각형만 세 개의 꼭짓점을 가집니다.                         |
| `QUADS`            | 각 요소가 4개의 꼭짓점으로 만들어져, 사각형을 구성합니다.                                                     |

### `BufferBuilder` 쓰기

`BufferBuilder`가 초기화 되면, 이제 데이터를 쓸 수 있습니다.

`BufferBuilder`로 버퍼를 만들거나, 꼭짓점끼리 서로 연결할 수 있습니다. To add a vertex, we use the `buffer.addVertex(Matrix4f, float, float, float)` method. The `Matrix4f` parameter is the transformation matrix, which we'll discuss in more detail later. 세 float 매개변수는 꼭짓점의 (x, y, z) 좌표를 의미합니다.

사용하면 꼭짓점에 추가적인 정보를 추가할 수 있게 꼭짓점 빌더가 반환됩니다. 정보를 추가할 때 추가한 `VertexFormat` 순서를 따르는 것이 중요합니다. 그렇지 않으면 OpenGL이 데이터를 제대로 해석하지 못할 수 있습니다. 꼭짓점 빌딩을 끝마친 후, 끝나기 전까지 버퍼에 데이터를 추가하거나 계속해서 꼭짓점을 더 추가하세요.

컬링의 개념을 이해하는 것도 가치 있는 일입니다. 컬링은 플레이어의 시야에서 보이지 않는 3차원 면을 제거하는 기법입니다. 만약 면의 꼭지점이 잘못된 순서로 배열되어 있으면, 컬링으로 인해 면이 정상적으로 렌더링되지 않을 수 있습니다.

#### 변환 행렬이 무엇인가요? {#what-is-a-transformation-matrix}

변환 행렬은 벡터를 변환하기 위하여 사용되는 4x4 행렬입니다. In Minecraft, the transformation matrix is just transforming the coordinates we give into the `addVertex` call. 변환을 통해 꼭지점의 크기를 키우거나, 움직이거나, 회전할 수 있습니다.

때로는 위치 행렬 또는 모델 행렬이라고도 합니다.

It's usually obtained via the `Matrix3x2fStack` class, which can be obtained via the `GuiGraphics` object by calling the `GuiGraphics#pose()` method.

#### 실전 예시: 삼각형 스트립 렌더링하기

현실적인 예시로 `BufferBuilder`를 쓰는 방법을 설명하는 것이 더 쉽습니다. Let's say we want to render something using the `VertexFormat.Mode.TRIANGLE_STRIP` draw mode and the `POSITION_COLOR` vertex format.

순서대로 HUD에 꼭짓점을 그려봅시다.

```text:no-line-numbers
(20, 20)
(5, 40)
(35, 40)
(20, 60)
```

`TRIANGLE_STRIP` 그리기 모드를 사용했으므로, 꼭짓점을 그리면 다음과 같은 과정을 거쳐 사랑스러운 다이아몬드가 렌더링될 것입니다.

![두 개의 삼각형을 형성하기 위하여 화면에 꼭짓점의 위치를 표시하는 네 단계](/assets/develop/rendering/concepts-practical-example-draw-process.png)

Since we're drawing on the HUD in this example, we'll use the `HudElementRegistry`:

:::warning IMPORTANT UPDATE

Starting from 1.21.8, the matrix stack passed for HUD rendering has been changed from `PoseStack` to `Matrix3x2fStack`. Most methods are slightly different and no longer take a `z` parameter, but the concepts are the same.

Additionally, the code below does not fully match the explanation above: you do not need to manually write to the `BufferBuilder`, because `GuiGraphics` methods automatically write to the HUD's `BufferBuilder` during preparation.

Read the important update above for more information.

:::

**Element registration:**

@[code lang=java transcludeWith=:::registration](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

**Implementation of `hudLayer()`:**

@[code lang=java transcludeWith=:::hudLayer](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

이렇게 하면 HUD에 두 삼각형이 그려지게 됩니다.

![최종 결과](/assets/develop/rendering/concepts-practical-example-final-result.png)

::: tip

색을 지정하거나 꼭짓점의 위치를 옮겨 어떤 변화가 일어나는지 알아보세요! 다른 그리기 모드와 꼭짓점 포맷을 사용해 볼 수도 있습니다.

:::

## The `PoseStack` {#the-posestack}

::: warning

This section's code and the text are discussing different things!

The code showcases `Matrix3x2fStack`, which is used for HUD rendering since 1.21.8, while the text describes `PoseStack`, which has slightly different methods.

Read the important update above for more information.

:::

어떻게 `BufferBuilder`를 쓰는지 알았다면, 이제 어떻게 모델을 움직이고, 좀 더 멋진 분들은 어떻게 애니메이션을 적용할지 궁금할 수도 있습니다. This is where the `PoseStack` class comes in.

The `PoseStack` class has the following methods:

- `pushPose()` - Pushes a new matrix onto the stack.
- `popPose()` - Pops the top matrix off the stack.
- `last()` - Returns the top matrix on the stack.
- `translate(x, y, z)` - 최상단 스택을 이동합니다.
- `translate(vec3)`
- `scale(x, y, z)` - 최상단 스택의 크기를 조절합니다.

다음 섹션에서 알아볼 쿼터니언을 통해 최상단 행렬을 곱할 수도 있습니다.

Taking from our example above, we can make our diamond scale up and down by using the `PoseStack` and the `tickDelta` - which is the "progress" between the last game tick and the next game tick. 추후에 [HUD에서 렌더링](./hud#render-tick-counter) 페이지에서 자세히 다룰 예정입니다.

::: warning

You must first push the matrix stack and then pop it after you're done with it. If you don't, you'll end up with a broken matrix stack, which will cause rendering issues.

행렬을 변환하기 전에 행렬 스택을 넣었는지 확인하세요!

:::

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

![다이아몬드의 크기가 변화하는 모습을 보여주는 영상](/assets/develop/rendering/concepts-matrix-stack.webp)

## 쿼터니언 (움직이는 것) {#quaternions-rotating-things}

::: warning

This section's code and the text are discussing different things!

The code showcases rendering on the HUD, while the text describes rendering the 3D world space.

Read the important update above for more information.

:::

쿼터니언은 3차원에서 회전을 표현하는 방법입니다. They are used to rotate the top matrix on the `PoseStack` via the `rotateAround(quaternionfc, x, y, z)` method.

It's highly unlikely you'll need to ever use a Quaternion class directly, since Minecraft provides various pre-built Quaternion instances in its `Axis` utility class.

Let's say we want to rotate our square around the z-axis. We can do this by using the `PoseStack` and the `rotateAround(quaternionfc, x, y, z)` method.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

결과는 다음과 같습니다.

![다이아몬드의 Z축이 변화하는 모습을 보여주는 영상](/assets/develop/rendering/concepts-quaternions.webp)
