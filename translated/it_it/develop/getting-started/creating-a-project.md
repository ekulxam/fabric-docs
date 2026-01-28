---
title: Creare un Progetto
description: Una guida passo per passo su come creare un nuovo progetto per una mod con il generatore di mod modello di Fabric.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric offre un modo facile per creare un nuovo progetto per una mod attraverso il Generatore di Mod Modello di Fabric - se vuoi, puoi creare un nuovo progetto manualmente usando la repository della mod esempio, dovresti riferirti alla sezione [Creazione Manuale del Progetto](#creazione-manuale-del-progetto).

## Generare un Progetto {#generating-a-project}

Puoi usare il [Generatore di Mod Modello di Fabric](https://fabricmc.net/develop/template/) per generare un nuovo progetto per la tua mod - dovresti compilare i campi richiesti, come il nome della mod, quello del package, e la versione di Minecraft per la quale vuoi sviluppare.

Il nome del package dovrebbe essere minuscolo, separato da punti, e unico per evitare conflitti con package di altri programmatori. It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![Anteprima del generatore](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Sezione Opzioni Avanzate](/assets/develop/getting-started/template-generator-advanced.png)

Una volta che hai compilato i campi richiesti, premi il pulsante `Generate`, e il generatore creerà un nuovo progetto per te sotto forma di file zip.

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## Creazione Manuale del Progetto {#manual-project-creation}

:::info PREREQUISITES

Ti servirà che [Git](https://git-scm.com/) sia installato per clonare la repository della mod esempio.

:::

Se non puoi usare il Generatore di Mod Modello di Fabric, dovresti creare un nuovo progetto manualmente seguendo questi passaggi.

Anzitutto, clona la repository della mod esempio tramite Git:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. Se la cartella `.git` dovesse non apparire, dovresti attivare la visualizzazione dei file nascosti nel tuo gestore file.

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

Di nuovo, come già detto in precedenza, se ricevi una notifica riguardo a uno script di build Gradle, dovresti cliccare il pulsante `Importa Progetto Gradle`.

### Modificare il Template {#modifying-the-template}

Una volta che il progetto sarà importato, dovresti modificare i dettagli del progetto per corrispondere a quelli della tua mod:

- Modifica il file `gradle.properties` del tuo progetto per cambiare le proprietà `maven_group` e `archive_base_name` e farle corrispondere con i dettagli della tua mod.
- Modifica il file `fabric.mod.json` per cambiare le proprietà `id`, `name`, e `descrizione` per farle corrispondere ai dettagli della tua mod.
- Assicurati di aggiornare le versioni di Minecraft, i mapping, il Loader e il Loom - tutte queste possono essere trovate attraverso <https://fabricmc.net/develop/> - per farle corrispondere alle versioni che vorresti prendere di mira.

Ovviamente puoi cambiare il nome del package e la classe principale della mod per farli corrispondere ai dettagli della tua mod.
