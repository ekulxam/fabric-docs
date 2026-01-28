---
title: 데이터 생성 설정
description: Fabric API를 통한 데이터 생성 설정 길라잡이.
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

## 데이터 생성이 무엇인가요? {#what-is-data-generation}

데이터 생성 (혹은 Datagen)은 제작법, 발전 과제, 태그, 아이템 모델, 언어 파일, 노획물 목록, 그리고 JSON 기반의 거의 모든 것을 프로그래밍의 방식으로 생성하기 위한 API입니다.

## 데이터 생성 활성화 {#enabling-data-generation}

### 프로젝트를 생성할 때 {#enabling-data-generation-at-project-creation}

Datagen을 활성화하는 가장 편리한 방법은 바로 프로젝트 제작입니다. [템플릿 생성기](https://fabricmc.net/develop/template/)를 사용하려는 경우 "Enable Data Generation" 박스체크를 누르세요.

![템플릿 생성기에서의 박스체크된 "Data Generation" 박스](/assets/develop/data-generation/data_generation_setup_01.png)

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
