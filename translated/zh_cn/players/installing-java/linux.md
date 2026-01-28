---
title: 在 Linux 上安装 Java
description: 在 Linux 上安装 Java 的逐步指南。
authors:
  - IMB11
next: false
---

这个指南将会指引您在 Linux 上安装 Java 21。

Minecraft 启动器自带 Java 安装，因此本节仅在你想要使用 Fabric 的 `.jar` 安装程序，或想要使用 Minecraft 服务器 `.jar` 时才相关。

## 1. 检查 Java 是否已经安装{#1-check-if-java-is-already-installed}

打开终端输入 `java -version` 并按下 `回车`。

![输入 "java -version" 的终端](/assets/players/installing-java/linux-java-version.png)

::: warning

To use Minecraft 1.21.11, you'll need at least Java 21 installed.

如果此命令显示的版本低于 21，则需要更新现有的 Java 安装，请继续阅读本页。

:::

## 2. Downloading and Installing Java 21 {#2-downloading-and-installing-java}

We recommend using OpenJDK 21, which is available for most Linux distributions.

### Arch Linux {#arch-linux}

::: info

For more information on installing Java on Arch Linux, see the [Arch Linux Wiki](https://wiki.archlinux.org/title/Java).

:::

You can install the latest JRE from the official repositories:

```sh
sudo pacman -S jre-openjdk
```

If you're running a server without the need for a graphical UI, you can install the headless version instead:

```sh
sudo pacman -S jre-openjdk-headless
```

If you plan to develop mods, you'll need the JDK instead:

```sh
sudo pacman -S jdk-openjdk
```

### Debian/Ubuntu {#debian-ubuntu}

You can install Java 21 using `apt` with the following commands:

```sh
sudo apt update
sudo apt install openjdk-21-jdk
```

### Fedora {#fedora}

You can install Java 21 using `dnf` with the following commands:

```sh
sudo dnf install java-21-openjdk
```

If you don't need a graphical UI, you can install the headless version instead:

```sh
sudo dnf install java-21-openjdk-headless
```

If you plan to develop mods, you'll need the JDK instead:

```sh
sudo dnf install java-21-openjdk-devel
```

### Other Linux Distributions {#other-linux-distributions}

If your distribution isn't listed above, you can download the latest JRE from [Adoptium](https://adoptium.net/temurin/)

You should refer to an alternative guide for your distribution if you plan to develop mods.

## 3. Verify That Java 21 Is Installed {#3-verify-that-java-is-installed}

Once the installation is complete, you can verify that Java 21 is installed by opening a terminal and typing `java -version`.

If the command runs successfully, you will see something like shown before, where the Java version is displayed:

![输入 "java -version" 的终端](/assets/players/installing-java/linux-java-version.png)
