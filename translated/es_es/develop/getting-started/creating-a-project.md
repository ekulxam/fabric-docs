---
title: Creando un Proyecto
description: Una guía paso a paso sobre como crear un proyecto de mod usando el generador de plantillas de mods de Fabric.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric provee una manera fácil de crear un nuevo proyecto de mod usando el Generador de Plantillas de Mods de Fabric - si quieres, puedes crear un nuevo proyecto manualmelnte usando el repositorio del mod de ejemplo, deberías visitar la sección de [Creación Manual de Proyectos](#manual-project-creation).

## Generando un Proyecto

Puedes usar el [Generador de Plantillas de Mods de Fabric](https://fabricmc.net/develop/template/) para generar un nuevo proyecto para tu mod - deberías llenar los campos requeridos, como el nombre del paquete y el nombre del mod, y la versión de Minecraft para la cual quieres desarrollar.

The package name should be lowercase, separated by dots, and unique to avoid conflicts with other programmers' packages. It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![Vista previa del generador](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Sección de Opciones Avanzadas](/assets/develop/getting-started/template-generator-advanced.png)

Una vez llenados los campos requeridos, haz clic en el botón de `Generate` (Generar), y el generador creará un nuevo proyecto para ti en la forma de archivo zip.

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## Creación Manual de Proyectos

:::info PREREQUISITES

Necesitarás tener instalado [Git](https://git-scm.com/) para clonar el repositorio del mod de ejemplo.

:::

Si no puedes usar el Generador de Plantillas de Mods de Fabric, puedes crear un nuevo proyecto siguiendo los siguientes pasos.

Primero, clona el repositorio del mod de ejemplo usando Git:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. Si el folder de `.git` no aparece, deberías habilitar la visualización de archivos ocultos en tu administrador de archivos.

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

Nuevamente, como ya hemos dicho, si recibes una notificación que habla sobre un _script_ de Gradle, deberías hacer clic en el botón de `Import Gradle Project` (Importar Proyecto de Gradle).

### Modificando La Plantilla

Una vez que el proyecto ha sido importado, deberías poder modificar los detalles del proyecto para que encajen con los detalles de tu mod:

- Modifica el archivo de `gradle.properties` de tu proyecto para cambiar las propiedades de `maven_group` (grupo de maven) y `archive_base_name` (nombre base del archivo) para que sean los de tu mod.
- Modifica el archivo de `fabric.mod.json` para cambiar las propiedades de `id`, `name`, y `description` para que sean los de tu mod.
- Asegúrate de actualizar las versiones de Minecraft, los mapeos, el cargador de Fabric y Loom - todos ellos pueden ser consultados en <https://fabricmc.net/develop/> - para tener las versiones deseadas.

Obviamente también puedes cambiar el nombre del paquete y la clase principal para que sean las de tu mod.
