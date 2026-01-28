---
title: 命令参数
description: 学习如何创建带有复杂参数的命令。
---

大多数命令都使用了参数。 有时参数是可选的，也就是说如果你不提供此参数，命令仍能运行。 一个节点可以有多个参数类型，但是注意有可能出现二义性，这是需要避免的。

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

在这个例子中，在命令文本 `/command_with_arg` 之后，你需要输入一个整数。 例如，如果运行 `/command_with_arg 3`，会收到反馈消息：

> 调用了 /command_with_arg 其中 value = 3

如果你输入 `/command_with_arg` 不带参数，命令无法正确解析。

接下来我们将添加第二个可选的参数：

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

现在你可以输入一个或者两个整数了。 如果提供了一个整数，那么会打印单个值的反馈文本。 如果提供了两个整数，那么会打印有两个值的反馈文本。

你可能发现，两次指定类似的执行内容有些不太必要。 因此，我们可以创建一个在两个执行中都使用的方法。

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## 自定义参数类型{#custom-argument-types}

如果原版没有你想要的参数类型，可以自己创建一个。 为此，创建一个类并继承 `ArgumentType<T>` 接口，其中 `T` 是参数的类型。

您需要实现 `parse` 这个方法，这个方法会把输入的字符串解析为期望的类型。

举个例子，您可以创建一个可以把格式形如 `{x, y, z}` 的字符串解析为一个 `BlockPos` 参数类型。

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### 注册自定义参数类型{#registering-custom-argument-types}

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
