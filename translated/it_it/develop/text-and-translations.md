---
title: Testo e Traduzioni
description: Documentazione esaustiva riguardo alla gestione di testo e traduzioni formattate in Minecraft.
authors:
  - IMB11
  - LordEnder-Kitty
---

<!-- markdownlint-configure-file { MD033: { allowed_elements: [br, ColorSwatch, u] } } -->

Whenever Minecraft displays text in-game, it's probably defined using a `Component` object.
Questo tipo personalizzato è preferito ad una `String` per permettere formattazione più avanzata, che comprende colori, grassetto, offuscamento, ed eventi ai clic. Permettono inoltre accesso più semplice al sistema di traduzione, rendendo semplice la traduzione di qualsiasi elemento dell'interfaccia.

Se hai mai lavorato con datapack o con funzioni prima d'ora, potresti notare similarità con il formato testo json usato per i displayName, i libri, e i cartelli tra le altre cose. As you
can probably guess, this is just a json representation of a `Component` object, and can be
converted to and from using `Component.Serializer`.

When making a mod, it is generally preferred to construct your `Component` objects directly
in code, making use of translations whenever possible.

## Literal Text Components {#literal-text-components}

The simplest way to create a `Component` object is to make a literal. Questa è proprio solo una stringa che verrà visualizzata com'è, senza alcuna formattazione predefinita.

These are created using the `Component.nullToEmpty` or `Component.literal` methods, which both act slightly
differently. `Component.nullToEmpty` accepts nulls as input, and will return a `Component` instance. In
contrast, `Component.literal` should not be given a null input, but returns a `MutableComponent`,
this being a subclass of `Component` that can be easily styled and concatenated. Ne parleremo di più dopo.

```java
Component literal = Component.nullToEmpty("Hello, world!");
MutableComponent mutable = Component.literal("Hello, world!");
// Keep in mind that a MutableComponent can be used as a Component, making this valid:
Component mutableAsText = mutable;
```

## Translatable Text Components {#translatable-text-components}

When you want to provide multiple translations for the same string of text, you can use the `Component.translatable` method to reference a translation key in any language file. Se la chiave di traduzione non esistesse, verrebbe convertita in testo letterale.

```java
Component translatable = Component.translatable("example-mod.text.hello");

// Similarly to literals, translatable text can be easily made mutable.
MutableComponent mutable = Component.translatable("example-mod.text.bye");
```

Il file di lingua, `en_us.json`, ha il seguente aspetto:

```json
{
  "example-mod.text.hello": "Hello!",
  "example-mod.text.bye": "Goodbye :("
}
```

Se volessi usare variabili nella traduzione, in maniera simile a come i messaggi di morte ti permettodo di usare i giocatori e gli oggetti coinvolti nella traduzione, puoi aggiungere queste variabili come parametri. Puoi aggiungerne quanti ne vuoi.

```java
Component translatable = Component.translatable("example-mod.text.hello", player.getDisplayName());
```

Puoi fare riferimento a queste variabili nella traduzione così:

```json
{
  "example-mod.text.hello": "%1$s said hello!"
}
```

In the game, `%1$s` will be replaced with the name of the player you referenced in the code. Usare `player.getDisplayName()` farà in modo che certe informazioni aggiuntive sull'entità appaiano in un tooltip mentre si passa il mouse sopra al nome nel messaggio in chat, e ciò in contrasto con `player.getName()`, che mostrerà comunque il nome, ma non i dettagli aggiuntivi. Similar can be done with itemStacks, using `stack.getDisplayName()`.

As for what `%1$s` even means, all you really need to know is that the number corresponds to which variable you are trying to use. Immagina di usare tre variabili.

```java
Component translatable = Component.translatable("example-mod.text.whack.item", victim.getDisplayName(), attacker.getDisplayName(), itemStack.toHoverableText());
```

If you want to reference what, in our case, is the attacker, you would use `%2$s` because it's the second variable that we passed in. Likewise, `%3$s` refers to the itemStack. Una traduzione con questo numero di parametri aggiuntivi potrebbe avere questo aspetto:

```json
{
  "example-mod.text.whack.item": "%1$s was whacked by %2$s using %3$s"
}
```

## Serializzare Testo {#serializing-text}

<!-- NOTE: These have been put into the example mod as they're likely to be updated to codecs in the next few updates. -->

Come già accennato prima, puoi serializzare testo a JSON con il codec di testo. Per maggiori informazioni, vedi la pagina sui [Codec](./codecs).

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

Questo produce JSON che può essere usato in datapack, comandi e altri posti che accettano il formato JSON di testo invece che il formato letterale o traducibile.

## Deserializzare Testo {#deserializing-text}

Furthermore, to deserialize a JSON text object into an actual `Component` class, again, use the codec.

@[code transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

## Formattare Testo {#formatting-text}

Forse sei familiare con gli standard di formattazione di Minecraft:

You can apply these formatting styles using the `ChatFormatting` enum on the `MutableComponent` class:

```java
MutableComponent result = Component.literal("Hello World!")
  .withStyle(ChatFormatting.AQUA, ChatFormatting.BOLD, ChatFormatting.UNDERLINE);
```

|              Colore             | Nome                             | Codice in Chat | Codice MOTD | Codice Hex |
| :-----------------------------: | -------------------------------- | :------------: | :---------: | :--------: |
| <ColorSwatch color="#000000" /> | Nero<br />`black`                |      `§0`      |  `\u00A70` |  `#000000` |
| <ColorSwatch color="#0000AA" /> | Blu Scuro<br />`dark_blue`       |      `§1`      |  `\u00A71` |  `#0000AA` |
| <ColorSwatch color="#00AA00" /> | Verde Scuro<br />`dark_green`    |      `§2`      |  `\u00A72` |  `#00AA00` |
| <ColorSwatch color="#00AAAA" /> | Ciano Scuro<br />`dark_aqua`     |      `§3`      |  `\u00A73` |  `#00AAAA` |
| <ColorSwatch color="#AA0000" /> | Rosso Scuro<br />`dark_red`      |      `§4`      |  `\u00A74` |  `#AA0000` |
| <ColorSwatch color="#AA00AA" /> | Viola Scuro<br />`dark_purple`   |      `§5`      |  `\u00A75` |  `#AA00AA` |
| <ColorSwatch color="#FFAA00" /> | Oro<br />`gold`                  |      `§6`      |  `\u00A76` |  `#FFAA00` |
| <ColorSwatch color="#AAAAAA" /> | Grigio<br />`gray`               |      `§7`      |  `\u00A77` |  `#AAAAAA` |
| <ColorSwatch color="#555555" /> | Grigio Scuro<br />`dark_gray`    |      `§8`      |  `\u00A78` |  `#555555` |
| <ColorSwatch color="#5555FF" /> | Blu<br />`blue`                  |      `§9`      |  `\u00A79` |  `#5555FF` |
| <ColorSwatch color="#55FF55" /> | Verde<br />`green`               |      `§a`      |  `\u00A7a` |  `#55FF55` |
| <ColorSwatch color="#55FFFF" /> | Ciano<br />`aqua`                |      `§b`      |  `\u00A7b` |  `#55FFFF` |
| <ColorSwatch color="#FF5555" /> | Rosso<br />`red`                 |      `§c`      |  `\u00A7c` |  `#FF5555` |
| <ColorSwatch color="#FF55FF" /> | Viola Chiaro<br />`light_purple` |      `§d`      |  `\u00A7d` |  `#FF55FF` |
| <ColorSwatch color="#FFFF55" /> | Giallo<br />`yellow`             |      `§e`      |  `\u00A7e` |  `#FFFF55` |
| <ColorSwatch color="#FFFFFF" /> | Bianco<br />`white`              |      `§f`      |  `\u00A7f` |  `#FFFFFF` |
|                                 | Resetta                          |      `§r`      |             |            |
|                                 | **Grassetto**                    |      `§l`      |             |            |
|                                 | ~~Barrato~~                      |      `§m`      |             |            |
|                                 | <u>Sottolineato</u>              |      `§n`      |             |            |
|                                 | _Corsivo_                        |      `§o`      |             |            |
|                                 | Offuscato                        |      `§k`      |             |            |
