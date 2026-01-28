---
title: Configurazione della Generazione di Dati
description: Una guida per configurare la Generazione di Dati con l'API di Fabric.
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

## Cos'è la Generazione di Dati? {#what-is-data-generation}

La generazione di dati (datagen) è un'API per generare programmaticamente ricette, progressi, tag, modelli di oggetti, file di lingua, loot table, e praticamente qualsiasi cosa basata su JSON.

## Attivare la Generazione di Dati {#enabling-data-generation}

### Durante la Creazione del Progetto {#enabling-data-generation-at-project-creation}

Il modo più semplice per attivare la datagen è durante la creazione del progetto. Attiva la casella "Enable Data Generation" mentre usi il [generatore di mod modello](https://fabricmc.net/develop/template).

![La casella "Data Generation" attiva nel generatore di mod modello](/assets/develop/data-generation/data_generation_setup_01.png)

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
