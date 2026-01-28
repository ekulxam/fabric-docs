---
title: Creare Comandi
description: Creare comandi con parametri e azioni complesse.
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

Creare comandi può permettere a uno sviluppatore di mod di aggiungere funzionalità che possono essere usate attraverso un comando. Questo tutorial ti insegnerà come registrare comandi e qual è la struttura generale dei comandi di Brigadier.

::: info

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based command library where
you build a tree of commands and arguments.

Brigadier is open-source: <https://github.com/Mojang/brigadier>

:::

## L'interface `Command` {#the-command-interface}

`com.mojang.brigadier.Command` è un'interfaccia funzionale, che esegue del codice specifico, e lancia una `CommandSyntaxException` in determinati casi. Ha un tipo generico `S`, che definisce il tipo della _sorgente del comando_.
La sorgente del comando fornisce del contesto in cui un comando è stato eseguito. In Minecraft, the command source is typically a
`CommandSourceStack` which can represent a server, a command block, a remote connection (RCON), a player or an entity.

L'unico metodo di `Command`, `run(CommandContext<S>)` accetta un `CommandContext<S>` come unico parametro e restituisce un intero. Il contesto del comando contiene la tua sorgente del comando di `S` e ti permette di ottenere parametri, osservare i nodi di un comando di cui è stato effettuato il parsing e vedere l'input usato in questo comando.

Come altre interfacce funzionali, viene solitamente usata come una lambda o come un riferimento a un metodo:

```java
Command<CommandSourceStack> command = context -> {
    return 0;
};
```

L'intero può essere considerato il risultato del comando. Di solito valori minori o uguali a zero indicano che un comando è fallito e non farà nulla. Valori positivi indicano che il comando ha avuto successo e ha fatto qualcosa. Brigadier fornisce una costante per indicare
il successo; `Command#SINGLE_SUCCESS`.

### What Can the `CommandSourceStack` Do? {#what-can-the-servercommandsource-do}

A `CommandSourceStack` provides some additional implementation-specific context when a command is run. Questo include la possibilità di ottenere l'entità che ha eseguito il comando, il mondo in cui esso è stato eseguito o il server su cui è stato eseguito.

Puoi accedere alla sorgente del comando dal contesto del comando chiamando `getSource()` sull'istanza di `CommandContext`.

```java
Command<CommandSourceStack> command = context -> {
    CommandSourceStack source = context.getSource();
    return 0;
};
```

## Registrare un Comando Basilare {#registering-a-basic-command}

I comandi sono registrati all'interno del `CommandRegistrationCallback` fornito dall'API di Fabric.

::: info

Per informazioni su come registrare i callback, vedi per favore la guida [Eventi](../events).

:::

The event should be registered in your [mod's initializer](../getting-started/project-structure#entrypoints).

Il callback ha tre parametri:

- `CommandDispatcher<CommandSourceStack> dispatcher` - Used to register, parse and execute commands. `S` è il tipo di fonte di comando che il dispatcher supporta.
- `CommandBuildContext registryAccess` - Provides an abstraction to registries that may be passed to certain command
  argument methods
- `Commands.CommandSelection environment` - Identifies the type of server the commands are being registered
  on.

Nell'initializer della mod, registriamo un semplice comando:

@[code lang=java transcludeWith=:::test_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

In the `sendSuccess()` method, the first parameter is the text to be sent, which is a `Supplier<Component>` to avoid
instantiating `Component` objects when not needed.

The second parameter determines whether to broadcast the feedback to other
moderators. In generale, se il comando deve ottenere informazioni senza effettivamente modificare il mondo, come il tempo corrente o una statistica di un giocatore, dovrebbe essere `false`. Se il comando fa qualcosa, come cambiare il tempo o modificare il punteggio di qualcuno, dovrebbe essere `true`.

If the command fails, instead of calling `sendSuccess()`, you may directly throw any exception and the server or client
will handle it appropriately.

`CommandSyntaxException` generalmente viene lanciata per indicare errori di sintassi nel comando o negli argomenti. Puoi anche implementare la tua eccezione personalizzata.

Per eseguire questo comando, devi scrivere `/test_command`, tutto minuscolo.

::: info

From this point onwards, we will be extracting the logic written within the lambda passed into `.executes()` builders into individual methods. We can then pass a method reference to `.executes()`. Faremo questo per chiarezza.

:::

### Ambiente di Registrazione {#registration-environment}

Se vuoi, puoi anche assicurarti che un comando venga registrato solo sotto circostanze specifiche, per esempio, solo nell'ambiente dedicato:

@[code lang=java highlight={2} transcludeWith=:::dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_dedicated_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Requisiti dei Comandi {#command-requirements}

Let's say you have a command that you only want moderators to be able to execute. Questo è dove il metodo `requires()` entra in gioco. The `requires()` method has one argument of a `Predicate<S>` which will supply a `CommandSourceStack`
to test with and determine if the `CommandSource` can execute the command.

@[code lang=java highlight={3} transcludeWith=:::required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_required_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

This command will only execute if the source of the command is a moderator at a minimum, including command
blocks. Altrimenti, il comando non è registrato.

This has the side effect of not showing this command in <kbd>Tab</kbd> completion to anyone who is not a moderator. This is
also why you cannot <kbd>Tab</kbd>-complete most commands when you do not enable cheats.

### Sotto Comandi {#sub-commands}

Per aggiungere un sotto comando, devi registrare il primo nodo letterale del comando normalmente. Per avere un sotto comando, devi aggiungere il nodo letterale successivo al nodo esistente.

@[code lang=java highlight={3} transcludeWith=:::sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_sub_command_one](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Similarmente agli argomenti, i nodi dei sotto comandi possono anch'essi essere opzionali. Nel caso seguente, sia `/command_two` che `/command_two sub_command_two` saranno validi.

@[code lang=java highlight={2,8} transcludeWith=:::sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_sub_command_two](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Comandi Lato Client {#client-commands}

L'API di Fabric ha un `ClientCommandManager` nel package `net.fabricmc.fabric.api.client.command.v2` che può essere usato per registrare comandi lato client. Il codice dovrebbe esistere solo nel codice lato client.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/client/command/ExampleModClientCommands.java)

## Reindirizzare Comandi {#command-redirects}

I comandi reindirizzati - anche noti come alias - sono un modo di reindirizzare la funzionalità di un comando a un altro. Questo è utile quando vuoi cambiare il nome di un comando, ma vuoi comunque supportare il vecchio nome.

::: warning

Brigadier [reinderizzerà soltanto i nodi di comandi contenenti parametri](https://github.com/Mojang/brigadier/issues/46). Se volessi reinderizzare il nodo di un comando senza parametri, fornisci un costruttore `.executes()` con un riferimento alla stessa logica presentata nell'esempio.

:::

@[code lang=java transcludeWith=:::redirect_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_redirected_by](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Domande Frequenti (FAQ) {#faq}

### Perché il Mio Codice Non Viene Compilato? {#why-does-my-code-not-compile}

- Catturare o lanciare una `CommandSyntaxException` - `CommandSyntaxException` non è una `RuntimeException`. Se la lanci, dovresti farlo in metodi che lanciano una `CommandSyntaxException` nelle firme dei metodi, oppure dovresti catturarla.
  Brigadier gestirà le eccezioni controllate e ti inoltrerà il messaggio d'errore effettivo nel gioco.

- Problemi con i generic - Potresti avere un problema con i generic una volta ogni tanto. If you are registering server
  commands (which are most of the case), make sure you are using `Commands.literal`
  or `Commands.argument` instead of `LiteralArgumentBuilder.literal` or `RequiredArgumentBuilder.argument`.

- Check the `sendSuccess()` method - You may have forgotten to provide a boolean as the second argument. Also remember
  that, since Minecraft 1.20, the first parameter is `Supplier<Component>` instead of `Component`.

- Un Command dovrebbe restituire un intero - Quando registri comandi, il metodo `executes()` accetta un oggetto `Command`, che è solitamente una lambda. La lambda dovrebbe restituire un intero, anziché altri tipi.

### Posso Registrare Comandi al Runtime? {#can-i-register-commands-at-runtime}

::: warning

You can do this, but it is not recommended. You would get the `Commands` from the server and add anything commands
you wish to its `CommandDispatcher`.

After that, you need to send the command tree to every player again
using `Commands.sendCommands(ServerPlayer)`.

This is required because the client locally caches the command tree it receives during login (or when moderator packets
are sent) for local completions-rich error messages.

:::

### Posso De-Registrare Comandi al Runtime? {#can-i-unregister-commands-at-runtime}

::: warning

You can also do this, however, it is much less stable than registering commands at runtime and could cause unwanted side
effects.

Per tenere le cose semplici, devi usare la reflection su Brigadier e rimuovere nodi. After this, you need to send the
command tree to every player again using `sendCommands(ServerPlayer)`.

Se non mandi l'albero di comandi aggiornato, il client potrebbe credere che il comando esista ancora, anche se fallirà l'esecuzione sul server.

:::

<!---->
