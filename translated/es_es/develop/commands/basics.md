---
title: Crear Comandos
description: Crea comandos con argumentos y acciones complejas.
authors:
  - atakku
  - dicedpixels
  - haykam
  - i509VCB
  - Juuxel
  - MildestToucan
  - modmuss50
  - mschae23
  - natanfudge
  - pyrofab
  - solidblock
  - technici4n
  - treeways
  - xpple
---

La creación de comandos le permite a desarrolladores de mods añadir funcionalidad que puede ser usada mediante un comando. Este tutorial te enseñará como registrar comandos y la estructura general de comandos de Brigadier.

::: info

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based command library where
you build a tree of commands and arguments.

Brigadier is open-source: <https://github.com/Mojang/brigadier>

:::

## The `Command` Interface {#the-command-interface}

`com.mojang.brigadier.Command` es una interfaz funcional, la cual corre un código específico, y tira una excepción de `CommandSyntaxException` (Excepción de Syntax de Comando) en algunos casos. Tiene un tipo genérico `S`, el cual define el tipo de la _fuente de comando_.
La fuente de comando nos dá el contexto en el que se corrió un comando. In Minecraft, the command source is typically a
`CommandSourceStack` which can represent a server, a command block, a remote connection (RCON), a player or an entity.

El método `run(CommandContext<S>)` en `Command` tiene un `CommandContext<S>` como el único parámetro y retorna un número entero. El contexto del comando contiene la fuente de comando de `S` y te permite obtener los argumentos, ver los nodos de comando analizados y ver el valor entrado en este comando.

Al igual que otras interfaces funcionales, puedes usar una expresión Lambda o una referencia de método:

```java
Command<CommandSourceStack> command = context -> {
    return 0;
};
```

El número entero retornado puede ser considerado el resultado del comando. Los valores iguales o menores a 0 típicamente significan que el comando ha fallado y no hará nada. Valores positivos indican que el comando fue exitoso e hizo algo. Brigadier provee una constante para indicar éxito; `Command#SINGLE_SUCCESS` (Éxito Único).

### What Can the `CommandSourceStack` Do? Brigadier es un analizador y despachador de comandos escrito por Mojang para Minecraft.

A `CommandSourceStack` provides some additional implementation-specific context when a command is run. Esto incluye la habilidad de obtener la entidad que ejecutó el comando, el mundo o el servidor en el que el comando fue ejecutado.

Puedes acceder la fuente del comando desde un contexto de comando llamando `getSource()` en la instancia de `CommandContext`.

```java
Command<CommandSourceStack> command = context -> {
    CommandSourceStack source = context.getSource();
    return 0;
};
```

## Registrar un Comando Básico

Los comandos son registrados mediante la clase `CommandRegistrationCallback` (Callback de Registración de Comandos) proveída por el Fabric API.

::: info

Para información sobre como registrar callbacks, por favor visita la guía de [Eventos](../events).

:::

The event should be registered in your [mod's initializer](../getting-started/project-structure#entrypoints).

El callback tiene tres parámetros:

- `CommandDispatcher<CommandSourceStack> dispatcher` - Used to register, parse and execute commands. `S` es el tipo de la fuente del comando que el despachador usa.
- `CommandBuildContext registryAccess` - Provides an abstraction to registries that may be passed to certain command
  argument methods
- `Commands.CommandSelection environment` - Identifies the type of server the commands are being registered
  on.

En el inicializador de mod, solo registramos un comando simple:

@[code lang=java transcludeWith=:::test_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

In the `sendSuccess()` method, the first parameter is the text to be sent, which is a `Supplier<Component>` to avoid
instantiating `Component` objects when not needed.

The second parameter determines whether to broadcast the feedback to other
moderators. Generalmente, si el comando solo es para verificar o consultar algo sin afectar realmente el mundo, como verificar el tiempo actual o el puntaje de un jugador, este parametro debería ser `false` (falso). Si el comando hace algo, como cambiar el tiempo o modificar el puntaje de alguien, entonces debería ser `true` (verdadero).

If the command fails, instead of calling `sendSuccess()`, you may directly throw any exception and the server or client
will handle it appropriately.

`CommandSyntaxException` es generalmente tirada para indicar errores en la sintaxis del comando o sus argumentos. También puedes implementar tus propias excepciones.

Para ejecutar este comando, debes escribir `/foo`; aquí importan las mayúsculas y minúsculas.

::: info

From this point onwards, we will be extracting the logic written within the lambda passed into `.executes()` builders into individual methods. We can then pass a method reference to `.executes()`. This is done for clarity.

:::

### Ambiente de Registración

Si se desea, también puedes asegurarte que un comando solo es registrado bajo ciertas circunstancias específicas, por ejemplo, solo en el ambiente dedicado:

@[code lang=java highlight={2} transcludeWith=:::dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Requerimientos de Comandos

Let's say you have a command that you only want moderators to be able to execute. Aquí entra el método `requires()`. The `requires()` method has one argument of a `Predicate<S>` which will supply a `CommandSourceStack`
to test with and determine if the `CommandSource` can execute the command.

@[code lang=java highlight={3} transcludeWith=:::required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

This command will only execute if the source of the command is a moderator at a minimum, including command
blocks. De lo contrario, el comando no es registrado.

This has the side effect of not showing this command in <kbd>Tab</kbd> completion to anyone who is not a moderator. This is
also why you cannot <kbd>Tab</kbd>-complete most commands when you do not enable cheats.

### Sub Comandos

Para agregar un sub comando, registras el primer nodo de comando normalmente. Para tener un sub comando, tienes que adjuntar el siguiente literal de nodo de comando al nodo existente.

@[code lang=java highlight={3} transcludeWith=:::sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Similarmente a los argumentos, los nodos de sub comandos pueden ser opcionales. En el siguiente caso ambos comandos `/subtater` y `/subtater subcommand` serán válidos.

@[code lang=java highlight={2,8} transcludeWith=:::sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Comandos de Cliente

El Fabric API tiene una clase `ClientCommandManager` en el paquete de `net.fabricmc.fabric.api.client.command.v2` que puede ser usado para registrar comandos en el lado del cliente. Este código solo debe existir en el lado del cliente.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/client/command/ExampleModClientCommands.java)

## Redireccionando Comandos

Redireccionadores de comandos - también llamados aliases - son una manea de redireccionar la funcionalidad de un comando a otro. Esto es útil cuando quieres cambiar el nombre de un comando, pero todavía quieres mantener soporte para el nombre viejo.

::: warning

Brigadier [will only redirect command nodes with arguments](https://github.com/Mojang/brigadier/issues/46). If you want to redirect a command node without arguments, provide an `.executes()` builder with a reference to the same logic as outlined in the example.

:::

@[code lang=java transcludeWith=:::redirect_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_redirected_by](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Preguntas Frequentes

### ¿Por qué mi código no compila? Es una libraría basada en un una estructura de árbol de comandos, donde construyes un árbol de comandos y argumentos.

- Atrapa o tira una excepción `CommandSyntaxException` - `CommandSyntaxException` no es una `RuntimeException` (Excepción en Tiempo de Ejecución). Si la tiras, debe ser en métodos que tiren un `CommandSyntaxException` en el signature (firma) del método, o ser atrapadas.
  Brigadier manejará las excepciones checked (comprobadas) y mandará el mensaje de error correspondiente en el juego para ti.

- Problemas con genéricos - Puede que tengas problemas con genéricos de vez en cuando. If you are registering server
  commands (which are most of the case), make sure you are using `Commands.literal`
  or `Commands.argument` instead of `LiteralArgumentBuilder.literal` or `RequiredArgumentBuilder.argument`.

- Check the `sendSuccess()` method - You may have forgotten to provide a boolean as the second argument. Also remember
  that, since Minecraft 1.20, the first parameter is `Supplier<Component>` instead of `Component`.

- Un comando debería retornar un número entero - Cuando registres comandos, el método `executes()` acepta un objeto `Command`, el cual usualmente es una expresión Lambda. El Lambda debería retornar un número entero, en lugar de otros tipos de valores.

### ¿Puedo registrar comandos durante la ejecución? La interfaz `Comand` (Comando)

::: warning

You can do this, but it is not recommended. You would get the `Commands` from the server and add anything commands
you wish to its `CommandDispatcher`.

After that, you need to send the command tree to every player again
using `Commands.sendCommands(ServerPlayer)`.

This is required because the client locally caches the command tree it receives during login (or when moderator packets
are sent) for local completions-rich error messages.

:::

### ¿Puedo registrar comandos durante la ejecución? <br>

::: warning

You can also do this, however, it is much less stable than registering commands at runtime and could cause unwanted side
effects.

Para mantener las cosas simples, vas a tener que usar reflexión en Brigadier para remover nodos. After this, you need to send the
command tree to every player again using `sendCommands(ServerPlayer)`.

Si no envias el árbol de comandos actualizado, el cliente pensará que un comando existe, aunque el servidor no podrá ejecutarlo.

:::

<!---->
