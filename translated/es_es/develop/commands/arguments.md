---
title: Argumentos de Comandos
description: Aprende a crear comandos con argumentos complejos.
---

Los argumentos son usados en la mayoría de los comandos. Algunas veces pueden ser opcionales, lo que significa que el usuario no tiene que dar un argumento para que el comando corra. Un nodo puede tener múltiples tipos de argumentos, pero ten cuidado de no crear ambigüedades.

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

En este caso, después del texto del comando `/argtater`, debes escribir un número entero. Por ejemplo, si corres `/argtater 3`, obtendrás el mensaje de respuesta `Called /argtater with value = 3`.

> Called /command_with_arg with value = 3

Si escribes `/argtater` sin argumentos, el comando no puede ser leído y analizado correctamente.

Ahora añadimos un segundo argumento opcional:

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Ahora puedes escribir uno o dós números enteros. Si le das un número entero, se mostrará un mensaje de respuesta con un solo valor. Si das dos números enteros, se mostrará un mensaje de respuesta con dos valores.

Puede que sientas que sea innecesario tener que especificar ejecuciones similares dos veces. Para ello, crearemos un método que se usará para ambas ejecuciones.

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Tipos de Argumentos Personalizados

Si el juego vanilla no tiene el tipo de argumento que necesitas, puedes crear tu propio tipo de argumento. Para esto, puedes crear una clase que implemente la interfaz `ArgumentType<T>`, donde `T` es el tipo del argumento.

Necesitarás implementar el método `parse`, el cual leerá y analizará la cadena de caracteres entrada por el usuario a el tipo de argumento deseado.

Por ejemplo, puedes crear un tipo de argumento que lea un `BlockPos` a partir de una cadena de caracteres con el siguiente formato: `{x, y, z}`

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### Registrar Tipos de Argumentos Personalizados

::: warning

You need to register the custom argument type on both the server and the client or else the command will not work!

:::

You can register your custom argument type in the `onInitialize` method of your [mod's initializer](../getting-started/project-structure#entrypoints) using the `ArgumentTypeRegistry` class:

@[code lang=java transcludeWith=:::register_custom_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Using Custom Argument Types {#using-custom-argument-types}

We can use our custom argument type in a command - by passing an instance of it into the `.argument` method on the command builder.

@[code lang=java highlight={3} transcludeWith=:::custom_arg_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java highlight={2} transcludeWith=:::execute_custom_arg_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Running the command, we can test whether or not the argument type works:

![Invalid argument](/assets/develop/commands/custom-arguments_fail.png)

![Valid argument](/assets/develop/commands/custom-arguments_valid.png)

![Command result](/assets/develop/commands/custom-arguments_result.png)
