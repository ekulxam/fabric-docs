---
title: VS Code 提示和技巧
description: 有用的提示和技巧让你的工作更轻松。
authors:
  - dicedpixels
prev:
  text: 在 VS Code 中生成源代码
  link: ./generating-sources
next: false
---

学习如何遍历生成源代码非常重要，这样你才能调试并理解 Minecraft 的内部工作原理。 这里我们概述了一些常见的 IDE 用法。

## 寻找 Minecraft 类 {#searching-for-a-minecraft-class}

生成源代码后， 你就应该可以搜索或查看 Minecraft 类。

### 查看类定义 {#viewing-class-definitions}

**快速打开**（<kbd>Ctrl</kbd>+<kbd>P</kbd>）：输入`#` 后跟类名（例如 `#Identifier`）。

![快速打开](/assets/develop/getting-started/vscode/quick-open.png)

**转到定义**（<kbd>F12</kbd>）：从源代码中，通过按 <kbd>Ctrl</kbd> + 单击其名称，或右键单击它并选择“转到定义”导航到类定义。

![转到定义](/assets/develop/getting-started/vscode/go-to-definition.png)

### 查找引用 {#finding-references}

你可以通过右键单击类名并单击**查找所有引用**来查找类的所有用法。

![查找所有引用](/assets/develop/getting-started/vscode/find-all-references.png)

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
