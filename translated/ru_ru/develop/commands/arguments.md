---
title: Аргументы команд
description: Узнайте, как создавать команды со сложными аргументами.
---

Большинство команд используют аргументы. Иногда они могут быть необязательными, что означает, что команда выполнится, даже если вы не предоставите этот аргумент. Один узел может иметь несколько типов аргументов, но будьте внимательны, чтобы избежать неоднозначности.

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

В этом случае после текста команды `/command_with_arg` следует ввести целое число. Например, если вы запустите `/command_with_arg 3`, вы получите сообщение:

> Вызывается /command_with_arg со значением = 3

Если ввести /command_with_arg без аргументов, команду не удастся правильно обработать.

Далее мы добавим необязательный второй аргумент:

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Теперь вы можете указать одно или два целых числа. Если вы укажете одно число, будет выведено сообщение с одним значением. Если вы укажете два числа, будет выведено сообщение с двумя значениями.

Возможно, вам покажется излишним дважды указывать схожие исполнения. Поэтому мы можем создать метод, который будет использоваться в обоих случаях.

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Собственные типы аргументов {#custom-argument-types}

Если в стандартной библиотеке нет нужного вам типа аргументов, вы можете создать свой. Для этого нужно создать класс, который наследуется от интерфейса `ArgumentType<T>`, где `T` — это тип аргумента.

Вам нужно будет реализовать метод `parse`, который преобразует входную строку в нужный тип.

Например, вы можете создать тип аргумента, который преобразует строку в `BlockPos` с форматом: `{x, y, z}`

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### Регистрация своих типов аргументов {#registering-custom-argument-types}

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
