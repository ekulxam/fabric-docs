---
title: Cadangan Perintah
description: Ketahui cara mencadangkan nilai argumen perintah kepada pengguna.
authors:
  - IMB11
---

Minecraft mempunyai sistem cadangan perintah yang berkuasa yang digunakan di banyak tempat, seperti perintah `/give`. Sistem ini membolehkan anda mencadangkan nilai untuk argumen perintah kepada pengguna, yang kemudiannya boleh mereka pilih - ini cara terbaik untuk menjadikan perintah anda lebih mesra pengguna dan ergonomik.

## Suggestion Providers {#suggestion-providers}

`SuggestionProvider` digunakan untuk membuat senarai cadangan yang akan dihantar kepada klien. Pembekal cadangan ialah antara muka berfungsi yang mengambil `CommandContext` dan `SuggestionBuilder` dan mengembalikan beberapa `Suggestions`. `SuggestionProvider` mengembalikan `CompletableFuture` kerana cadangan mungkin tidak tersedia dengan serta-merta.

## Using Suggestion Providers {#using-suggestion-providers}

To use a suggestion provider, you need to call the `suggests` method on the argument builder. This method takes a `SuggestionProvider` and returns the modified argument builder with the suggestion provider attached.

@[code java highlight={4} transcludeWith=:::command_with_suggestions](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code java transcludeWith=:::execute_command_with_suggestions](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Built-in Suggestion Providers {#built-in-suggestion-providers}

There are a few built-in suggestion providers that you can use:

| Pembekal Cadangan                         | Keterangan                                                     |
| ----------------------------------------- | -------------------------------------------------------------- |
| `SuggestionProviders.SUMMONABLE_ENTITIES` | Mencadangkan semua entiti yang boleh diseru.   |
| `SuggestionProviders.AVAILABLE_SOUNDS`    | Mencadangkan semua suara yang boleh dimainkan. |
| `LootCommand.SUGGESTION_PROVIDER`         | Suggests all loot tables that are available.   |
| `SuggestionProviders.ALL_BIOMES`          | Mencadangkan semua biom yang tersedia.         |

## Creating a Custom Suggestion Provider {#creating-a-custom-suggestion-provider}

If a built-in provider doesn't satisfy your needs, you can create your own suggestion provider. To do this, you need to create a class that implements the `SuggestionProvider` interface and override the `getSuggestions` method.

For this example, we'll make a suggestion provider that suggests all the player usernames on the server.

@[code java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/PlayerSuggestionProvider.java)

To use this suggestion provider, you would simply pass an instance of it into the `.suggests` method on the argument builder.

@[code java highlight={4} transcludeWith=:::command_with_custom_suggestions](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code java transcludeWith=:::execute_command_with_custom_suggestions](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Obviously, suggestion providers can be more complex, since they can also read the command context to provide suggestions based on the command's state - such as the arguments that have already been provided.

This could be in the form of reading the player's inventory and suggesting items, or entities that are nearby the player.
