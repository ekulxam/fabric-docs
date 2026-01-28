---
title: Setting Up Your Development Environment
description: Fabric을 이용하여 모드를 만들기 위해 개발 환경을 조성하는 방법에 대한 단계별 길라잡이입니다.
authors:
  - 2xsaiko
  - andrew6rant
  - asiekierka
  - daomephsta
  - dicedpixels
  - falseresync
  - IMB11
  - its-miroma
  - liach
  - mkpoli
  - modmuss50
  - natanfudge
  - SolidBlock-cn
  - telepathicgrunt
authors-nogithub:
  - siglong
outline: false
---

## Install JDK 21 {#install-jdk-21}

To develop mods for Minecraft 1.21.11, you will need JDK 21.

If you need help installing Java, you can refer to the [Java installation guides](../../players/installing-java/).

## Set Up Your IDE {#set-up-your-ide}

To start developing mods with Fabric, you will need to set up a development environment using IntelliJ IDEA (recommended), or alternatively Visual Studio Code.

<ChoiceComponent :choices="[
{
 name: 'IntelliJ IDEA',
 href: './intellij-idea/setting-up',
 icon: 'simple-icons:intellijidea',
 color: '#FE2857',
},
{
 name: 'Visual Studio Code',
 href: './vscode/setting-up',
 icon: 'codicon:vscode',
 color: '#007ACC',
},
]" />
