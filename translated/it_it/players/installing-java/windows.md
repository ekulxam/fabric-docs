---
title: Installare Java su Windows
description: Una guida passo per passo su come installare Java su Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

Questa guida ti spiegherà come installare Java 21 su Windows.

Il Launcher di Minecraft ha la sua versione di Java installata, quindi questa sezione è rilevante solo se vuoi usare l'installer `.jar` di Fabric, oppure se vuoi usare il `.jar` del Server di Minecraft.

## 1. Controlla Se Java È Già Installato {#1-check-if-java-is-already-installed}

Per controllare se Java è già installato devi prima aprire il prompt dei comandi.

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![Dialogo Esegui su Windows che mostra "cmd.exe" scritto nella barra](/assets/players/installing-java/windows-run-dialog.png)

Una volta aperto il prompt dei comandi, scrivi `java -version` e premi <kbd>Invio</kbd>.

Se il comando funziona correttamente, vedrai qualcosa come questo. Se il comando ha fallito, procedi con il prossimo passaggio.

![Il prompt dei comandi con scritto "java -version"](/assets/players/installing-java/windows-java-version.png)

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

![Il prompt dei comandi con scritto "java -version"](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
