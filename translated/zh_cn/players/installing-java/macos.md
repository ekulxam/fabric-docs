---
title: 在 macOS 上安装 Java
description: 在 macOS 上安装 Java 的逐步指南。
authors:
  - dexman545
  - ezfe
next: false
---

本指南将会指引你在 macOS 上安装 Java 21。

Minecraft 启动器自带 Java 安装，因此本节仅在你想要使用 Fabric 的 `.jar` 安装程序，或想要使用 Minecraft 服务器 `.jar` 时才相关。

## 1. 检查 Java 是否已经安装 {#1-check-if-java-is-already-installed}

在终端（位于 `/Applications/Utilities/Terminal.app`）中输入以下命令，然后按 <kbd>Enter</kbd> 键：

```sh
$(/usr/libexec/java_home -v 21)/bin/java --version
```

你应该会看到类似以下内容：

```text:no-line-numbers
openjdk 21.0.9 2025-10-21 LTS
OpenJDK Runtime Environment Temurin-21.0.9+10 (build 21.0.9+10-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.9+10 (build 21.0.9+10-LTS, mixed mode, sharing)
```

请注意版本号：在上例中，版本号为 `21.0.9`。

::: warning

To use Minecraft 1.21.11, you'll need at least Java 21 installed.

If this command displays any version lower than 21, you'll need to update your existing Java installation; keep reading this page.

:::

## 2. Downloading and Installing Java 21 {#2-downloading-and-installing-java}

We recommend using [Adoptium's build of OpenJDK 21](https://adoptium.net/temurin/releases?version=21&os=mac&arch=any&mode=filter):

![Temurin Java Download Page](/assets/players/installing-java/macos-download-java.png)

Make sure to select version "21 - LTS", and choose the `.PKG` installer format.
You should also choose the correct architecture depending on your system's chip:

- If you have an Apple M-series chip, choose `aarch64` (the default)
- If you have an Intel chip, choose `x64`
- Follow these [instructions to know which chip is in your Mac](https://support.apple.com/en-us/116943)

After downloading the `.pkg` installer, open it and follow the prompts:

![Temurin Java Installer](/assets/players/installing-java/macos-installer.png)

You will have to enter your administrator password to complete the installation:

![macOS Password Prompt](/assets/players/installing-java/macos-password-prompt.png)

### Using Homebrew {#using-homebrew}

If you already have [Homebrew](https://brew.sh) installed, you can install Java 21 using `brew` instead:

```sh
brew install --cask temurin@21
```

## 3. Verify That Java 21 Is Installed {#3-verify-that-java-is-installed}

Once the installation is complete, you can verify that Java 21 is active by opening Terminal again and typing `$(/usr/libexec/java_home -v 21)/bin/java --version`.

If the command succeeds, you should see something like this:

```text:no-line-numbers
openjdk 21.0.9 2025-10-21 LTS
OpenJDK Runtime Environment Temurin-21.0.9+10 (build 21.0.9+10-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.9+10 (build 21.0.9+10-LTS, mixed mode, sharing)
```

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
