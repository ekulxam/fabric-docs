---
title: 文字和翻譯
description: 有關 Minecraft 處理格式化文字和翻譯的綜合文檔。
authors:
  - IMB11
  - LordEnder-Kitty
---

<!-- markdownlint-configure-file { MD033: { allowed_elements: [br, ColorSwatch, u] } } -->

Whenever Minecraft displays text in-game, it's probably defined using a `Component` object.
這個自定義類型作為取代 `String` 以允許更進階的格式，包括色彩、粗體、混淆和點擊事件。 他們也允許輕鬆翻譯系統，讓 UI 界面中的任意文字都能夠輕鬆本地化。

若你以前寫過資料包或函數，你可能會發現它類似於用作 顯示名稱、書本、告示牌即諸多事物的 json 格式化字串。 As you
can probably guess, this is just a json representation of a `Component` object, and can be
converted to and from using `Component.Serializer`.

When making a mod, it is generally preferred to construct your `Component` objects directly
in code, making use of translations whenever possible.

## Literal Text Components {#literal-text-components}

The simplest way to create a `Component` object is to make a literal. 這就只是個直接被顯示的字串，使用預設格式。

These are created using the `Component.nullToEmpty` or `Component.literal` methods, which both act slightly
differently. `Component.nullToEmpty` accepts nulls as input, and will return a `Component` instance. In
contrast, `Component.literal` should not be given a null input, but returns a `MutableComponent`,
this being a subclass of `Component` that can be easily styled and concatenated. 稍後會詳細介紹。

```java
Component literal = Component.nullToEmpty("Hello, world!");
MutableComponent mutable = Component.literal("Hello, world!");
// Keep in mind that a MutableComponent can be used as a Component, making this valid:
Component mutableAsText = mutable;
```

## Translatable Text Components {#translatable-text-components}

When you want to provide multiple translations for the same string of text, you can use the `Component.translatable` method to reference a translation key in any language file. 若本地化鍵不存在，則本地化鍵會轉換成字面文字。

```java
Component translatable = Component.translatable("example-mod.text.hello");

// Similarly to literals, translatable text can be easily made mutable.
MutableComponent mutable = Component.translatable("example-mod.text.bye");
```

語言檔案 `en_us.json` 看起來像如此：

```json
{
  "example-mod.text.hello": "Hello!",
  "example-mod.text.bye": "Goodbye :("
}
```

如果你希望在翻譯中使用變數，如同死亡訊息允許玩家何物品參與翻譯一般，您可以新增所述變數作為參數。 你可以新增任意多個變數。

```java
Component translatable = Component.translatable("example-mod.text.hello", player.getDisplayName());
```

你需要在翻譯中參照這些變數：

```json
{
  "example-mod.text.hello": "%1$s said hello!"
}
```

In the game, `%1$s` will be replaced with the name of the player you referenced in the code. Using `player.getDisplayName()` will make it so that additional information about the entity will appear in a tooltip when hovering over the name in the chat message as opposed to using `player.getName()`, which will still get the name; however, it will not show the extra details. Similar can be done with itemStacks, using `stack.getDisplayName()`.

As for what `%1$s` even means, all you really need to know is that the number corresponds to which variable you are trying to use. Let's say you have three variables that you are using.

```java
Component translatable = Component.translatable("example-mod.text.whack.item", victim.getDisplayName(), attacker.getDisplayName(), itemStack.toHoverableText());
```

If you want to reference what, in our case, is the attacker, you would use `%2$s` because it's the second variable that we passed in. Likewise, `%3$s` refers to the itemStack. A translation with this many additional parameters might look like this:

```json
{
  "example-mod.text.whack.item": "%1$s was whacked by %2$s using %3$s"
}
```

## Serializing Text {#serializing-text}

<!-- NOTE: These have been put into the example mod as they're likely to be updated to codecs in the next few updates. -->

As mentioned before, you can serialize text to JSON using the text codec. For more information on codecs, see the [Codec](./codecs) page.

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

This produces JSON that can be used datapacks, commands and other places that accept the JSON format of text instead of literal or translatable text.

## Deserializing Text {#deserializing-text}

Furthermore, to deserialize a JSON text object into an actual `Component` class, again, use the codec.

@[code transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

## Formatting Text {#formatting-text}

You may be familiar with Minecraft's formatting standards:

You can apply these formatting styles using the `ChatFormatting` enum on the `MutableComponent` class:

```java
MutableComponent result = Component.literal("Hello World!")
  .withStyle(ChatFormatting.AQUA, ChatFormatting.BOLD, ChatFormatting.UNDERLINE);
```

|              Color              | Name                             | Chat Code |  MOTD Code |  Hex Code |
| :-----------------------------: | -------------------------------- | :-------: | :--------: | :-------: |
| <ColorSwatch color="#000000" /> | Black<br />`black`               |    `§0`   | `\u00A70` | `#000000` |
| <ColorSwatch color="#0000AA" /> | Dark Blue<br />`dark_blue`       |    `§1`   | `\u00A71` | `#0000AA` |
| <ColorSwatch color="#00AA00" /> | Dark Green<br />`dark_green`     |    `§2`   | `\u00A72` | `#00AA00` |
| <ColorSwatch color="#00AAAA" /> | Dark Aqua<br />`dark_aqua`       |    `§3`   | `\u00A73` | `#00AAAA` |
| <ColorSwatch color="#AA0000" /> | Dark Red<br />`dark_red`         |    `§4`   | `\u00A74` | `#AA0000` |
| <ColorSwatch color="#AA00AA" /> | Dark Purple<br />`dark_purple`   |    `§5`   | `\u00A75` | `#AA00AA` |
| <ColorSwatch color="#FFAA00" /> | Gold<br />`gold`                 |    `§6`   | `\u00A76` | `#FFAA00` |
| <ColorSwatch color="#AAAAAA" /> | Gray<br />`gray`                 |    `§7`   | `\u00A77` | `#AAAAAA` |
| <ColorSwatch color="#555555" /> | Dark Gray<br />`dark_gray`       |    `§8`   | `\u00A78` | `#555555` |
| <ColorSwatch color="#5555FF" /> | Blue<br />`blue`                 |    `§9`   | `\u00A79` | `#5555FF` |
| <ColorSwatch color="#55FF55" /> | Green<br />`green`               |    `§a`   | `\u00A7a` | `#55FF55` |
| <ColorSwatch color="#55FFFF" /> | Aqua<br />`aqua`                 |    `§b`   | `\u00A7b` | `#55FFFF` |
| <ColorSwatch color="#FF5555" /> | Red<br />`red`                   |    `§c`   | `\u00A7c` | `#FF5555` |
| <ColorSwatch color="#FF55FF" /> | Light Purple<br />`light_purple` |    `§d`   | `\u00A7d` | `#FF55FF` |
| <ColorSwatch color="#FFFF55" /> | Yellow<br />`yellow`             |    `§e`   | `\u00A7e` | `#FFFF55` |
| <ColorSwatch color="#FFFFFF" /> | White<br />`white`               |    `§f`   | `\u00A7f` | `#FFFFFF` |
|                                 | Reset                            |    `§r`   |            |           |
|                                 | **Bold**                         |    `§l`   |            |           |
|                                 | ~~Strikethrough~~                |    `§m`   |            |           |
|                                 | <u>Underline</u>                 |    `§n`   |            |           |
|                                 | _Italic_                         |    `§o`   |            |           |
|                                 | Obfuscated                       |    `§k`   |            |           |
