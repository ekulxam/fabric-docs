---
title: Paramètres de Commandes
description: Apprenez comment créer des commandes avec des paramètres complexes.
---

La notion de paramètres (`Argument`) est utilisée dans la plupart des commandes. Des fois, ces paramètres peuvent être optionnels, ce qui veut dire que si vous ne donnez pas ce paramètre, la commande va quand même s'exécuter. Un nœud peut avoir plusieurs types de paramètres, mais n'oubliez pas qu'il y a une possibilité d'ambiguïté, qui devrait toujours être évitée.

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Dans ce cas d'exemple, après la commande textuelle `/argtater`, vous devez donner un nombre. Par exemple, si vous exécutez `/argtater 3`, vous allez avoir en retour le message `Called /argtater with value = 3`.

> Called /command_with_arg with value = 3

Si vous tapez `/argater` sans arguments, la commande ne pourra pas être correctement analysée.

Nous ajoutons ensuite un second paramètre optionnel :

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Maintenant, vous pouvez donner un ou deux arguments de type nombre à la commande. Si vous donnez un seul nombre, vous allez avoir en retour un texte avec une seule valeur d'affichée. Si vous donnez deux nombres, vous allez avoir en retour un texte avec deux valeurs d'affichées.

Vous pouvez penser qu'il n'est pas nécessaire de spécifier plusieurs fois des exécutions similaires. Donc, nous allons créer une méthode qui va être utilisée pour les deux cas d'exécution.

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Types de paramètre personnalisé {#custom-argument-types}

Si le type de paramètre de commande dont vous avez besoin n'existe pas en vanilla, vous pouvez toujours le créer par vous-même. Pour le faire, vous avez besoin de créer une nouvelle classe qui hérite l'interface `ArgumentType<T>` où `T` est le type du paramètre.

Vous devrez implémenter la méthode `parse`, qui va donc analyser la saisie de chaine de caractères afin de la transformer en le type désiré.

Par exemple, vous pouvez créer un type de paramètre qui transforme une chaine de caractères en `BlockPos` avec le format suivant : `{x, y, z}`

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### Enregistrer des types de paramètres personnalisés {#registering-custom-argument-types}

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
