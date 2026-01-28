---
title: Прозрачность и тонировка
description: Узнайте, как изменять внешний вид блоков и динамически окрашивать их.
authors:
  - cassiancc
  - dicedpixels
---

Иногда может возникнуть необходимость в особом обращении с внешним видом блоков в игре. Например, некоторые блоки могут казаться прозрачными, а на другие может быть нанесен оттенок.

Давайте посмотрим, как мы можем изменять внешний вид блока.

Для этого примера давайте зарегистрируем блок. Если вы не знакомы с этим процессом, сначала ознакомьтесь с информацией о [регистрации блока](./first-block).

@[code lang=java transcludeWith=:::block](@/reference/latest/src/main/java/com/example/docs/appearance/ExampleModAppearance.java)

Обязательно добавьте:

- [Состояние блока](./blockstates) в `/blockstates/waxcap.json`
- [модель](./block-models) в `/models/block/waxcap.json`
- [текстура](./first-block#models-and-textures) в `/textures/block/waxcap.png`

Если все правильно, вы сможете увидеть блок в игре. Однако вы заметите, что после размещения блок выглядит некорректно.

![Неправильный вид блока](/assets/develop/transparency-and-tinting/block_appearance_0.png)

Это связано с тем, что текстура с прозрачностью потребует некоторой дополнительной настройки.

## Манипулирование внешним видом блока {#manipulating-block-appearance}

Даже если текстура вашего блока прозрачная или полупрозрачная, он все равно будет выглядеть непрозрачным. Чтобы исправить это, вам необходимо настроить _Chunk Section Layer_ вашего блока.

Chunk Section Layer — это категории, используемые для группировки различных типов поверхностей блоков для рендеринга. Это позволяет игре использовать правильные визуальные эффекты и оптимизации для каждого типа.

Нам необходимо зарегистрировать наш блок с правильным уровнем секции блока. Vanilla предоставляет следующие возможности.

- `SOLID`: По умолчанию, сплошной блок без прозрачности.
- `CUTOUT` и `CUTOUT_MIPPED`: блок, использующий прозрачность, например Glass или Flowers. `CUTOUT_MIPPED` будет лучше смотреться на расстоянии.
- `TRANSLUCENT`: Блок, использующий полупрозрачные (частично прозрачные) пиксели, например, «Витраж» или «Вода».

В нашем примере используется прозрачность, поэтому будет использоваться `CUTOUT`.

В вашем **инициализаторе клиента** зарегистрируйте свой блок с правильным `ChunkSectionLayer`, используя `BlockRenderLayerMap` API Fabric.

@[code lang=java transcludeWith=:::block_render_layer_map](@/reference/latest/src/client/java/com/example/docs/appearance/ExampleModAppearanceClient.java)

Теперь ваш блок должен иметь надлежащую прозрачность.

![Правильный вид блока](/assets/develop/transparency-and-tinting/block_appearance_1.png)

## Поставщики цветов блоков {#block-color-providers}

Even though our block looks fine in-game, its texture is grayscale. We could dynamically apply a color tint, like how vanilla Leaves change color based on biomes.

Fabric API provides `ColorProviderRegistry` to register a tint color provider, which we'll use to dynamically color the block.

Let's use this API to register a tint such that, when our Waxcap block is placed on grass, it will look green, otherwise it'll look brown.

In your **client initializer**, register your block to the `ColorProviderRegistry`, along with the appropriate logic.

@[code lang=java transcludeWith=:::color_provider](@/reference/latest/src/client/java/com/example/docs/appearance/ExampleModAppearanceClient.java)

Now, the block will be tinted based on where its placed.

![Block With Color Provider](/assets/develop/transparency-and-tinting/block_appearance_2.png)
