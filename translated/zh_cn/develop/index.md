---
title: 开发者指南
description: 我们的社区编写的开发者指南，涵盖许多主题，从创建一个模组和设置你的开发环境，一直到渲染、网络通信、数据生成等。
authors:
  - IMB11
  - its-miroma
  - itsmiir
authors-nogithub:
  - basil4088
---

Fabric 是一款适用于《Minecraft：Java 版》的轻量级模组工具链，设计简洁易用。 它允许开发者对原版游戏进行修改（“模组”），添加新功能或更改现有机制。

本文档将指导你使用 Fabric 进行模组开发，从[创建第一个模组](./getting-started/creating-a-project)和[设置环境](./getting-started/setting-up)，到[渲染](./rendering/basic-concepts)、[网络](./networking)、[数据生成](./data-generation/setup)等高级主题，应有尽有。

完整的页面列表请查看侧边栏。

::: tip

In case you need it at any time, a fully-working mod with all the source code of this documentation is available in the [`/reference` folder on GitHub](https://github.com/FabricMC/fabric-docs/tree/main/reference/latest).

:::

## Prerequisites {#prerequisites}

Before you start modding with Fabric, you need to have some understanding of developing with Java, and of Object-Oriented Programming in general.

Here are some resources that might help you familiarize with Java and OOP:

- [W3: Java Tutorials](https://www.w3schools.com/java/)
- [Codecademy: Learn Java](https://www.codecademy.com/learn/learn-java)
- [W3: Java OOP](https://www.w3schools.com/java/java_oop.asp)
- [Medium: Introduction to OOP](https://medium.com/@Adekola_Olawale/beginners-guide-to-object-oriented-programming-a94601ea2fbd)

## What Does Fabric Offer? {#what-does-fabric-offer}

The Fabric Project is centered around three main components:

- **Fabric Loader**: a flexible, platform-independent loader of mods, primarily designed for Minecraft: Java Edition
- **Fabric API**: a complementary set of APIs and tools mod developers can use when creating mods
- **Fabric Loom**: a [Gradle](https://gradle.org/) plugin, enabling developers to easily develop and debug mods

### What Does Fabric API Offer? {#what-does-fabric-api-offer}

Fabric API provides a wide set of APIs that build on top of the vanilla functionality to allow advanced or simpler development.

For example, it provides new hooks, events, utilities such as transitive access wideners, access to internal registries such as the compostable items registry, and more.
