---
title: Windows에 Java 설치하기
description: A step-by-step guide on how to install Java on Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

이 가이드는 Windows에 Java 21을 설치하는 과정을 안내해 줄 것입니다.

Minecraft 런처에서 자체적인 Java 설치를 제공하기 때문에, 이 단락은 `.jar` 기반의 Fabric 설치기를 사용하고자 하거나, Minecraft 서버 `.jar`를 사용하고자 할 때에만 유용합니다.

## 1. 이미 Java가 설치되었는지 확인하기 {#1-check-if-java-is-already-installed}

Java가 설치되어 있는지 확인하려면, 먼저 명령 프롬프트를 시작해야 합니다.

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![입력란에 "cmd.exe"가 입력된 Windows 실행 다이얼로그](/assets/players/installing-java/windows-run-dialog.png)

명령 프롬프트를 시작하는 데 성공했다면, `java -version`을 입력하고 <kbd>엔터 키</kbd>를 누릅니다.

명령어가 성공적으로 실행되었다면, 아래와 같이 표시될 것입니다. 명령어가 실행되지 않았다면, 다음 단계로 바로 이동하십시오.

!["java -version"이 입력된 명령 프롬프트](/assets/players/installing-java/windows-java-version.png)

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

!["java -version"이 입력된 명령 프롬프트](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
