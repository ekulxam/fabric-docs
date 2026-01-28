---
title: Estructura de Proyecto
description: Una descripción general de la estructura de un proyecto de mod de Fabric.
authors:
  - IMB11
---

Esta página cubrirá la estructura de un proyecto de mod de Fabric, y el propósito de cada archivo y folder en el proyecto.

## `fabric.mod.json`

El archivo `fabric.mod.json` es el archivo principal que describe tu mod al Lanzador de Fabric. Contiene información como el ID del mod, versión, y dependencias.

Los campos más importantes en el archivo `fabric.mod.json` son:

- `id`: El ID del mod, el cual debería ser único.
- `name`: El nombre del mod.
- `enviornment`: El entorno en el que tu mod será corrido en, como `client`, `server` o `*` para ambos.
- `entrypoints`: Los puntos de entrada que tu mod provee, como `main` o `client`.
- `depends:` Los mods en los que tu mod depende en.
- `mixins:` Los mixins que tu mod provee.

You can see an example `fabric.mod.json` file below - this is the `fabric.mod.json` file for the mod that powers this documentation site.

:::details `fabric.mod.json` of the Example Mod

@[code lang=json](@/reference/latest/src/main/resources/fabric.mod.json)

:::

## Puntos de Entrada

Como ya fue mencionado, el archivo `fabric.mod.json` contiene un campo llamado `entrypoints` - este campo es usado para especificar los puntos de entrada que tu mod usa.

The template mod generator creates both a `main` and `client` entrypoint by default:

- The `main` entrypoint is used for common code, and it's contained in a class which implements `ModInitializer`
- El generador de plantillas de mods crea los puntos de entrada `main` y `client` por defecto - el punto de entrada de `main` es usado para código común, mientras que el punto de entrada de `client` es usado para código exclusivo o específico para el cliente.

Estos puntos de entrada son llamados respectivamente cuando el juego comienza.

Lo anterior es un ejemplo de un punto de entrada de `main` simple, el cual manda un mensaje a la consola cuando el juego empieza.

@[code lang=java transcludeWith=#entrypoint](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

## `src/main/resources`

El folder `src/main/resources` es usado para guardar los recursos que tu mod usa, como las texturas, modelos, y sonidos.

Aquí también se puede encontrar el archivo `fabric.mod.json` y cualquier archivo de configuración de mixin que tu mod use.

Assets are stored in a structure that mirrors the structure of resource packs - for example, a texture for a block would be stored in `assets/example-mod/textures/block/block.png`.

## `src/client/resources`

El folder de `src/main/java` es usado para guardar el código fuente de Java para tu mod - existe tanto los entornos del cliente y el servidor.

## `src/client/java`

El folder de `src/client/java` es usado para guardar el código fuente de Java exclusivo del cliente, como código para renderización o lógica para el lado del cliente - como proveedores de colores de bloque.

## `src/main/java`

El folder de `src/client/resources` es usado para guardar recursos específicos al cliente, como las texturas, modelos, y sonidos que solo son usados en el lado del cliente.
