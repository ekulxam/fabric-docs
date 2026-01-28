---
title: Conceptos Básicos de Renderización
description: Aprende sobre los conceptos básicos de renderización usando el motor de renderización de Minecraft.
authors:
  - "0x3C50"
  - IMB11
  - MildestToucan
---

<!---->

::: warning

Although Minecraft is built using OpenGL, as of version 1.17+ you cannot use legacy OpenGL methods to render your own things. Instead, you must use the new `BufferBuilder` system, which formats rendering data and uploads it to OpenGL to draw.

En resumen, tienes que usar el motor de renderizado de Minecraft, o hacer tu propio que utilize `GL.glDrawElements()` (dibujarElementos).

:::

:::warning IMPORTANT UPDATE

Starting from 1.21.6, large changes are being implemented to the rendering pipeline, such as moving towards `RenderType`s and `RenderPipeline`s and more importantly, `RenderState`s, with the ultimate goal of being able to prepare the next frame while drawing the current frame. In the "preparation" phase, all game data used for rendering is extracted to `RenderState`s, so another thread can work on drawing that frame while the next frame is being extracted.

For example, in 1.21.8 GUI rendering adopted this model, and `GuiGraphics` methods simply add to the render state. The actual uploading to the `BufferBuilder` happens at the end of the preparation phase, after all elements have been added to the `RenderState`. See `GuiRenderer#prepare`.

This article covers the basics of rendering and, while still somewhat relevant, most times there are higher levels of abstractions for better performance and compatibility. For more information, see [Rendering in the World](./world).

:::

Esta página tratará los conceptos básicos de renderizado usando el nuevo sistema, cubriendo la terminología y conceptos importantes.

Although much of rendering in Minecraft is abstracted through the various `GuiGraphics` methods, and you'll likely not need to touch anything mentioned here, it's still important to understand the basics of how rendering works.

## The `Tesselator` {#the-tesselator}

The `Tesselator` is the main class used to render things in Minecraft. Es una clase de tipo única, lo que quiere decir que solo hay una instancia de él en el juego. You can get the instance using `Tesselator.getInstance()`.

## El `BufferBuilder` (Constructor de Buffer)

El `BufferBuilder` es la clase usada para formatear y subir información o datos de renderización a OpenGL. Es usado para crear un buffer, el cual es subido a OpenGL para que lo dibuje.

The `Tesselator` is used to create a `BufferBuilder`, which is used to format and upload rendering data to OpenGL.

### Inicializando el `BufferBuilder`

Antes de que puedas enviar datos al `BufferBuilder`, debes inicializarlo. This is done using `Tesselator#begin(...)` method, which takes in a `VertexFormat` and a draw mode and returns a `BufferBuilder`.

#### Formatos de Vertice

El `VertexFormat` define los elementos que incluimos en nuestro buffer de datos y determina como estos elementos deberían ser transmitidos a OpenGL.

The following default `VertexFormat` elements are available at `DefaultVertexFormat`:

| Elemento                      | Formato                                                                                    |
| ----------------------------- | ------------------------------------------------------------------------------------------ |
| `EMPTY`                       | `{ }`                                                                                      |
| `BLOCK`                       | `{ posición, color, textura uv, luz de textura (2 shorts), normal de textura (3 sbytes) }` |
| `NEW_ENTITY`                  | `{ posición, color, textura uv, cubierta (2 shorts), luz de textura, normal (3 sbytes) }`  |
| `PARTICLE`                    | `{ posición, textura uv, color, luz de textura }`                                          |
| `POSITION`                    | `{ posición }`                                                                             |
| `POSITION_COLOR`              | `{ posición, color }`                                                                      |
| `POSITION_COLOR_NORMAL`       | `{ posición, color, normal }`                                                              |
| `POSITION_COLOR_LIGHTMAP`     | `{ posición, color, luz }`                                                                 |
| `POSITION_TEX`                | `{ posición, uv }`                                                                         |
| `POSITION_TEX_COLOR`          | `{ posición, uv, color }`                                                                  |
| `POSITION_COLOR_TEX_LIGHTMAP` | `{ posición, color, uv, luz }`                                                             |
| `POSITION_TEX_LIGHTMAP_COLOR` | `{ posición, uv, luz, color }`                                                             |
| `POSITION_TEX_COLOR_NORMAL`   | `{ posición, uv, color, normal }`                                                          |

#### Modos de Dibujado

El modo de dibujado define como son dibujados los datos. The following draw modes are available at `VertexFormat.Mode`:

| Modo de Dibujado   | Descripción                                                                                                                                                                |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `LINES`            | Cada elemento está compuesto de 2 vértices y es representado como una sola línea.                                                                          |
| `LINE_STRIP`       | El primer elemento requiere 2 vértices. Elementos adicionales son dibujados con un solo nuevo vértice, creando una línea continua.         |
| `DEBUG_LINES`      | Similar to `Mode.LINES`, but the line is always exactly one pixel wide on the screen.                                                                      |
| `DEBUG_LINE_STRIP` | Same as `Mode.LINE_STRIP`, but lines are always one pixel wide.                                                                                            |
| `TRIANGLES`        | Cada elemento está compuesto de 3 vértices, formando un triángulo.                                                                                         |
| `TRIANGLE_STRIP`   | Empieza con 3 vértices para formar el primer triángulo. Cada vértice adicional produce un nuevo triángulo usando los dos últimos vértices. |
| `TRIANGLE_FAN`     | Empieza con 3 vértices para formar el primer triángulo. Cada vértice adicional produce un nuevo triángulo con el primer y último vértice.  |
| `QUADS`            | Cada elemento está formado de 4 vértices, formando un cuadrilátero.                                                                                        |

### Escribir datos para el `BufferBuilder`

Una vez inicializado el `BufferBuilder`, puedes escribir datos en él.

El `BufferBuilder` nos permite construir nuestro buffer, vértice por vértice. To add a vertex, we use the `buffer.addVertex(Matrix4f, float, float, float)` method. The `Matrix4f` parameter is the transformation matrix, which we'll discuss in more detail later. Los 3 parámetros flotantes representan las coordenadas (x, y, z) de la posición del vértice.

Este método retorna un constructor de vértice, el cual podemos usar para especificar información adicional para el vértice. Es crucial seguir el orden de nuestro `VertexFormat` definido a la hora de agregar esta información. De lo contrario, `OpenGL` puede que no interprete nuestros datos correctamente. After we've finished building a vertex, just continue adding more vertices and data to the buffer until you're done.

También es importante entender el concepto de _culling_ (eliminación). _Culling_ es el proceso de remover caras de un objeto 3D que no son visibles desde la perspectiva del observador. Si los vértices de una cara son especificadas en el orden incorrecto, las caras pueden no mostrarse correctamente debido al _culling_.

#### ¿Que es una Matriz de Transformación? {#what-is-a-transformation-matrix}

Una matriz de transformación es una matriz 4x4 que es usada para transformar un vector. In Minecraft, the transformation matrix is just transforming the coordinates we give into the `addVertex` call. La transformación puede modificar el tamaño de nuestro modelo, moverlo o rotarlo.

También se conoce como la matriz de posición, o la matriz de modelo.

It's usually obtained via the `Matrix3x2fStack` class, which can be obtained via the `GuiGraphics` object by calling the `GuiGraphics#pose()` method.

#### Un Ejemplo Práctico: Renderizar una Banda de Triángulos

Es más facil explicar como escribir al `BufferBuilder` usando un ejemplo práctico. Let's say we want to render something using the `VertexFormat.Mode.TRIANGLE_STRIP` draw mode and the `POSITION_COLOR` vertex format.

Vamos a dibujar vértices usando los siguientes puntos en el HUD (en orden):

```text:no-line-numbers
(20, 20)
(5, 40)
(35, 40)
(20, 60)
```

Esto nos debería dar un hermoso diamante - ya que estamos usando el modo de dibujado de `TRIANGLE_STRIP`, el renderizado realizará los siguientes pasos:

![Cuatro pasos que muestran el posicionamiento de los vértices en la pantalla para formar dos triángulos](/assets/develop/rendering/concepts-practical-example-draw-process.png)

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

Esto resulta en lo siguiente siendo dibujado en el HUD:

![Resultado Final](/assets/develop/rendering/concepts-practical-example-final-result.png)

::: tip

¡Intenta jugar con los colores y las posiciones del vértice para ver que pasa! También puedes intentar usar diferentes modos de dibujado y formatos de vértice.

:::

## The `PoseStack` {#the-posestack}

::: warning

This section's code and the text are discussing different things!

The code showcases `Matrix3x2fStack`, which is used for HUD rendering since 1.21.8, while the text describes `PoseStack`, which has slightly different methods.

Read the important update above for more information.

:::

Después de aprender a escribir al `BufferBuilder`, puede que te preguntes como transformar el modelo - o incluso animarlo. This is where the `PoseStack` class comes in.

The `PoseStack` class has the following methods:

- `pushPose()` - Pushes a new matrix onto the stack.
- `popPose()` - Pops the top matrix off the stack.
- `last()` - Returns the top matrix on the stack.
- `translate(x, y, z)` - Traslada la matriz en el tope de la pila.
- `translate(vec3)`
- `scale(x, y, z)` - Cambia el tamaño de la matriz en el tope de la pila.

También puedes multiplicar la matriz en el tope de la pila usando cuaterniones, los cuales cubriremos en la siguiente sección.

Taking from our example above, we can make our diamond scale up and down by using the `PoseStack` and the `tickDelta` - which is the "progress" between the last game tick and the next game tick. We'll clarify this later in the [Rendering in the HUD](./hud#render-tick-counter) page.

::: warning

You must first push the matrix stack and then pop it after you're done with it. If you don't, you'll end up with a broken matrix stack, which will cause rendering issues.

¡Asegúrate de empujar la pila de matrices antes de tener una matriz de transformación!

:::

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

![Un video mostrando el diamante aumentando y disminuyendo su tamaño](/assets/develop/rendering/concepts-matrix-stack.webp)

## Cuaterniones (Para Rotar Cosas)

::: warning

This section's code and the text are discussing different things!

The code showcases rendering on the HUD, while the text describes rendering the 3D world space.

Read the important update above for more information.

:::

Los cuaterniones son una manera de representar rotaciones en un espacio tridimensional. They are used to rotate the top matrix on the `PoseStack` via the `rotateAround(quaternionfc, x, y, z)` method.

It's highly unlikely you'll need to ever use a Quaternion class directly, since Minecraft provides various pre-built Quaternion instances in its `Axis` utility class.

Let's say we want to rotate our square around the z-axis. We can do this by using the `PoseStack` and the `rotateAround(quaternionfc, x, y, z)` method.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

El resultado será el siguiente:

![Un video que muestra el diamante rotando alrededor del eje z](/assets/develop/rendering/concepts-quaternions.webp)
