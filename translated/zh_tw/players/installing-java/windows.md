---
title: 在 Windows 上安裝 Java
description: A step-by-step guide on how to install Java on Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

這個指南將引導你在 Windows 上安裝 Java 21。

Minecraft 啟動器附帶了自己的 Java 安裝，因此這部分只在你想使用 Fabric 的 `.jar` 安裝程式，或者你想使用 Minecraft 伺服器的 `.jar` 時相關。

## 1. 檢查 Java 是否已安裝 {#1-check-if-java-is-already-installed}

要檢查 Java 是否已安裝，你首先必須開啟命令提示字元。

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![Windows執行對話方塊中的「cmd.exe」](/assets/players/installing-java/windows-run-dialog.png)

開啟命令提示字元後，輸入 `java -version` 並按下 <kbd>Enter</kbd> 鍵。

如果命令成功執行，你會看到類似這樣的內容。 如果命令失敗，請繼續進行下一步。 如果命令失敗，請繼續進行下一步。 如果命令失敗，請繼續進行下一步。

![命令提示字元中輸入了「java -version」](/assets/players/installing-java/windows-java-version.png)

::: warning

To use Minecraft 1.21.11, you'll need at least Java 21 installed.

If this command displays any version lower than 21, you'll need to update your existing Java installation; keep reading this page.

:::

## 2. Download the Java 21 Installer {#2-download-the-java-installer}

To install Java 21, you'll need to download the installer from [Adoptium](https://adoptium.net/temurin/releases?version=21&os=windows&arch=any&mode=filter).

You'll want to download the `Windows Installer (.msi)` version:

![Adoptium download page with Windows Installer (.msi) highlighted](/assets/players/installing-java/windows-download-java.png)

You should choose `x86` if you have a 32-bit operating system, or `x64` if you have a 64-bit operating system.

The majority of modern computers will have a 64-bit operating system. If you are unsure, try using the 64-bit download.

## 3. Run the Installer! {#3-run-the-installer}

Follow the steps in the installer to install Java 21. When you reach this page, you should set the following features to "Entire feature will be installed on local hard drive":

- `Set JAVA_HOME environment variable` - This will be added to your PATH.
- `JavaSoft (Oracle) registry keys`

![Java 21 installer with "Set JAVA_HOME variable" and "JavaSoft (Oracle) registry keys" highlighted](/assets/players/installing-java/windows-wizard-screenshot.png)

Once you've done that, you can click `Next` and continue with the installation.

::: warning

Windows won't always tell other programs that Java is installed until you restart your computer.

**Make sure to restart your computer before continuing!**

:::

## 4. Verify That Java 21 Is Installed {#4-verify-that-java-is-installed}

Once the installation is complete, you can verify that Java 21 is installed by opening the command prompt again and typing `java -version`.

If the command runs successfully, you will see something like shown before, where the Java version is displayed:

![命令提示字元中輸入了「java -version」](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
