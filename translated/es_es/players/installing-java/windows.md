---
title: Instalar Java en Windows
description: A step-by-step guide on how to install Java on Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

Esta guía te enseñará como instalar Java 17 en Windows.

El launcher de Minecraft viene con su propia instalación de Java, por lo que esta sección solo es relevante si quieres usar el instalador de Fabric `.jar`, o si quieres usar el `.jar` del Servidor de Minecraft.

## 1. Verifica si Java ya está instalado

Para comprobar si Java ya está instalado, primero debes abrir la línea de comandos.

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![Diálogo de Ejecución de Windows con "cmd.exe" en la barra de ejecución](/assets/players/installing-java/windows-run-dialog.png)

Una vez abierta la línea de comandos, escribe `java -version` y presiona <kbd>Enter</kbd>.

Si el comando corre exitosamente, verás algo similar a esto. Si el comando falla, procede al siguiente paso.

![Línea de comandos con "java -version" escrito](/assets/players/installing-java/windows-java-version.png)

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

![Línea de comandos con "java -version" escrito](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
