---
title: Testing Automatizzato
description: Una guida per scrivere test automatici con Fabric Loader JUnit.
authors:
  - kevinthegreat1
---

Questa pagina spiega come scrivere codice che testi parti della tua mod automaticamente. Ci sono due modi per testare la tua mod automaticamente: unit test con Fabric Loader JUnit o test nel gioco con il framework Gametest di Minecraft.

Gli unit test dovrebbero essere usato per testare le componenti del tuo codice, come i metodi e le classi ausiliarie, mentre test nel gioco avviano un client e un server Minecraft reali per eseguire i tuoi test, il che li rende appropriati per testare funzionalità e gameplay.

## Unit Testing {#unit-testing}

Poiché il modding in Minecraft si affida a strumenti di modifica del byte-code da eseguire come i Mixin, aggiungere e usare JUnit normalmente non funzionerebbe. Questo è il motivo per cui Fabric fornisce Fabric Loader JUnit, un plugin JUnit che attiva lo unit testing in Minecraft.

### Configurare Fabric Loader JUnit {#setting-up-fabric-loader-junit}

Anzitutto, dobbiamo aggiungere Fabric Loader JUnit all'ambiente di sviluppo. Aggiungi il seguente blocco di dipendenze al tuo `build.gradle`:

@[code transcludeWith=:::automatic-testing:1](@/reference/build.gradle)

Poi, dobbiamo informare Gradle su come usare Fabric Loader JUnit per il testing. Puoi fare ciò aggiungendo il codice seguente al tuo `build.gradle`:

@[code transcludeWith=:::automatic-testing:2](@/reference/latest/build.gradle)

### Scrivere Test {#writing-tests}

Appena ricaricato Gradle, sei pronto a scrivere test.

Questi test si scrivono proprio come altri test JUnit regolari, con un po' di configurazione aggiuntiva se vuoi accedere a classi che dipendono dalle registry, come `ItemStack`. Se JUnit ti è familiare, puoi saltare a [Impostare le Registry](#setting-up-registries).

#### Impostare la Tua Prima Classe di Test {#setting-up-your-first-test-class}

I test si scrivono nella cartella `src/test/java`.

Una convenzione per i nomi è quella di rispecchiare la struttura del package della classe che stai testando. Per esempio, per testare `src/main/java/com/example/docs/codec/BeanType.java`, dovresti creare la classe `src/test/java/com/example/docs/codec/BeanTypeTest.java`. Nota che abbiamo aggiunto `Test` alla fine del nome della classe. Questo ti permette anche di accedere facilmente a metodi e attributi privati al package.

Un'altra convenzione è avere un package `test`, quindi `src/test/java/com/example/docs/test/codec/BeanTypeTest.java`. Questo previene alcuni problemi dovuti all'uso dello stesso package se usi i moduli Java.

After creating the test class, use <kbd>⌘/CTRL</kbd>+<kbd>N</kbd> to bring up the Generate menu. Seleziona Test e comincia a scrivere il nome del tuo metodo, che di solito inizia con `test`. Premi <kbd>Invio</kbd> quando hai fatto. For more tips and tricks on using the IDE, see [IDE Tips and Tricks](./getting-started/tips-and-tricks#code-generation).

![Generare un metodo di test](/assets/develop/misc/automatic-testing/unit_testing_01.png)

Puoi ovviamente scrivere la firma del metodo manualmente, e qualsiasi istanza del metodo senza parametri e come tipo restituito void sarà identificato come metodo di test. Dovresti alla fine avere il seguente:

![Un metodo di test vuoto con indicatori di test](/assets/develop/misc/automatic-testing/unit_testing_02.png)

Nota gli indicatori a freccia verde nel margine: puoi facilmente eseguire un test cliccandoci sopra. In alternativa, i tuoi test si eseguiranno in automatico ad ogni build, incluse le build di CI come GitHub Actions. Se stai usando GitHub Actions, non dimenticare di leggere [Configurare le GitHub Actions](#setting-up-github-actions).

Ora è il tempo di scrivere il tuo codice di test effettivo. Puoi assicurare condizioni con `org.junit.jupiter.api.Assertions`. Dai un'occhiata ai test seguenti:

@[code lang=java transcludeWith=:::automatic-testing:4](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

Per una spiegazione di cosa fa questo codice, consulta la pagina [Codec](./codecs#registry-dispatch).

#### Impostare le Registry {#setting-up-registries}

Ottimo, il primo test è funzionato! Ma aspetta, il secondo test è fallito? Nei log, otteniamo uno dei seguenti errori.

<<< @/public/assets/develop/automatic-testing/crash-report.log

Questo è perché stiamo provando ad accedere alla registry o a una classe che dipende su queste (o, in casi rari, dipende su altre classi Minecraft come `SharedConstants`), ma Minecraft non è stato inizializzato. Dobbiamo solo inizializzarlo un po' perché funzionino le registry. Ti basta aggiungere il codice seguente all'inizio del tuo metodo `beforeAll`.

@[code lang=java transcludeWith=:::automatic-testing:7](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

### Configurare le GitHub Actions {#setting-up-github-actions}

::: info

This section assumes that you are using the standard GitHub Action workflow included with the example mod and with the mod template.

:::

Your tests will now run on every build, including those by CI providers such as GitHub Actions. But what if a build fails? We need to upload the logs as an artifact so we can view the test reports.

Add this to your `.github/workflows/build.yaml` file, below the `./gradlew build` step.

```yaml
- name: Store reports
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: reports
    path: |
      **/build/reports/
      **/build/test-results/
```

## Game Tests {#game-tests}

Minecraft provides the game test framework for testing server-side features. Fabric additionally provides client game tests for testing client-side features, similar to an end-to-end test.

### Setting up Game Tests with Fabric Loom {#setting-up-game-tests-with-fabric-loom}

Both server and client game tests can be set up manually or with Fabric Loom. This guide will use Loom.

To add game tests to your mod, add the following to your `build.gradle`:

@[code transcludeWith=:::automatic-testing:game-test:1](@/reference/latest/build.gradle)

To see all available options, see [the Loom documentation on tests](./loom/fabric-api#tests).

#### Setting up Game Test Directory {#setting-up-game-test-directory}

::: info

You only need this section if you enabled `createSourceSet`, which is recommended. You can, of course, do your own Gradle magic, but you'll be on your own.

:::

If you enabled `createSourceSet` like the example above, your gametest will be in a separate source set with a separate `fabric.mod.json`. The module name defaults to `gametest`. Create a new `fabric.mod.json` in `src/gametest/resources/` as shown:

<<< @/reference/latest/src/gametest/resources/fabric.mod.json

Note that this `fabric.mod.json` expects a server game test at `src/gametest/java/com/example/docs/ExampleModGameTest`, and a client game test at `src/gametest/java/com/example/docs/ExampleModClientGameTest`.

### Writing Game Tests {#writing-game-tests}

You can now create server and client game tests in the `src/gametest/java` directory. Here is a basic example for each:

::: code-group

<<< @/reference/latest/src/gametest/java/com/example/docs/ExampleModGameTest.java [Server]

<<< @/reference/latest/src/gametest/java/com/example/docs/ExampleModClientGameTest.java [Client]

:::

See the respective Javadocs in Fabric API for more info.

### Running Game Tests {#running-game-tests}

Server game tests will be run automatically with the `build` Gradle task. You can run client game tests with the `runClientGameTest` Gradle task.

### Run Game Tests on GitHub Actions {#run-game-tests-on-github-actions}

Existing GitHub Action workflows using `build` will run server game tests automatically. To run client game tests with GitHub Actions, add the following snippet to your `build.gradle` and the following job to your workflow. The Gradle snippet will run client game tests using [Loom's production run tasks](./loom/production-run-tasks), and the job will execute the production run task in the CI.

::: warning

Currently, game test may fail on GitHub Actions due to an error in the network synchronizer. If you encounter this error, you can add `-Dfabric.client.gametest.disableNetworkSynchronizer=true` to the JVM args in your production run task declaration.

:::

@[code transcludeWith=:::automatic-testing:game-test:2](@/reference/latest/build.gradle)

@[code transcludeWith=:::automatic-testing:game-test:3](@/.github/workflows/build.yaml)
