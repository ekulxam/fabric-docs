---
title: Argumentos de comandos
description: Aprenda como criar comandos com argumentos complexos.
---

Argumentos são usados na maioria dos comandos. Algumas vezes eles podem ser opcionais, o que significa que se você não colocar esse argumento, o comando ainda vai rodar. Um nó pode incluir vários tipos de argumentos, mas é preciso ter cuidado para evitar ambiguidades, que devem ser evitadas.

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Nesse caso, após o texto do comando `/command_with_arg`, você deve digitar um número inteiro. Por exemplo, se você executar `/command_with_arg 3`, você vai receber a mensagem de feedback:

> Called /command_with_arg with value = 3

Se você digitar `/command_with_arg` sem argumentos, o comando não poderá ser analisado corretamente.

Então, adicionamos um segundo argumento opcional:

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Agora você pode digitar um ou dois números inteiros. Se você fornecer um inteiro, um texto de feedback com um único valor será impresso. Se você fornecer dois inteiros, um texto de feedback com dois valores será impresso.

Você pode achar desnecessário especificar execuções semelhantes duas vezes. Portanto, podemos criar um método que será usado em ambas as execuções.

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Tipos de Argumento Personalizados {#custom-argument-types}

Se o vanilla não tiver o tipo de argumento que você precisa, você pode criar o seu próprio. Para fazer isso, é necessário criar uma classe que herde da interface `ArgumentType<T>`, onde `T` é o tipo de argumento.

Você precisará implementar o método `parse`, que irá analisar a string de entrada para o tipo desejado.

Por exemplo, você pode criar um tipo de argumento que analise um `BlockPos` a partir de uma string no seguinte formato: {x, y, z}\`

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### Registrando Tipos de Argumento Personalizados {#registering-custom-argument-types}

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
