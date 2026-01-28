---
title: 建立專案
description: 如何一步一步使用 Fabric 範本模組產生器建立新模組專案的指南。
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric 提供了一種使用 Fabric 範本模組產生器輕鬆建立新模組專案的方法——如果你願意，你可以使用範例模組倉儲手動建立新專案，你應該參閱[手動建立專案](#manual-project-creation)章節。

## 生成專案 {#generating-a-project}

你可以使用 [Fabric 範本模組產生器](https://fabricmc.net/develop/template/) 生成你的新模組專案 —— 有一些必填的項目，例如模組名稱、套件名稱，以及你想基於其開發的 Minecraft 版本。

套件名稱應是小寫字母，由點區隔；套件名稱須具有獨特性，避免和其他開發者的套件衝突。 It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![生成器的預覽圖](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Advanced options section](/assets/develop/getting-started/template-generator-advanced.png)

必填項目輸入完後，點擊 `Generate` 按鈕，產生器會以 zip 檔的形式產生新專案供你使用。

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## 手動建立專案 {#manual-project-creation}

:::info PREREQUISITES

你會需要安裝 [Git](https://git-scm.com/) 來複製範例模組倉儲。

:::

如果你無法使用 Fabric 範本模組產生器，你可以按照以下步驟手動建立新專案。

首先，使用 Git 複製範例模組倉儲：

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. 如果 `.git` 資料夾沒有顯示，你需要在檔案管理器中開啟 `顯示隱藏的項目` 。

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

再次強調，如果你收到關於 Gradle 建構腳本的通知，你應該點擊 `Import Gradle Project` 按鈕。

### 修改模板 {#modifying-the-template}

專案導入後，你應該修改專案的細節，以符合你的模組：

- 修改專案中的 `gradle.properties` 檔案，將 `maven_group` 和 `archive_base_name` 屬性改成為你的模組的資訊。
- 修改 `fabric.mod.json` 文件，將 `id`、`name` 和 `description` 屬性改為與你的模組的資訊。
- 記得更新Minecraft的版本、映射、Loader和Loom的版本，你可以透過 <https://fabricmc.net/develop/> 查詢相關資訊，以確保它們符合你期望的目標版本。

你也可以修改套件名稱和模組的主類別以符合你的模組。
