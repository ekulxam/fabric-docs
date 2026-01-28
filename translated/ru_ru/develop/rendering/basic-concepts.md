---
title: Основные концепции рендеринга
description: Изучите основные концепции рендеринга с использованием движка рендеринга Minecraft.
authors:
  - "0x3C50"
  - IMB11
  - MildestToucan
---

<!---->

::: warning

Although Minecraft is built using OpenGL, as of version 1.17+ you cannot use legacy OpenGL methods to render your own things. Instead, you must use the new `BufferBuilder` system, which formats rendering data and uploads it to OpenGL to draw.

Подводя итог, вам придется использовать систему рендеринга Minecraft или создать свою собственную, использующую `GL.glDrawElements()`.

:::

:::warning ВАЖНОЕ ОБНОВЛЕНИЕ

Starting from 1.21.6, large changes are being implemented to the rendering pipeline, such as moving towards `RenderType`s and `RenderPipeline`s and more importantly, `RenderState`s, with the ultimate goal of being able to prepare the next frame while drawing the current frame. In the "preparation" phase, all game data used for rendering is extracted to `RenderState`s, so another thread can work on drawing that frame while the next frame is being extracted.

For example, in 1.21.8 GUI rendering adopted this model, and `GuiGraphics` methods simply add to the render state. Фактическая загрузка в `BufferBuilder` происходит в конце фазы подготовки, после того как все элементы добавлены в `RenderState`. См. `GuiRenderer#prepare`.

В этой статье рассматриваются основы рендеринга, и хотя она по-прежнему остается в определенной степени актуальной, в большинстве случаев для лучшей производительности и совместимости используются более высокие уровни абстракции. Для получения более подробной информации см. [Рендеринг в мире](./world).

:::

На этой странице будут рассмотрены основы рендеринга с использованием новой системы, а также ключевые термины и концепции.

Although much of rendering in Minecraft is abstracted through the various `GuiGraphics` methods, and you'll likely not need to touch anything mentioned here, it's still important to understand the basics of how rendering works.

## The `Tesselator` {#the-tesselator}

The `Tesselator` is the main class used to render things in Minecraft. Это singleton, то есть в игре существует только один его экземпляр. You can get the instance using `Tesselator.getInstance()`.

## `BufferBuilder` {#the-bufferbuilder}

`BufferBuilder` — это класс, используемый для форматирования и загрузки данных рендеринга в OpenGL. Он используется для создания буфера, который затем загружается в OpenGL для отрисовки.

The `Tesselator` is used to create a `BufferBuilder`, which is used to format and upload rendering data to OpenGL.

### Инициализация `BufferBuilder` {#initializing-the-bufferbuilder}

Прежде чем что-либо записать в `BufferBuilder`, его необходимо инициализировать. This is done using `Tesselator#begin(...)` method, which takes in a `VertexFormat` and a draw mode and returns a `BufferBuilder`.

#### Форматы вершин {#vertex-formats}

`VertexFormat` определяет элементы, которые мы включаем в наш буфер данных, и описывает, как эти элементы должны передаваться в OpenGL.

The following default `VertexFormat` elements are available at `DefaultVertexFormat`:

| Элемент                       | Формат                                                                                  |
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

#### Режимы рисования {#draw-modes}

Режим рисования определяет, как будут отображаться данные. The following draw modes are available at `VertexFormat.Mode`:

| Режим рисования    | Описание                                                                                                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `LINES`            | Каждый элемент состоит из 2 вершин и представлен в виде одной линии.                                                                                         |
| `LINE_STRIP`       | Для первого элемента требуется 2 вершины. Дополнительные элементы рисуются всего с одной новой вершиной, создавая непрерывную линию.         |
| `DEBUG_LINES`      | Similar to `Mode.LINES`, but the line is always exactly one pixel wide on the screen.                                                                        |
| `DEBUG_LINE_STRIP` | Same as `Mode.LINE_STRIP`, but lines are always one pixel wide.                                                                                              |
| `TRIANGLES`        | Каждый элемент состоит из 3 вершин, образующих треугольник.                                                                                                  |
| `TRIANGLE_STRIP`   | Начинается с 3 вершин первого треугольника. Каждая дополнительная вершина образует новый треугольник с двумя последними вершинами.           |
| `TRIANGLE_FAN`     | Начинается с 3 вершин первого треугольника. Каждая дополнительная вершина образует новый треугольник с первой вершиной и последней вершиной. |
| `QUADS`            | Каждый элемент состоит из 4 вершин, образующих четырехугольник.                                                                                              |

### Запись в `BufferBuilder` {#writing-to-the-bufferbuilder}

После инициализации `BufferBuilder` вы можете записывать в него данные.

`BufferBuilder` позволяет нам создавать наш буфер, вершина за вершиной. To add a vertex, we use the `buffer.addVertex(Matrix4f, float, float, float)` method. The `Matrix4f` parameter is the transformation matrix, which we'll discuss in more detail later. Три параметра с плавающей точкой представляют собой координаты (x, y, z) положения вершины.

Этот метод возвращает конструктор вершин, который мы можем использовать для указания дополнительной информации о вершине. При добавлении этой информации крайне важно соблюдать порядок, определенный в нашем `VertexFormat`. Если мы этого не сделаем, OpenGL может неправильно интерпретировать наши данные. После того как мы закончим построение вершины, просто продолжайте добавлять новые вершины и данные в буфер, пока не закончите.

Также стоит понять концепцию отсечения (culling). Отсечение — это процесс удаления граней трехмерной фигуры, которые не видны с точки зрения наблюдателя. Если вершины грани указаны в неправильном порядке, грань может отображаться неправильно из-за отсечения.

#### Что такое матрица трансформации? {#what-is-a-transformation-matrix}

Матрица преобразования — это матрица размером 4x4, которая используется для преобразования вектора. In Minecraft, the transformation matrix is just transforming the coordinates we give into the `addVertex` call. Преобразования позволяют масштабировать нашу модель, перемещать и вращать ее.

Иногда ее называют матрицей позиций или матрицей моделей.

It's usually obtained via the `Matrix3x2fStack` class, which can be obtained via the `GuiGraphics` object by calling the `GuiGraphics#pose()` method.

#### Рендеринг полосы треугольников {#rendering-a-triangle-strip}

Проще объяснить, как писать в `BufferBuilder`, на практическом примере. Let's say we want to render something using the `VertexFormat.Mode.TRIANGLE_STRIP` draw mode and the `POSITION_COLOR` vertex format.

Мы собираемся нарисовать вершины в следующих точках HUD (по порядку):

```text:no-line-numbers
(20, 20)
(5, 40)
(35, 40)
(20, 60)
```

Это должно дать нам прекрасный ромб — поскольку мы используем режим рисования `TRIANGLE_STRIP`, render выполнит следующие шаги:

![Четыре шага, демонстрирующие размещение вершин на экране для формирования двух треугольников](/assets/develop/rendering/concepts-practical-example-draw-process.png)

Поскольку в этом примере мы рисуем на HUD, мы будем использовать `HudElementRegistry`:

:::warning ВАЖНОЕ ОБНОВЛЕНИЕ

Starting from 1.21.8, the matrix stack passed for HUD rendering has been changed from `PoseStack` to `Matrix3x2fStack`. Most methods are slightly different and no longer take a `z` parameter, but the concepts are the same.

Additionally, the code below does not fully match the explanation above: you do not need to manually write to the `BufferBuilder`, because `GuiGraphics` methods automatically write to the HUD's `BufferBuilder` during preparation.

Для получения более подробной информации прочтите важное обновление выше.

:::

**Element registration:**

@[code lang=java transcludeWith=:::registration](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

**Implementation of `hudLayer()`:**

@[code lang=java transcludeWith=:::hudLayer](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

В результате на HUD отображается следующее:

![Конечный результат](/assets/develop/rendering/concepts-practical-example-final-result.png)

::: tip

Попробуйте поиграться с цветами и положением вершин, чтобы посмотреть, что получится! Вы также можете попробовать использовать различные режимы рисования и форматы вершин.

:::

## The `PoseStack` {#the-posestack}

::: warning

This section's code and the text are discussing different things!

The code showcases `Matrix3x2fStack`, which is used for HUD rendering since 1.21.8, while the text describes `PoseStack`, which has slightly different methods.

Для получения более подробной информации прочтите важное обновление выше.

:::

Узнав, как писать в `BufferBuilder`, вы, возможно, зададитесь вопросом, как преобразовать свою модель или даже анимировать ее. This is where the `PoseStack` class comes in.

The `PoseStack` class has the following methods:

- `pushPose()` - Pushes a new matrix onto the stack.
- `popPose()` - Pops the top matrix off the stack.
- `last()` - Returns the top matrix on the stack.
- `translate(x, y, z)` — переводит верхнюю матрицу в стеке.
- `translate(vec3)`
- `scale(x, y, z)` — масштабирует верхнюю матрицу в стеке.

Вы также можете умножить верхнюю матрицу в стеке, используя кватернионы, о чем мы поговорим в следующем разделе.

Taking from our example above, we can make our diamond scale up and down by using the `PoseStack` and the `tickDelta` - which is the "progress" between the last game tick and the next game tick. Мы разъясним это позже на странице [Рендеринг в Hud](./hud#tick-delta).

::: warning

You must first push the matrix stack and then pop it after you're done with it. If you don't, you'll end up with a broken matrix stack, which will cause rendering issues.

Обязательно выдвиньте стек матриц, прежде чем получить матрицу преобразования!

:::

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

![Видео, демонстрирующее увеличение и уменьшение размера алмаза](/assets/develop/rendering/concepts-matrix-stack.webp)

## Кватернионы (вращающиеся вещи) {#quaternions-rotating-things}

::: warning

This section's code and the text are discussing different things!

Код демонстрирует рендеринг на HUD, тогда как текст описывает рендеринг трехмерного пространства мира.

Для получения более подробной информации прочтите важное обновление выше.

:::

Кватернионы — это способ представления вращений в трехмерном пространстве. They are used to rotate the top matrix on the `PoseStack` via the `rotateAround(quaternionfc, x, y, z)` method.

It's highly unlikely you'll need to ever use a Quaternion class directly, since Minecraft provides various pre-built Quaternion instances in its `Axis` utility class.

Допустим, мы хотим повернуть наш квадрат вокруг оси z. We can do this by using the `PoseStack` and the `rotateAround(quaternionfc, x, y, z)` method.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

Результатом этого является следующее:

![Видео, демонстрирующее вращение алмаза вокруг оси z](/assets/develop/rendering/concepts-quaternions.webp)
