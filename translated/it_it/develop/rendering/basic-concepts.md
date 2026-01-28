---
title: Concetti Base del Rendering
description: Impara i concetti base del rendering usando il motore grafico di Minecraft.
authors:
  - "0x3C50"
  - IMB11
  - MildestToucan
---

<!---->

::: warning

Although Minecraft is built using OpenGL, as of version 1.17+ you cannot use legacy OpenGL methods to render your own things. Instead, you must use the new `BufferBuilder` system, which formats rendering data and uploads it to OpenGL to draw.

Per riassumere, devi usare il sistema di rendering di Minecraft, o crearne uno tuo che usa `GL.glDrawElements()`.

:::

:::warning IMPORTANT UPDATE

Starting from 1.21.6, large changes are being implemented to the rendering pipeline, such as moving towards `RenderType`s and `RenderPipeline`s and more importantly, `RenderState`s, with the ultimate goal of being able to prepare the next frame while drawing the current frame. In the "preparation" phase, all game data used for rendering is extracted to `RenderState`s, so another thread can work on drawing that frame while the next frame is being extracted.

For example, in 1.21.8 GUI rendering adopted this model, and `GuiGraphics` methods simply add to the render state. The actual uploading to the `BufferBuilder` happens at the end of the preparation phase, after all elements have been added to the `RenderState`. See `GuiRenderer#prepare`.

This article covers the basics of rendering and, while still somewhat relevant, most times there are higher levels of abstractions for better performance and compatibility. For more information, see [Rendering in the World](./world).

:::

Questa pagina tratterà le basi del rendering usando il nuovo sistema, presentando terminologia e concetti chiave.

Although much of rendering in Minecraft is abstracted through the various `GuiGraphics` methods, and you'll likely not need to touch anything mentioned here, it's still important to understand the basics of how rendering works.

## The `Tesselator` {#the-tesselator}

The `Tesselator` is the main class used to render things in Minecraft. È un singleton, cioè solo un'istanza è presente in gioco. You can get the instance using `Tesselator.getInstance()`.

## Il `BufferBuilder` {#the-bufferbuilder}

Il `BufferBuilder` è la classe usata per formattare e caricare i dati di rendering su OpenGL. Viene usata per creare un buffer, che viene caricato su OpenGL per essere disegnato.

The `Tesselator` is used to create a `BufferBuilder`, which is used to format and upload rendering data to OpenGL.

### Inizializzare il `BufferBuilder` {#initializing-the-bufferbuilder}

Prima di poter scrivere al `BufferBuilder`, devi inizializzarlo. This is done using `Tesselator#begin(...)` method, which takes in a `VertexFormat` and a draw mode and returns a `BufferBuilder`.

#### Formati dei Vertici {#vertex-formats}

Il `VertexFormat` definisce gli elementi che includiamo nel nostro buffer di dati e precisa come questi elementi debbano essere trasmessi a OpenGL.

The following default `VertexFormat` elements are available at `DefaultVertexFormat`:

| Elemento                      | Formato                                                                                 |
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

#### Modalità di Disegno {#draw-modes}

La modalità di disegno definisce come sono disegnati i dati. The following draw modes are available at `VertexFormat.Mode`:

| Modalità di Disegno | Descrizione                                                                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `LINES`             | Ogni elemento è fatto da 2 vertici ed è rappresentato come una linea singola.                                                                       |
| `LINE_STRIP`        | Il primo elemento richiede 2 vertici. Elementi addizionali vengono disegnati con un solo nuovo vertice, creando una linea continua. |
| `DEBUG_LINES`       | Similar to `Mode.LINES`, but the line is always exactly one pixel wide on the screen.                                                               |
| `DEBUG_LINE_STRIP`  | Same as `Mode.LINE_STRIP`, but lines are always one pixel wide.                                                                                     |
| `TRIANGLES`         | Ogni elemento è fatto da 3 vertici, formando un triangolo.                                                                                          |
| `TRIANGLE_STRIP`    | Inizia con 3 vertici per il primo triangolo. Ogni vertice aggiuntivo forma un nuovo triangolo con gli ultimi due vertici.           |
| `TRIANGLE_FAN`      | Inizia con 3 vertici per il primo triangolo. Ogni vertice aggiuntivo forma un triangolo con il primo e l'ultimo vertice.            |
| `QUADS`             | Ogni elemento è fatto da 4 vertici, formando un quadrilatero.                                                                                       |

### Scrivere al `BufferBuilder` {#writing-to-the-bufferbuilder}

Una volta che il `BufferBuilder` è inizializzato, puoi scriverci dei dati.

Il `BufferBuilder` permette di costruire il nostro buffer, un vertice dopo l'altro. To add a vertex, we use the `buffer.addVertex(Matrix4f, float, float, float)` method. The `Matrix4f` parameter is the transformation matrix, which we'll discuss in more detail later. I tre parametri float rappresentano le coordinate (x, y, z) della posizione del vertice.

Questo metodo restituisce un costruttore di vertice, che possiamo usare per specificare informazioni addizionali per il vertice. È cruciale seguire l'ordine del nostro `VertexFormat` definito quando aggiungiamo questa informazione. Se non lo facciamo, OpenGL potrebbe non interpretare i nostri dati correttamente. Dopo aver finito la costruzione di un vertice, se vuoi puoi continuare ad aggiungere altri vertici e dati al buffer.

Importante è anche capire il concetto di culling. Il culling è il processo con cui si rimuovono facce di una forma 3D che non sono visibili dalla prospettiva dell'osservatore. Se i vertici per una faccia sono specificati nell'ordine sbagliato, la faccia potrebbe non essere renderizzata correttamente a causa del culling.

#### Cos'è una Matrice di Trasformazione? {#what-is-a-transformation-matrix}

Una matrice di trasformazione è una matrice 4x4 che viene usata per trasformare un vettore. In Minecraft, the transformation matrix is just transforming the coordinates we give into the `addVertex` call. Le trasformazioni possono scalare il nostro modello, muoverlo e ruotarlo.

A volte viene chiamata anche matrice di posizione, o matrice modello.

It's usually obtained via the `Matrix3x2fStack` class, which can be obtained via the `GuiGraphics` object by calling the `GuiGraphics#pose()` method.

#### Renderizzare una Striscia di Triangoli {#rendering-a-triangle-strip}

Spiegare come scrivere al `BufferBuilder` è più semplice con un esempio pratico. Let's say we want to render something using the `VertexFormat.Mode.TRIANGLE_STRIP` draw mode and the `POSITION_COLOR` vertex format.

Disegneremo vertici nelle seguenti posizioni sul HUD (in ordine):

```text:no-line-numbers
(20, 20)
(5, 40)
(35, 40)
(20, 60)
```

Questo dovrebbe darci un diamante carino - siccome stiamo usando la modalità di disegno `TRIANGLE_STRIP`, il renderizzatore eseguirà i seguenti passaggi:

![Quattro passaggi che mostrano il posizionamento dei vertici sullo schermo per formare due triangoli](/assets/develop/rendering/concepts-practical-example-draw-process.png)

Siccome stiamo disegnando sulla HUD in questo esempio, useremo la `HudElementRegistry`:

:::warning IMPORTANT UPDATE

Starting from 1.21.8, the matrix stack passed for HUD rendering has been changed from `PoseStack` to `Matrix3x2fStack`. Most methods are slightly different and no longer take a `z` parameter, but the concepts are the same.

Additionally, the code below does not fully match the explanation above: you do not need to manually write to the `BufferBuilder`, because `GuiGraphics` methods automatically write to the HUD's `BufferBuilder` during preparation.

Read the important update above for more information.

:::

**Element registration:**

@[code lang=java transcludeWith=:::registration](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

**Implementation of `hudLayer()`:**

@[code lang=java transcludeWith=:::hudLayer](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

Questo risulta nel seguente disegno sul HUD:

![Risultato Finale](/assets/develop/rendering/concepts-practical-example-final-result.png)

::: tip

Prova a giocare coi colori e le posizioni dei vertici per vedere che succede! Puoi anche provare a usare modalità di disegno e formati vertice differenti.

:::

## The `PoseStack` {#the-posestack}

::: warning

This section's code and the text are discussing different things!

The code showcases `Matrix3x2fStack`, which is used for HUD rendering since 1.21.8, while the text describes `PoseStack`, which has slightly different methods.

Read the important update above for more information.

:::

Dopo aver imparato come scrivere al `BufferBuilder`, ti starai chiedendo come trasformare il tuo modello - anche animarlo magari. This is where the `PoseStack` class comes in.

The `PoseStack` class has the following methods:

- `pushPose()` - Pushes a new matrix onto the stack.
- `popPose()` - Pops the top matrix off the stack.
- `last()` - Returns the top matrix on the stack.
- `translate(x, y, z)` - Trasla la matrice in cima allo stack.
- `translate(vec3)`
- `scale(x, y, z)` - Scala la matrice in cima allo stack.

Puoi anche moltiplicare la matrice in cima allo stack usando i quaternioni, che tratteremo nella prossima sezione.

Taking from our example above, we can make our diamond scale up and down by using the `PoseStack` and the `tickDelta` - which is the "progress" between the last game tick and the next game tick. Chiariremo questo ulteriormente nella pagina [Rendering nel HUD](./hud#render-tick-counter).

::: warning

You must first push the matrix stack and then pop it after you're done with it. If you don't, you'll end up with a broken matrix stack, which will cause rendering issues.

Assicurati di spingere lo stack di matrici prima di prendere una matrice di trasformazione!

:::

@[code lang=java transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

![Un video che mostra il diamante ingrandirsi e rimpicciolirsi](/assets/develop/rendering/concepts-matrix-stack.webp)

## Quaternioni (Cose che Ruotano) {#quaternions-rotating-things}

::: warning

This section's code and the text are discussing different things!

The code showcases rendering on the HUD, while the text describes rendering the 3D world space.

Read the important update above for more information.

:::

I quaternioni sono un modo di rappresentare rotazioni in uno spazio 3D. They are used to rotate the top matrix on the `PoseStack` via the `rotateAround(quaternionfc, x, y, z)` method.

It's highly unlikely you'll need to ever use a Quaternion class directly, since Minecraft provides various pre-built Quaternion instances in its `Axis` utility class.

Let's say we want to rotate our square around the z-axis. We can do this by using the `PoseStack` and the `rotateAround(quaternionfc, x, y, z)` method.

@[code lang=java transcludeWith=:::3](@/reference/latest/src/client/java/com/example/docs/rendering/RenderingConceptsEntrypoint.java)

Il risultato è il seguente:

![Un video che mostra il diamante che ruota attorno all'asse z](/assets/develop/rendering/concepts-quaternions.webp)
