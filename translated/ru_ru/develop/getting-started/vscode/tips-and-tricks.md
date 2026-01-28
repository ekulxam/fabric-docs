---
title: Советы и рекомендации по работе с VS Code
description: Полезные советы и рекомендации, которые облегчат вашу работу.
authors:
  - dicedpixels
prev:
  text: Генерация исходных текстов в VS Code
  link: ./generating-sources
next: false
---

Важно научиться работать с генерируемыми исходниками, чтобы отладить и понять внутреннюю работу Minecraft. Здесь мы расскажем о некоторых распространенных вариантах использования IDE.

## Поиск класса Minecraft {#searching-for-a-minecraft-class}

После создания источников. вы должны иметь возможность искать или просматривать классы Minecraft.

### Просмотр определений классов {#viewing-class-definitions}

**Быстрое открытие** (<kbd>Ctrl</kbd>+<kbd>P</kbd>): Введите `#`, за которым следует имя класса (например, `#Identifier`).

![Быстрое открытие](/assets/develop/getting-started/vscode/quick-open.png)

**Перейти к определению** (<kbd>F12</kbd>): В исходном коде перейдите к определению класса, нажав <kbd>Ctrl</kbd> + щелкнув на его имени, или щелкнув ПКМ и выбрав "Go to Definition".

![Перейти к определению](/assets/develop/getting-started/vscode/go-to-definition.png)

### Поиск ссылок {#finding-references}

Вы можете найти все случаи использования класса, щелкнув ПКМ на имени класса и нажав **Find All References**.

![Найти все ссылки](/assets/develop/getting-started/vscode/find-all-references.png)

::: info

If the functions above do not work as expected, it's likely that sources are not attached properly. This can generally be fixed by cleaning up the workspace cache.

- Click the **Show Java Status Menu** button in the status bar.

![Show Java Status](/assets/develop/getting-started/vscode/java-ready.png)

- In the menu that just opened, click **Clean Workspace Cache...** and confirm the operation.

![Clear Workspace](/assets/develop/getting-started/vscode/clear-workspace.png)

- Close and reopen the project.

:::

## Viewing Bytecode {#viewing-bytecode}

Viewing bytecode is necessary when writing mixins. However, Visual Studio Code lacks native support for bytecode viewing, and the few extensions which add it might not work.

In such case, you can use Java's inbuilt `javap` to view bytecode.

- **Locate the path to Minecraft JAR:**

  Open the Explorer view, expand the **Java Projects** section. Expand the **Reference Libraries** node in the project tree and locate a JAR with `minecraft-` in its name. Right-click on the JAR and copy the full path.

  It might look something like this:

  ```:no-line-numbers
  C:/project/.gradle/loom-cache/minecraftMaven/net/minecraft/minecraft-merged-503b555a3d/1.21.8-net.fabricmc.yarn.1_21_8.1.21.8+build.1-v2/minecraft-merged-503b555a3d-1.21.8-net.fabricmc.yarn.1_21_8.1.21.8+build.1-v2.jar
  ```

![Copy Path](/assets/develop/getting-started/vscode/copy-path.png)

- **Run `javap`:**

  You can then run `javap` by providing the above path as the `cp` (class path) and the fully qualified class name as the final argument.

  ```sh
  javap -cp C:/project/.gradle/loom-cache/minecraftMaven/net/minecraft/minecraft-merged-503b555a3d/1.21.8-net.fabricmc.yarn.1_21_8.1.21.8+build.1-v2/minecraft-merged-503b555a3d-1.21.8-net.fabricmc.yarn.1_21_8.1.21.8+build.1-v2.jar -c -private net.minecraft.util.Identifier
  ```

  This will print the bytecode in your terminal output.

![javap](/assets/develop/getting-started/vscode/javap.png)
