---
title: Отчёты о сбоях
description: Узнайте, что можно делать с отчётами о сбоях и как их читать.
authors:
  - IMB11
---

<!---->

::: tip

Если у вас возникли какие-либо трудности с поиском причины сбоя, вы можете обратиться за помощью в [Fabric Discord](https://discord.fabricmc.net/) в каналах `#player-support` или `#server-admin-support`.

:::

Отчёты о сбоях являются важной частью устранения проблем с вашей игрой или сервером. Они содержат много информации об ошибках и могут помочь вам найти причину краша.

## Поиск отчётов о сбоях {#finding-crash-reports}

Отчёты о сбоях находятся в папке `crash-reports` в папке вашей игры. Если у вас сервер, то они хранятся в папке `crash-reports` в папке сервера.

Для неофициальных лаунчеров вам следует посмотреть в их документацию, чтобы найти отчёты о сбоях.

Отчёты о сбоях можно найти по этим путям:

::: code-group

```text:no-line-numbers [Windows]
%appdata%\.minecraft\crash-reports
```

```text:no-line-numbers [macOS]
~/Library/Application Support/minecraft/crash-reports
```

```text:no-line-numbers [Linux]
~/.minecraft/crash-reports
```

:::

## Чтение отчётов о сбоях {#reading-crash-reports}

Отчёты о сбоях очень большие, их чтение может быть непонятным. Однако они содержат много информации об ошибке и могут помочь вам найти причину краша.

Для этого гайда, мы используем [этот отчёт об ошибках](/assets/players/crash-report-example.log).

:::details Show Crash Report

<<< @/public/assets/players/crash-report-example.log

:::

### Crash Report Sections {#crash-report-sections}

Crash reports consist of several sections, each separated using a header:

- `---- Minecraft Crash Report ----`, the summary of the report. This section will contain the main error that caused the crash, the time it occurred, and the relevant stack trace. This is the most important section of the crash report as the stack trace can usually contain references to the mod that caused the crash.
- `-- Last Reload --`, this section isn't really useful unless the crash occurred during a resource reload (<kbd>F3</kbd>+<kbd>T</kbd>). This section will contain the time of the last reload, and the relevant stack trace of any errors that occurred during the reload process. These errors are usually caused by resource packs, and can be ignored unless they are causing issues with the game.
- `-- System Details --`, this section contains information about your system, such as the operating system, Java version, and the amount of memory allocated to the game. This section is useful for determining if you are using the correct version of Java, and if you have allocated enough memory to the game.
  - In this section, Fabric will have included a custom line that says `Fabric Mods:`, followed by a list of all the mods you have installed. This section is useful for determining if any conflicts could have occurred between mods.

### Breaking Down the Crash Report {#breaking-down-the-crash-report}

Now that we know what each section of the crash report is, we can start to break down the crash report and find the cause of the crash.

Using the example linked above, we can analyze the crash report and find the cause of the crash, including the mods that caused the crash.

The stack trace in the `---- Minecraft Crash Report ----` section is the most important in this case, as it contains the main error that caused the crash.

:::details Show Error

<<< @/public/assets/players/crash-report-example.log{7}

:::

With the amount of mods mentioned in the stack trace, it can be difficult to point fingers, but the first thing to do is to look for the mod that caused the crash.

In this case, the mod that caused the crash is `snownee`, as it is the first mod mentioned in the stack trace.

However, with the amount of mods mentioned, it could mean there are some compatibility issues between the mods, and the mod that caused the crash may not be the mod that is at fault. In this case, it is best to report the crash to the mod author, and let them investigate the crash.

## Mixin Crashes {#mixin-crashes}

::: info

Mixins are a way for mods to modify the game without having to modify the game's source code. They are used by many mods, and are a very powerful tool for mod developers.

:::

When a mixin crashes, it will usually mention the mixin in the stack trace, and the class that the mixin is modifying.

Method mixins will contain `mod-id$handlerName` in the stack trace, where `mod-id` is the mod's ID, and `handlerName` is the name of the mixin handler.

```text:no-line-numbers
... net.minecraft.class_2248.method_3821$$$mod-id$handlerName() ... // [!code focus]
```

You can use this information to find the mod that caused the crash, and report the crash to the mod author.

## Что делать с отчётами о сбоях? {#what-to-do-with-crash-reports}

The best thing to do with crash reports is to upload them to a paste site, and then share the link with the mod author, either on their issue trackers or through some form of communication (Discord etc.).

This will allow the mod author to investigate the crash, potentially reproduce it, and solve the problem that caused it.

Common paste sites that are used frequently for crash reports are:

- [GitHub Gist](https://gist.github.com/)
- [mclo.gs](https://mclo.gs/)
- [Pastebin](https://pastebin.com/)
