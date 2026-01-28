---
title: Текст и переводы
description: Подробная документация по работе Minecraft с форматированным текстом и переводами.
authors:
  - IMB11
  - LordEnder-Kitty
---

<!-- markdownlint-configure-file { MD033: { allowed_elements: [br, ColorSwatch, u] } } -->

Whenever Minecraft displays text in-game, it's probably defined using a `Component` object.
Этот пользовательский тип используется вместо `String` для обеспечения более сложного форматирования,
включая цвета, выделение жирным шрифтом, обфускации и событий нажатия (click). Они также обеспечивают легкий доступ
в систему перевода, что упрощает перевод любого элемента пользовательского интерфейса в
разные языки.

Если вы раньше работали с пакетами данных (датапаками) или функциями, вы можете увидеть параллели с
текстовым форматом json, используемым для отображения названий, книг, табличек и другого. As you
can probably guess, this is just a json representation of a `Component` object, and can be
converted to and from using `Component.Serializer`.

When making a mod, it is generally preferred to construct your `Component` objects directly
in code, making use of translations whenever possible.

## Literal Text Components {#literal-text-components}

The simplest way to create a `Component` object is to make a literal. По сути это строка которая будет отображена как есть, без форматирования по умолчанию.

These are created using the `Component.nullToEmpty` or `Component.literal` methods, which both act slightly
differently. `Component.nullToEmpty` accepts nulls as input, and will return a `Component` instance. In
contrast, `Component.literal` should not be given a null input, but returns a `MutableComponent`,
this being a subclass of `Component` that can be easily styled and concatenated. Подробнее об этом будет сказано позже.

```java
Component literal = Component.nullToEmpty("Hello, world!");
MutableComponent mutable = Component.literal("Hello, world!");
// Keep in mind that a MutableComponent can be used as a Component, making this valid:
Component mutableAsText = mutable;
```

## Translatable Text Components {#translatable-text-components}

When you want to provide multiple translations for the same string of text, you can use the `Component.translatable` method to reference a translation key in any language file. Если ключ не существует, ключ перевода конвертируется в литерал.

```java
Component translatable = Component.translatable("example-mod.text.hello");

// Similarly to literals, translatable text can be easily made mutable.
MutableComponent mutable = Component.translatable("example-mod.text.bye");
```

Файл языка, `en_us.json`, выглядит как то так:

```json
{
  "example-mod.text.hello": "Hello!",
  "example-mod.text.bye": "Goodbye :("
}
```

Если вы хотите иметь возможность использовать переменные в переводе, подобно тому, как сообщения о смерти позволяют вам использовать задействованных игроков и предметы в переводе, вы можете добавить указанные переменные в качестве параметров. Вы можете добавить сколько угодно параметров.

```java
Component translatable = Component.translatable("example-mod.text.hello", player.getDisplayName());
```

Вы можете ссылаться на эти переменные в переводе следующим образом:

```json
{
  "example-mod.text.hello": "%1$s said hello!"
}
```

In the game, `%1$s` will be replaced with the name of the player you referenced in the code. Использование `player.getDisplayName()` приведет к тому, что дополнительная информация об объекте появится во всплывающей подсказке при наведении курсора на имя в сообщении чата, в отличие от использования `player.getName()`, которое все равно получит имя; однако дополнительные детали не отображаются. Similar can be done with itemStacks, using `stack.getDisplayName()`.

As for what `%1$s` even means, all you really need to know is that the number corresponds to which variable you are trying to use. Допустим, у вас есть три переменные, которые вы используете.

```java
Component translatable = Component.translatable("example-mod.text.whack.item", victim.getDisplayName(), attacker.getDisplayName(), itemStack.toHoverableText());
```

If you want to reference what, in our case, is the attacker, you would use `%2$s` because it's the second variable that we passed in. Likewise, `%3$s` refers to the itemStack. Перевод с таким количеством дополнительных параметров может выглядеть так:

```json
{
  "example-mod.text.whack.item": "%1$s was whacked by %2$s using %3$s"
}
```

## Десериализация Текста {#deserializing-text}

<!-- NOTE: These have been put into the example mod as they're likely to be updated to codecs in the next few updates. -->

Как было упомянуто выше, вы можете сериализовать текст в JSON используя текстовый кодек. Для большей информации об кодеках, посмотрите страницу [Codec](./codecs).

@[code transcludeWith=:::1](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

В результате получается JSON, который можно использовать в датапаках, командах и других местах которые принимают JSON формат текста вместо буквального или переводимого текста.

## Сериализация Текста {#serializing-text}

Furthermore, to deserialize a JSON text object into an actual `Component` class, again, use the codec.

@[code transcludeWith=:::2](@/reference/latest/src/client/java/com/example/docs/rendering/TextTests.java)

## Форматирование Текста {#formatting-text}

Вы можете быть знакомы со стандартом форматирования в Minecraft:

You can apply these formatting styles using the `ChatFormatting` enum on the `MutableComponent` class:

```java
MutableComponent result = Component.literal("Hello World!")
  .withStyle(ChatFormatting.AQUA, ChatFormatting.BOLD, ChatFormatting.UNDERLINE);
```

|               Цвет              | Название                                                               | Код в чате | Код для MOTD | HEX-код |
| :-----------------------------: | ---------------------------------------------------------------------- | :--------: | :----------: | :-----: |
| <ColorSwatch color="#000000" /> | Чёрный (black)                                      |     §0     |    \u00A70   | #000000 |
| <ColorSwatch color="#0000AA" /> | Тёмно-синий (dark_blue)        |     §1     |    \u00A71   | #0000AA |
| <ColorSwatch color="#00AA00" /> | Тёмно-зелёный (dark_green)     |     §2     |    \u00A72   | #00AA00 |
| <ColorSwatch color="#00AAAA" /> | Тёмно-голубой (dark_aqua)      |     §3     |    \u00A73   | #00AAAA |
| <ColorSwatch color="#AA0000" /> | Тёмно-красный (dark_red)       |     §4     |    \u00A74   | #AA0000 |
| <ColorSwatch color="#AA00AA" /> | Тёмно-фиолетовый (dark_purple) |     §5     |    \u00A75   | #AA00AA |
| <ColorSwatch color="#FFAA00" /> | Золотой (gold)                                      |     §6     |    \u00A76   | #FFAA00 |
| <ColorSwatch color="#AAAAAA" /> | Серый (gray)                                        |     §7     |    \u00A77   | #AAAAAA |
| <ColorSwatch color="#555555" /> | Тёмно-серый (dark_gray)        |     §8     |    \u00A78   | #555555 |
| <ColorSwatch color="#5555FF" /> | Синий (blue)                                        |     §9     |    \u00A79   | #5555FF |
| <ColorSwatch color="#55FF55" /> | Зелёный (green)                                     |     §a     |    \u00A7a   | #55FF55 |
| <ColorSwatch color="#55FFFF" /> | Аква<br />`aqua`                                                       |     §b     |    \u00A7b   | #55FFFF |
| <ColorSwatch color="#FF5555" /> | Красный (red)                                       |     §c     |    \u00A7c   | #FF5555 |
| <ColorSwatch color="#FF55FF" /> | Фиолетовый (light_purple)      |     §d     |    \u00A7d   | #FF55FF |
| <ColorSwatch color="#FFFF55" /> | Жёлтый (yellow)                                     |     §e     |    \u00A7e   | #FFFF55 |
| <ColorSwatch color="#FFFFFF" /> | Белый (white)                                       |     §f     |    \u00A7f   | #FFFFFF |
|                                 | Сброс форматирования                                                   |     §r     |              |         |
|                                 | **Жирный**                                                             |     §l     |              |         |
|                                 | ~~Зачеркивание~~                                                       |     §m     |              |         |
|                                 | <u>Подчёркнутый</u>                                                    |     §n     |              |         |
|                                 | _курсив_                                                               |     §o     |              |         |
|                                 | Зашифрованный                                                          |     §k     |              |         |
