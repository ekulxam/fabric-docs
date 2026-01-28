---
title: Renderer dei Blocchi-Entità
description: Impara come vivacizzare il rendering con i renderer di blocchi-entità.
authors:
  - natri0
---

A volte non basta usare il formato del modello di Minecraft. Se devi aggiungere del rendering dinamico all'aspetto del tuo blocco, dovrai usare un `BlockEntityRenderer`.

Per esempio, facciamo in modo che il Blocco Contatore dell'[articolo sui Blocchi-Entità](../blocks/block-entities) mostri il numero di clic sulla sua faccia superiore.

## Creare un BlockEntityRenderer {#creating-a-blockentityrenderer}

Il rendering di blocchi-entità utilizza un sistema di presenta/renderizza dove prima presenti i dati necessari a renderizzare un oggetto a schermo, poi il gioco renderizza l'oggetto usando lo stato presentato.

Nel creare un `BlockEntityRenderer` per il `CounterBlockEntity`, è importante inserire la classe nell'insieme delle fonti corretto, come `src/client/`, se il tuo progetto divide gli insiemi delle fonti tra client e server. Accedere a classi legate al rendering direttamente nell'insieme delle fonti `src/main/` non è sicuro perché quelle classi potrebbero essere caricare su un server.

Prima di tutto, dobbiamo creare un `BlockEntityRenderState` per il nostro `CounterBlockEntity` per contenere i dati che verrano usati per il rendering. In questo caso, sarà necessario che i `clicks` siano disponibili durante il rendering

@[code transcludeWith=::render-state](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderState.java)

Poi creiamo un `BlockEntityRenderer` per il nostro `CounterBlockEntity`.

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

The new class has a constructor with `BlockEntityRendererProvider.Context` as a parameter. Il `Context` ha alcune utilità per il rendering, come l'`ItemRenderer` o il `TextRenderer`.
Also, by including a constructor like this, it becomes possible to use the constructor as the `BlockEntityRendererProvider` functional interface itself:

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/ExampleModBlockEntityRenderer.java)

Sovrascriveremo alcuni metodi per impostare lo stato di rendering, oltre al metodo `render` dove verrà impostata la logica di rendering.

`createRenderState` può essere usato per inizializzare lo stato di rendering.

@[code transclude={31-34}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

`extractRenderState` can be used to update the render state with entity data.

@[code transclude={36-42}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

Dovresti registrare i renderer dei blocchi-entità nella tua classe `ClientModInitializer`.

`BlockEntityRenderers` is a registry that maps each `BlockEntityType` with custom rendering code to its respective `BlockEntityRenderer`.

## Disegnare su Blocchi {#drawing-on-blocks}

Ora che abbiamo un renderer, possiamo disegnare. Il metodo `render` viene chiamato ad ogni frame, ed è qui che la magia del rendering avviene.

### Orientarsi {#moving-around}

Anzitutto, dobbiamo bilanciare e ruotare il testo in modo che sia sul lato superiore del blocco.

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
