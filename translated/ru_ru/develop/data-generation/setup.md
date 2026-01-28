---
title: Настройка генерации данных
description: Руководство по настройке генерации данных с помощью API Fabric.
authors:
  - ArkoSammy12
  - Earthcomputer
  - haykam821
  - Jab125
  - matthewperiut
  - modmuss50
  - Shnupbups
  - skycatminepokie
  - SolidBlock-cn
authors-nogithub:
  - jmanc3
  - mcrafterzz
---

## Что такое генерация данных? {#what-is-data-generation}

Генерация данных (или datagen) - это API для программной генерации рецептов, улучшений, тегов, моделей предметов, языковых файлов, таблиц лута и вообще всего, что основано на JSON.

## Активация генерации данных {#enabling-data-generation}

### При создании проекта {#enabling-data-generation-at-project-creation}

Проще всего включить datagen при создании проекта. Установите флажок "Включить генерацию данных" при использовании [генератора шаблонов] (https://fabricmc.net/develop/template/).

![Установленный флажок "Генерация данных" на генераторе шаблонов](/assets/develop/data-generation/data_generation_setup_01.png)

::: tip

If datagen is enabled, you should have a "Data Generation" run configuration and a `runDatagen` Gradle task.

:::

### Manually {#manually-enabling-data-generation}

First, we need to enable datagen in the `build.gradle` file.

@[code transcludeWith=:::datagen-setup:configure](@/reference/build.gradle)

Next, we need an entrypoint class. This is where our datagen starts. Place this somewhere in the `client` package - this example places it at `src/client/java/com/example/docs/datagen/ExampleModDataGenerator.java`.

@[code lang=java transcludeWith=:::datagen-setup:generator](@/reference/latest/src/client/java/com/example/docs/datagen/ExampleModDataGenerator.java)

Finally, we need to tell Fabric about the entrypoint in our `fabric.mod.json`:

<!-- prettier-ignore -->

```json
{
  // ...
  "entrypoints": {
    // ...
    "client": [
      // ...
    ],
    "fabric-datagen": [ // [!code ++]
      "com.example.docs.datagen.ExampleModDataGenerator" // [!code ++]
    ] // [!code ++]
  }
}
```

::: warning

Don't forget to add a comma (`,`) after the previous entrypoint block!

:::

Close and reopen IntelliJ to create a run configuration for datagen.

## Creating a Pack {#creating-a-pack}

Inside your datagen entrypoint's `onInitializeDataGenerator` method, we need to create a `Pack`. Later, you'll add **providers**, which put generated data into this `Pack`.

@[code lang=java transcludeWith=:::datagen-setup:pack](@/reference/latest/src/client/java/com/example/docs/datagen/ExampleModDataGenerator.java)

## Running Data Generation {#running-data-generation}

To run datagen, use the run configuration in your IDE, or run `./gradlew runDatagen` in the console. The generated files will be created in `src/main/generated`.

## Next Steps {#next-steps}

Now that datagen is set up, we need to add **providers**. These are what generate the data to add to your `Pack`. The following pages outline how to do this.

- [Advancements](./advancements)
- [Loot Tables](./loot-tables)
- [Recipes](./recipes)
- [Tags](./tags)
- [Translations](./translations)
- [Block Models](./block-models)
- [Item Models](./item-models)
