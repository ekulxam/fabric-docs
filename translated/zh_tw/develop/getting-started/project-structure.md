---
title: 專案結構
description: Fabric 模組專案結構概述
authors:
  - IMB11
---

本頁將介紹 Fabric 模組專案的結構和專案中每個檔案和資料夾的用途。

## `fabric.mod.json` {#fabric-mod-json}

`fabric.mod.json` 是向 Fabric Loader 描述你的模組的主要檔案。 這包含了模組的 ID、版本和前置等訊息。 這包含了模組的 ID、版本和前置等訊息。

`fabric.mod.json` 檔案中最重要的部分是:

- `id`: 模組的ID，應該要是獨一無二的。
- `name`: 模組的名稱。
- `environment`: 模組運行環境，可以是 `client` ，可以是 `server`，或是 `*`。
- `entrypoints`: 模組的進入點，例如 `main` 或 `client` 。
- `depends`: 你的模組所需要的前置模組。
- `mixins`: 模組提供的 Mixin。

You can see an example `fabric.mod.json` file below - this is the `fabric.mod.json` file for the mod that powers this documentation site.

:::details `fabric.mod.json` of the Example Mod

@[code lang=json](@/reference/latest/src/main/resources/fabric.mod.json)

:::

## Entrypoints {#entrypoints}

如前所述，`fabric.mod.json` 檔案包含 —— 個名為 `entrypoints` 的欄位 - 這個欄位用來指定你的模組提供的進入點。

預設情況下，模板模組生成 `main` 和 `client` 入口點：

- `main` 入口點用於通用程式碼，它包含在實作 `ModInitializer` 的類別中
- `client` 入口點用於僅限於客戶端的程式碼，它包含在實作 `ClientModInitializer` 的類別中

這些進入點會在遊戲啟動時分別被調用。

上面是一個簡單的 `main` 進入點範例，在遊戲啟動時向控制台記錄一條訊息。

@[code lang=java transcludeWith=#entrypoint](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

## `src/main/resources` {#src-main-resources}

`src/main/resources` 資料夾用於儲存模組的資源檔案，例如材質、模型和音效。

它也是 `fabric.mod.json` 和模組使用的 Mixin 配置檔案存放的位置。

Assets are stored in a structure that mirrors the structure of resource packs - for example, a texture for a block would be stored in `assets/example-mod/textures/block/block.png`.

## `src/client/resources` {#src-client-resources}

`src/client/resources` 資料夾用於儲存客戶端特定的資源檔案，例如僅在客戶端使用的材質、模型和音效。

## `src/main/java` {#src-main-java}

`src/client/java` 資料夾用於存放特定於客戶端的 Java 原始碼 —— 例如渲染程式碼或客戶端邏輯，如方塊顏色提供程式。

## `src/client/java` {#src-client-java}

`src/client/java` 資料夾用於存放特定於客戶端的 Java 原始碼 —— 例如渲染程式碼或客戶端邏輯，如方塊顏色提供程式。
