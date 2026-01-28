---
title: Instalowanie oprogramowania Java na Windows
description: A step-by-step guide on how to install Java on Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

Ten przewodnik pokaże ci, jak zainstalować Javę 21 na systemie Windows.

Launcher Minecrafta zawiera własną instalację Javy, więc ta sekcja jest istotna tylko, jeśli chcesz użyć instalatora Fabric w formacie `.jar` lub jeśli chcesz użyć pliku `.jar` serwera Minecraft.

## 1. Sprawdź, czy nie masz już zainstalowanej Javy {#1-check-if-java-is-already-installed}

Aby sprawdzić, czy masz już zainstalowaną Javę, musisz najpierw otworzyć wiersz poleceń.

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![Okno dialogowe Uruchamiania w systemie Windows z wpisanym tekstem "cmd.exe" w pasku uruchamiania](/assets/players/installing-java/windows-run-dialog.png)

Po otwarciu wiersza poleceń wpisz `java -version` i naciśnij <kbd>Enter</kbd>.

Jeśli polecenie zostanie wykonane pomyślnie, to zobaczysz coś takiego. Jeśli polecenie się nie powiodło, przejdź do następnego kroku.

![Wiersz poleceń z wpisanym poleceniem "java -version"](/assets/players/installing-java/windows-java-version.png)

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

![Wiersz poleceń z wpisanym poleceniem "java -version"](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
