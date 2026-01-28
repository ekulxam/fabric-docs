---
title: 数据生成设置
description: 使用 Fabric API 设置数据生成的指南。
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

## 数据生成是什么？ {#what-is-data-generation}

数据生成 (又称 Datagen) 是一种 API，用于以编程方式生成配方、进度、标签、物品模型、语言文件、战利品表以及基本上任何基于 JSON 的内容。

## 启用数据生成 {#enabling-data-generation}

### 在项目创建时 {#enabling-data-generation-at-project-creation}

启用数据生成的最简单方法是在创建项目时。 使用[模板生成器](https://fabricmc.net/develop/template/)时，勾选“启用数据生成”框。

![模板生成器上勾选的“数据生成”框](/assets/develop/data-generation/data_generation_setup_01.png)

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
