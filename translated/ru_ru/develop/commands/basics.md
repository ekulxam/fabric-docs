---
title: Создание команд
description: Создавайте команды со сложными аргументами и действиями.
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

Создание команд позволяет разработчику мода добавлять функционал, который может быть использован при вызове команд. Это руководство научит вас регистрировать команды и общую структуру команд Brigadier.

::: info

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based command library where
you build a tree of commands and arguments.

Brigadier is open-source: <https://github.com/Mojang/brigadier>

:::

## Интерфейс `Command` {#the-command-interface}

`com.mojang.brigadier.Command` это функциональный интерфейс, который запускает конкретный код, и исключает `CommandSyntaxException` в определённых случаях. Он имеет общий тип `S`, который определяет тип _источник команды_.
Источник команды предоставляет контекст, в котором была запущена команда. In Minecraft, the command source is typically a
`CommandSourceStack` which can represent a server, a command block, a remote connection (RCON), a player or an entity.

Единственный метод в `Command`, это `run(CommandContext<S>)` он берёт `CommandContext<S>` в качестве единственного аргумента и возвращает целое число. Командный контекст содержит источник вашей команды как `S` и позволяет получить аргументы, посмотрите на разобранные командные ноды и увидите вводные данные, используемые в этой команде.

Как и другие функциональные интерфейсы, этот используется постоянно как лямбда или ссылка на метод:

```java
Command<CommandSourceStack> command = context -> {
    return 0;
};
```

Целое число может быть результатом команды. Обычно значения меньше или равные нулю означают, что команда не выполнена и ничего не сделает. Позитивные значения означают, что команда успешно выполнилась и что-то выполнила. Brigadier предоставляет константу для обозначения успеха; `Command#SINGLE_SUCCESS`.

### What Can the `CommandSourceStack` Do? {#what-can-the-servercommandsource-do}

A `CommandSourceStack` provides some additional implementation-specific context when a command is run. Это добавляет возможность получить сущность которая выполнила команду, мир в котором команда выполнилась команда или сервер на котором запустилась команда.

Вы можете получить командный источник из командного контекста при вызове `getSource()` в экземпляре `CommandContext`.

```java
Command<CommandSourceStack> command = context -> {
    CommandSourceStack source = context.getSource();
    return 0;
};
```

## Регистрация основной команды {#registering-a-basic-command}

Команды регистрируются в `CommandRegistrationCallback`, предоставляемый Fabric API.

::: info

Сведения о регистрации обратных вызовов смотрите в [События](../events).

:::

Событие должно быть зарегистрировано в вашем [инициализаторе мода](../getting-started/project-structure#entrypoints).

Обратный вызов имеет три аргумента:

- `CommandDispatcher<CommandSourceStack> dispatcher` - Used to register, parse and execute commands. `S` - это тип источника команд, который поддерживает диспатчер команд.
- `CommandBuildContext registryAccess` - Provides an abstraction to registries that may be passed to certain command
  argument methods
- `Commands.CommandSelection environment` - Identifies the type of server the commands are being registered
  on.

В инициализаторе мода мы просто регистрируем простую команду:

@[code lang=java transcludeWith=:::test_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

In the `sendSuccess()` method, the first parameter is the text to be sent, which is a `Supplier<Component>` to avoid
instantiating `Component` objects when not needed.

The second parameter determines whether to broadcast the feedback to other
moderators. Обычно, если команда предназначена для запроса чего-либо без затрагивания мира, например запросить текущее время или счёт игрока, оно должно быть `false`. Если команда делает что-либо, например изменение времени или изменение чего-либо счёт, оно должно быть `true`.

If the command fails, instead of calling `sendSuccess()`, you may directly throw any exception and the server or client
will handle it appropriately.

`CommandSyntaxException` обычно выбрасывается для обозначения синтаксических ошибок в команде или в аргументах. Вы можете так же имплементировать своё исключение.

Чтобы выполнить эту команду, необходимо ввести `/test_command`, при этом регистр символов имеет значение.

::: info

From this point onwards, we will be extracting the logic written within the lambda passed into `.executes()` builders into individual methods. We can then pass a method reference to `.executes()`. Это сделано для ясности.

:::

### Среда регистрации {#registration-environment}

При желании вы так же можете сделать так, чтобы когда команда регистрировалась с определёнными условиями, например только в выделенной среде:

@[code lang=java highlight={2} transcludeWith=:::dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Требования к команде {#command-requirements}

Let's say you have a command that you only want moderators to be able to execute. Здесь метод `requires()` вступает в игру. The `requires()` method has one argument of a `Predicate<S>` which will supply a `CommandSourceStack`
to test with and determine if the `CommandSource` can execute the command.

@[code lang=java highlight={3} transcludeWith=:::required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

This command will only execute if the source of the command is a moderator at a minimum, including command
blocks. Иначе, команда не зарегистрируется.

This has the side effect of not showing this command in <kbd>Tab</kbd> completion to anyone who is not a moderator. This is
also why you cannot <kbd>Tab</kbd>-complete most commands when you do not enable cheats.

### Подкоманды {#sub-commands}

Чтобы добавить подкоманду, вы должны зарегистрировать первый литеральный нод команды. Чтобы иметь подкоманду, вы должны добавить следующий литеральный нод к существующему ноду.

@[code lang=java highlight={3} transcludeWith=:::sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Подобно аргументам, ноды подкоманд могут также быть опциональными. В следующем случае будут допустимы как `/command_two`, так и `/command_two sub_command_two`.

@[code lang=java highlight={2,8} transcludeWith=:::sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Клиентские команды {#client-commands}

Fabric API имеет `ClientCommandManager` в пакете`net.fabricmc.fabric.api.client.command.v2` который можно использовать для регистрации команд на клиентской стороне. Код должен существовать только в коде клиентской стороны.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/client/command/ExampleModClientCommands.java)

## Перенаправление команд {#command-redirects}

Перенаправление команд - также известное как псевдонимы - это способ перенаправить функционал одной команды к другой. Это полезно, если вы хотите изменить название команды, но всё равно хотите поддерживать старое название.

::: warning

Brigadier [будет перенаправлять только командные узлы с аргументами](https://github.com/Mojang/brigadier/issues/46). Если вы хотите перенаправить командный узел без аргументов, предоставьте конструктор `.executes()` со ссылкой на ту же логику, что описана в примере.

:::

@[code lang=java transcludeWith=:::redirect_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_redirected_by](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## ЧАВО {#faq}

### Почему мой код не компилируется? {#why-does-my-code-not-compile}

- Словите или выбросите `CommandSyntaxException` - `CommandSyntaxException`- это не `RuntimeException`. Если вы выбросите это, то оно должно быть в методах, которые выбрасывают `CommandSyntaxException` в сигнатурном методе, или его нужно будет поймать.
  Brigadier обработает проверенные исключение и отправит вам сообщение об ошибке в игре.

- Время от времени у вас могут возникать проблемы с джинериками. If you are registering server
  commands (which are most of the case), make sure you are using `Commands.literal`
  or `Commands.argument` instead of `LiteralArgumentBuilder.literal` or `RequiredArgumentBuilder.argument`.

- Check the `sendSuccess()` method - You may have forgotten to provide a boolean as the second argument. Also remember
  that, since Minecraft 1.20, the first parameter is `Supplier<Component>` instead of `Component`.

- Команда должна возвращать целое число - При регистрации команд, метод `executes()` принимает объект `Command`, обычно это лямбда. Лямбда должна возвращать только целое число.

### Могу я зарегистрировать команды во время выполнения? {#can-i-register-commands-at-runtime}

::: warning

You can do this, but it is not recommended. You would get the `Commands` from the server and add anything commands
you wish to its `CommandDispatcher`.

After that, you need to send the command tree to every player again
using `Commands.sendCommands(ServerPlayer)`.

This is required because the client locally caches the command tree it receives during login (or when moderator packets
are sent) for local completions-rich error messages.

:::

### Могу я отменить регистрацию команд во время выполнения? {#can-i-unregister-commands-at-runtime}

::: warning

You can also do this, however, it is much less stable than registering commands at runtime and could cause unwanted side
effects.

Для простоты, вы должны использовать отражение Brigadier и удалить ноды. After this, you need to send the
command tree to every player again using `sendCommands(ServerPlayer)`.

Если вы не будете отправлять обновления команде tree, клиент может всё ещё подумать, что команда существует, хотя сервер провалит его выполнение.

:::

<!---->
