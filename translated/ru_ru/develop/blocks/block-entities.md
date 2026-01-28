---
title: Сущности блока
description: Узнайте, как создавать блочные сущности для своих пользовательских блоков.
authors:
  - natri0
---

Сущности блока - это способ хранения дополнительных данных для блока, которые не являются частью состояния блока: содержимое инвентаря, пользовательское имя и так далее.
В Minecraft используются блочные сущности для таких блоков, как сундуки, печи и командные блоки.

В качестве примера мы создадим блок, который будет подсчитывать, сколько раз на нем щелкнули ПКМ.

## Создание сущности блока {#creating-the-block-entity}

Чтобы Minecraft распознал и загрузил новые сущности блока, нам нужно создать тип сущности блока. Это делается путем расширения класса `BlockEntity` и регистрации его в новом классе `ModBlockEntities`.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Регистрация `BlockEntity` дает `BlockEntityType`, подобный `COUNTER_BLOCK_ENTITY`, который мы использовали выше:

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/entity/ModBlockEntities.java)

::: tip

Обратите внимание, что конструктор `CounterBlockEntity` принимает два параметра, а конструктор `BlockEntity` - три: `BlockEntityType`, `BlockPos` и `BlockState`.
Если бы мы жёстко не закодировали `BlockEntityType`, класс `ModBlockEntities` не скомпилировался бы! Это происходит потому, что `BlockEntityFactory`, является функциональным интерфейсом, описывает функцию, которая принимает только два параметра, как и наш конструктор.

:::

## Создание блока {#creating-the-block}

Next, to actually use the block entity, we need a block that implements `EntityBlock`. Давайте создадим такой блок и назовем его `CounterBlock`.

::: tip

К этому можно подойти двумя способами:

- create a block that extends `BaseEntityBlock` and implement the `createBlockEntity` method
- create a block that implements `EntityBlock` by itself and override the `createBlockEntity` method

We'll use the first approach in this example, since `BaseEntityBlock` also provides some nice utilities.

:::

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/CounterBlock.java)

Using `BaseEntityBlock` as the parent class means we also need to implement the `createCodec` method, which is rather easy.

В отличие от блоков, которые являются сингл тонами, для каждого экземпляра блока создается новая сущность блока. Это делается с помощью метода `createBlockEntity`, который принимает позицию и `BlockState`, и возвращает `BlockEntity`, или `null`, если его не должно быть.

Не забудьте зарегистрировать блок в классе `ModBlocks`, как в [Создайте свой первый блок].(../blocks/first-block) guide:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

## Использование сущности блока {#using-the-block-entity}

Теперь, когда у нас есть сущность блока, мы можем использовать её для хранения кол-ва раз, когда на блоке щёлкали ПКМ. Для этого мы добавим поле `clicks` в класс `CounterBlockEntity`:

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Метод `setChanged`, используемый в `incrementClicks`, сообщает игре, что данные этой сущности были обновлены; это может быть полезно, когда мы добавим методы для сериализации счётчика и загрузки его обратно из файла сохранения.

Далее нам нужно увеличивать это поле каждый раз, когда на блоке щелкают ПКМ. Это делается путём переопределения метода `useWithoutItem` в классе `CounterBlock`:

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/CounterBlock.java)

Since the `BlockEntity` is not passed into the method, we use `level.getBlockEntity(pos)`, and if the `BlockEntity` is not valid, return from the method.

!["Вы нажали на блок в 6-й раз" сообщение на экране после щелчка ПКМ](/assets/develop/blocks/block_entities_1.png)

## Сохранение и загрузка данных {#saving-loading}

Теперь, когда у нас есть функциональный блок, мы должны сделать так, чтобы счетчик не сбрасывался между перезапусками игры. Это делается путем сериализации в NBT при сохранении игры и десериализации при ее загрузке.

Saving to NBT is done through `ValueInput`s and `ValueOutput`s. Эти классы отвечают за сохранение ошибок с кодирования/декодирования, а также за отслеживание реестров во время процесса сериализации.

You can read from a `ValueInput` using the `read` method, passing in a `Codec` for the desired type. Likewise, you can write to a `ValueOutput` by using the `store` method, passing in a Codec for the type, and the value.

Также есть методы для примитивных типов, такие как `getInt`, `getShort`, `getBoolean` и т. д. для записи и `putInt`, `putShort`, `putBoolean` и т. д. для записи. The View also provides methods for working with lists, nullable types, and nested objects.

Сериализация выполняется с помощью метода `saveAdditional`:

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Here, we add the fields that should be saved into the passed `ValueOutput`: in the case of the counter block, that's the `clicks` field.

Reading is similar, you get the values you saved previously from the `ValueInput`, and save them in the BlockEntity's fields:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Теперь, если мы сохраним и перезагрузим игру, блок счетчика должен продолжиться с того места, на котором он остановился при сохранении.

While `saveAdditional` and `loadAdditional` handle saving and loading to and from disk, there is still an issue:

- Сервер знает правильное значение `clicks`.
- Клиент не получает правильное значение при загрузке чанка.

Чтобы исправить это, мы переопределяем `getUpdateTag`:

@[code transcludeWith=:::7](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Теперь, когда игрок входит в игру или перемещается в чанк, где есть блок, он сразу же увидит правильное значение счетчика.

## Тики {#tickers}

The `EntityBlock` interface also defines a method called `getTicker`, which can be used to run code every tick for each instance of the block. Мы можем реализовать это, создав статический метод, который будет использоваться в качестве `BlockEntityTicker`:

Метод `getTicker` также должен проверить, совпадает ли переданный `BlockEntityType` с тем, который мы используем, и если да, то вернуть функцию, которая будет вызываться каждый тик. Thankfully, there is a utility function that does the check in `BaseEntityBlock`:

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/CounterBlock.java)

`CounterBlockEntity::tick` - это ссылка на статический метод `tick`, который мы должны создать в классе `CounterBlockEntity`. Структурировать его таким образом не обязательно, но это хорошая практика, чтобы сохранить код чистым и организованным.

Допустим, мы хотим сделать так, чтобы счетчик увеличивался только раз в 10 тиков (2 раза в секунду). Мы можем сделать это, добавив поле `ticksSinceLast` в класс `CounterBlockEntity` и увеличивая его с каждым тиком:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

Не забудьте сериализовать и десериализовать это поле!

Теперь мы можем использовать `ticksSinceLast`, чтобы проверить, можно ли увеличить счетчик в `incrementClicks`:

@[code transcludeWith=:::6](@/reference/latest/src/main/java/com/example/docs/block/entity/custom/CounterBlockEntity.java)

::: tip

Если блок-сущность не отображается, попробуйте проверить регистрационный код! Он должен передать в `BlockEntityType.Builder` блоки, которые действительны для этой сущности, иначе он выдаст предупреждение в консоли:

```log
[13:27:55] [Server thread/WARN] (Minecraft) Block entity example-mod:counter @ BlockPos{x=-29, y=125, z=18} state Block{example-mod:counter_block} invalid for ticking:
```

:::

<!---->
