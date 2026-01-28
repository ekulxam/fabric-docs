---
title: Генерация исходных текстов в IntelliJ IDEA
description: Руководство по генерации исходных текстов Minecraft в IntelliJ IDEA.
authors:
  - dicedpixels
prev:
  text: Запуск игры в IntelliJ IDEA
  link: ./launching-the-game
next:
  text: Советы и рекомендации для IntelliJ IDEA
  link: ./tips-and-tricks
---

Набор инструментов Fabric позволяет получить доступ к исходному коду Minecraft, сгенерировав его локально, а для удобной навигации по нему можно использовать IntelliJ IDEA. Чтобы сгенерировать исходные тексты, необходимо запустить задачу Gradle `genSources`.

Это можно сделать из Gradle View, как показано выше, запустив задачу `genSources` в **Tasks** > **`fabric`**:
![Задача genSources в панели Gradle](/assets/develop/getting-started/intellij/gradle-gensources.png)

Также вы можете выполнить команду из терминала:

```sh:no-line-numbers
./gradlew genSources
```

![Задача genSources в терминале](/assets/develop/getting-started/intellij/terminal-gensources.png)

## Прикрепление источников {#attaching-sources}

IntelliJ требует еще одного дополнительного шага - прикрепления сгенерированных исходных текстов к проекту.

Чтобы сделать это, откройте любой класс Minecraft. Вы можете <kbd>Ctrl</kbd> + Клик, чтобы перейти к определению, которое открывает класс, или использовать "Поиск везде", чтобы открыть класс.

Для примера откроем файл `MinecraftServer.class`. Теперь вверху должен появиться синий баннер со ссылкой "**Выберите источники...**".

![Выберите источники](/assets/develop/getting-started/intellij/choose-sources.png)

Нажмите на кнопку "**Выбрать источники...**", чтобы открыть диалог выбора файлов. По умолчанию этот диалог открывается в правильном месте расположения сгенерированных источников.

Выберите файл, который заканчивается на **`-sources`**, и нажмите **Открыть**, чтобы подтвердить выбор.

![Диалоговое окно выбора источников](/assets/develop/getting-started/intellij/choose-sources-dialog.png)

You should now have the ability to search for references. If you are using a mapping set that contains Javadocs, like [Parchment](https://parchmentmc.org/) (for Mojang Mappings) or Yarn, you should now also see Javadocs.

![Комментарии Javadoc в исходных текстах](/assets/develop/getting-started/intellij/javadoc.png)
