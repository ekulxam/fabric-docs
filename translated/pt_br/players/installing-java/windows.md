---
title: Instalando Java no Windows
description: A step-by-step guide on how to install Java on Windows.
authors:
  - IMB11
  - skycatminepokie
next: false
---

Este guia o orientará na instalação do Java 17 no Windows.

O Launcher do Minecraft vem com sua própria instalação do Java, portanto esta seção só é relevante se você quiser usar o instalador baseado em `.jar` do Fabric ou se quiser usar o `.jar` do Servidor do Minecraft.

## 1. Verificar Se o Java Já Está Instalado

Para verificar se o Java já está instalado, você deve primeiro abrir o prompt de comando.

You can do this by pressing <kbd>Windows</kbd>+<kbd>R</kbd> and typing `cmd.exe` into the box that appears.

![Caixa de diálogo Executar do Windows com "cmd.exe" na barra de execução](/assets/players/installing-java/windows-run-dialog.png)

Após abrir o prompt de comando, digite `java -version` e pressione <kbd>Enter</kbd>.

Se o comando for executado com êxito, você verá algo parecido com isto. Se o comando falhar, prossiga para o próximo passo.

![Prompt de comando com "java -version" digitado](/assets/players/installing-java/windows-java-version.png)

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

![Prompt de comando com "java -version" digitado](/assets/players/installing-java/windows-java-version.png)

If you encounter any issues, feel free to ask for help in the [Fabric Discord](https://discord.fabricmc.net/) in the `#player-support` channel.
