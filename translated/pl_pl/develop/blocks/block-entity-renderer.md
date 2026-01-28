---
title: Rendery Bytów Bloków
description: Dowiedz się, jak urozmaicić renderowanie za pomocą rendererów bytów bloków.
authors:
  - natri0, Maksiurino
---

Czasami korzystanie z formatu modelu Minecrafta nie wystarczy. Jeśli chcesz dodać dynamiczne renderowanie do elementów wizualnych swojego bloku, musisz użyć `BlockEntityRenderer`.

Na przykład, sprawmy, aby Counter Block z [artykułu o Bytów Bloków](../blocks/block-entities) wyświetlał liczbę kliknięć na swojej górnej ścianie.

## Tworzenie BlockEntityRenderer {#creating-a-blockentityrenderer}

Block entity rendering uses a submit/render system where you first submit the data required to render an object to the screen, the game then renders the object using it's submitted state.

Podczas tworzenia `BlockEntityRenderer` dla `CounterBlockEntity` ważne jest, aby umieścić class w odpowiednim zestawie źródłowym, takim jak `src/client/`, jeśli twój projekt używa "split client and common sources". Uzyskiwanie dostępu do classów związanych z renderowaniem bezpośrednio w zestawie źródłowym `src/main` nie jest bezpieczne, gdyż classy te mogą być ładowane na serwerze.

First, we need to create a `BlockEntityRenderState` for our `CounterBlockEntity` to hold the data that will be used for rendering. In this case, we will need the `clicks` to be available during rendering.

@[code transcludeWith=::render-state](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderState.java)

Then we create a `BlockEntityRenderer` for our `CounterBlockEntity`.

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

The new class has a constructor with `BlockEntityRendererProvider.Context` as a parameter. Ten `Context` ma kilka przydatnych narzędzi renderujących, takich jak `ItemRenderer` lub `TextRenderer`.
Also, by including a constructor like this, it becomes possible to use the constructor as the `BlockEntityRendererProvider` functional interface itself:

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/ExampleModBlockEntityRenderer.java)

We will override a few methods to set up the render state along with the `render` method where the rendering logic will be set up.

`createRenderState` can be used to initialize the render state.

@[code transclude={31-34}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

`extractRenderState` can be used to update the render state with entity data.

@[code transclude={36-42}](@/reference/latest/src/client/java/com/example/docs/rendering/blockentity/CounterBlockEntityRenderer.java)

Powinieneś zarejestrować renderery bytów bloków w classie `ClientModInitializer`.

`BlockEntityRenderers` is a registry that maps each `BlockEntityType` with custom rendering code to its respective `BlockEntityRenderer`.

## Rysowanie na Blokach {#drawing-on-blocks}

Teraz, jeżeli mamy renderer, możemy zacząć rysować. Metoda `render` jest przywoływana co klatkę, i tu się dzieje magia.

### Poruszanie się {#moving-around}

Najpierw musimy przesunąć i obrócić tekst tak, aby znalazł się na górnej stronie bloku.

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
