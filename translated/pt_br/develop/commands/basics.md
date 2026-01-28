---
title: Criando Comandos
description: Crie comandos com argumentos e ações complexas.
authors:
  - Atakku
  - dicedpixels
  - haykam821
  - i509VCB
  - Juuxel
  - MildestToucan
  - modmuss50
  - mschae23
  - natanfudge
  - Pyrofab
  - SolidBlock-cn
  - Technici4n
  - Treeways
  - xpple
---

Criar comandos pode permitir que o desenvolvedor de mods adicione funcionalidade que pode ser utilizada através de um comando. Este tutorial lhe ensinará como registrar comandos e a estrutural geral de comandos do Brigadier.

::: info

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based command library where
you build a tree of commands and arguments.

Brigadier is open-source: <https://github.com/Mojang/brigadier>

:::

## A interface `Command` {#the-command-interface}

`com.mojang.brigadier.Command` é uma interface funcional que executa algum código específico e gera uma `CommandSyntaxtException` em certos casos. Ela tem um tipo genérico `S`, que define o tipo de _fonte de comando_.
A fonte de comando fornece algum contexto sobre qual um comando foi executado. In Minecraft, the command source is typically a
`CommandSourceStack` which can represent a server, a command block, a remote connection (RCON), a player or an entity.

O único método em `Command`, `run(CommandContext<S>)` recebe um `CommandContext<S>` como único parâmetro e retorna um inteiro. O contexto do comando contém a fonte do comando `S` e lhe permite obter argumentos, examinar os nós de comando analisados e ver a entrada usada neste comando.

Como outras interfaces funcionais, ela é geralmente usada como um lambda ou uma referência de método:

```java
Command<CommandSourceStack> command = context -> {
    return 0;
};
```

O inteiro pode ser considerado o resultado do comando. Normalmente valores menores ou iguais a zero significam que um comando falhou e não fará nada. Valores positivos significam que o comando foi bem-sucedido e fez algo. O Brigadier fornece uma constante que indica sucesso; `Command#SINGLE_SUCESS`.

### What Can the `CommandSourceStack` Do? {#what-can-the-servercommandsource-do}

A `CommandSourceStack` provides some additional implementation-specific context when a command is run. Isso inclui a capacidade de obter a entidade que executou o comando, o mundo onde o comando foi executado, ou o servidor onde o comando foi executado.

Você pode acessar a fonte do comando a partir de um contexto de comando chamado `getSource()` na instância de `CommandContext`.

```java
Command<CommandSourceStack> command = context -> {
    CommandSourceStack source = context.getSource();
    return 0;
};
```

## Registrando um Comando Básico {#registering-a-basic-command}

Comandos são registrados dentro do `CommandRegistrationCallback` fornecido pela Fabric API.

::: info

Para informações sobre como registrar callbacks, por favor, consulte o guia de [Eventos](../events).

:::

The event should be registered in your [mod's initializer](../getting-started/project-structure#entrypoints).

O callback possui três parâmetros:

- `CommandDispatcher<CommandSourceStack> dispatcher` - Used to register, parse and execute commands. `S`é o tipo de fonte de comando que o command dispatcher suporta.
- `CommandBuildContext registryAccess` - Provides an abstraction to registries that may be passed to certain command
  argument methods
- `Commands.CommandSelection environment` - Identifies the type of server the commands are being registered
  on.

No inicializador do mod, nós apenas registramos um comando simples:

@[code lang=java transcludeWith=:::test_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

In the `sendSuccess()` method, the first parameter is the text to be sent, which is a `Supplier<Component>` to avoid
instantiating `Component` objects when not needed.

The second parameter determines whether to broadcast the feedback to other
moderators. Geralmente, se o comando for para consultar algo sem realmente afetar o mundo, como verificar a hora atual ou a pontuação de algum jogador, ele deve ser `false`. Se o comando fizer algo, como mudar a hora, ou modificar a pontuação de um jogador, ele deve ser `true`.

If the command fails, instead of calling `sendSuccess()`, you may directly throw any exception and the server or client
will handle it appropriately.

`CommandSyntaxException` é geralmente lançada para indicar erros de sintaxe em comandos ou argumentos. Você também pode implementar sua própria exceção.

Para executar este comando, você deve digitar `/test_command`, que diferencia maiúsculas e minúsculas.

::: info

From this point onwards, we will be extracting the logic written within the lambda passed into `.executes()` builders into individual methods. We can then pass a method reference to `.executes()`. Isso é feito para maior clareza.

:::

### Ambiente de Registro {#registration-environment}

Se desejar, você também pode garantir que um comando seja registrado apenas sob circunstâncias específicas, por exemplo, apenas no ambiente dedicado:

@[code lang=java highlight={2} transcludeWith=:::dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Requisitos de Comando {#command-requirements}

Let's say you have a command that you only want moderators to be able to execute. É aqui que o método `requires()` entra em jogo. The `requires()` method has one argument of a `Predicate<S>` which will supply a `CommandSourceStack`
to test with and determine if the `CommandSource` can execute the command.

@[code lang=java highlight={3} transcludeWith=:::required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

This command will only execute if the source of the command is a moderator at a minimum, including command
blocks. Caso contrário, o comando não é registrado.

This has the side effect of not showing this command in <kbd>Tab</kbd> completion to anyone who is not a moderator. This is
also why you cannot <kbd>Tab</kbd>-complete most commands when you do not enable cheats.

### Subcomandos {#sub-commands}

Para adicionar um subcomando, você registra o primeiro nó literal do comando normalmente. Para ter um subcomando, você deve anexar o próximo nó literal ao nó existente.

@[code lang=java highlight={3} transcludeWith=:::sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Similar aos argumentos, os nós de subcomando também podem ser definidos como opcionais. No caso a seguir, tanto `/command_two` quanto `/command_two sub_command_two` serão validos.

@[code lang=java highlight={2,8} transcludeWith=:::sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Comandos do Cliente {#client-commands}

A Fabric API possuim um `ClientCommandManager` no pacote `net.fabricmc.fabric.api.client.command.v2` que pode ser usado para registrar comandos no lado do cliente. O código deve existir apenas no código do lado do cliente.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/client/command/ExampleModClientCommands.java)

## Redirecionamentos de Comando {#command-redirects}

Redirecionamentos de Comando — também conhecidos como aliases, ou apelidos — são uma forma de redirecionar a funcionalidade de um comando para outro. Isso é útil quando você deseja alterar o nome de um comando, mas ainda quer manter o suporte ao nome antigo.

::: warning

O Brigadier [só irá redirecionar nós de comando que possuam argumentos](https://github.com/Mojang/brigadier/issues/46). Se você quiser redirecionar um nó de comando sem argumentos, forneça um builder `.executes()` com uma referência à mesma lógica, conforme descrito no exemplo.

:::

@[code lang=java transcludeWith=:::redirect_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_redirected_by](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Perguntas Frequentes {#faq}

### Por Que Meu Código Não Compila? {#why-does-my-code-not-compile}

- Capture ou lance uma `CommandSyntaxException` — `CommandSyntaxException` não é uma `RuntimeException`. Se você a lança, isso deve ser feito em métodos que declaram a `CommandSyntaxException` em suas assinaturas, ou ela deve ser capturada.
  O Brigadier vai lidar com as exceções checadas e encaminhará a mensagem de erro apropriada no jogo para você.

- Problemas com generics — Você pode ter um problema com generics ocasionalmente. If you are registering server
  commands (which are most of the case), make sure you are using `Commands.literal`
  or `Commands.argument` instead of `LiteralArgumentBuilder.literal` or `RequiredArgumentBuilder.argument`.

- Check the `sendSuccess()` method - You may have forgotten to provide a boolean as the second argument. Also remember
  that, since Minecraft 1.20, the first parameter is `Supplier<Component>` instead of `Component`.

- Um Comando deve retornar um inteiro — Ao registrar comandos, o método `executes()` aceita um objeto `Command`, que geralmente é um lambda. O lambda deve retornar um valor inteiro, em vez de outros tipos.

### Posso Registar Comandos em Tempo de Execução? {#can-i-register-commands-at-runtime}

::: warning

You can do this, but it is not recommended. You would get the `Commands` from the server and add anything commands
you wish to its `CommandDispatcher`.

After that, you need to send the command tree to every player again
using `Commands.sendCommands(ServerPlayer)`.

This is required because the client locally caches the command tree it receives during login (or when moderator packets
are sent) for local completions-rich error messages.

:::

### Posso Desregistrar Comandos em Tempo de Execução? {#can-i-unregister-commands-at-runtime}

::: warning

You can also do this, however, it is much less stable than registering commands at runtime and could cause unwanted side
effects.

Para manter as coisas simples, você precisa usar reflection no Brigadier e remover nós. After this, you need to send the
command tree to every player again using `sendCommands(ServerPlayer)`.

Se você não enviar a árvore de comandos atualizada, o cliente pode pensar que um comando ainda existe, mesmo que o servidor falhe na execução.

:::

<!---->
