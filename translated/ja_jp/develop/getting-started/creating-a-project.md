---
title: プロジェクトの作成
description: Fabric テンプレート Mod ジェネレータを使用した Mod 作成の段階的なガイド。
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric は、Mod プロジェクトを簡単に作成することを可能にする Fabric Template Mod Generator を提供しています。手動でプロジェクトを作成したい場合は、サンプルの Mod リポジトリを使用することができます。[手動で作成する](#manual-project-creation) セクションを参照してください。

## プロジェクトの生成 {#generating-a-project}

[Fabric Template Mod Generator](https://fabricmc.net/develop/template/) を使用すると新しいプロジェクトを生成することができます。パッケージ名、Mod 名、Mod が対応する Minecraft バージョンの入力が必要です。

パッケージ名は小文字で、ドットで区切り、一意であることが求められます。他のプログラマーのパッケージと競合しないようにしてください。 It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![ジェネレータのプレビュー](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Advanced Options セクション](/assets/develop/getting-started/template-generator-advanced.png)

必要な項目の入力が完了したら、`Generate` ボタンを押してください。ジェネレータが新しいプロジェクトを zip ファイル形式で生成します。

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## 手動によるプロジェクトの作成 {#manual-project-creation}

:::info PREREQUISITES

サンプルの Mod リポジトリをクローンするためには、[Git](https://git-scm.com/) のインストールが必要になります。

:::

Fabric テンプレート Mod ジェネレータを使用できない場合は、次のステップを踏むことで手動で新しいプロジェクトを作成できます。

まず、Git を使用してサンプル Mod リポジトリをクローンします:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. `.git` フォルダが表示されない場合は、ファイルマネージャで隠しファイルの表示を有効にする必要があります。

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

前述の通り、Gradle ビルドスクリプトに関する通知が表示された場合は、`Gradle プロジェクトのインポート` ボタンを押してください。

### テンプレートの編集 {#modifying-the-template}

プロジェクトがインポートされたら、プロジェクトの詳細をあなたの Mod の詳細に合わせて編集してください:

- `gradle.properties` ファイル内の `maven_group` と `archive_base_name` を編集します。
- `fabric.mod.json` ファイル内の `id`、`name`、`description` プロパティを編集します。
- Minecraft、マッピング、Fabric Loader、Loom のバージョンを、対応させたい値に編集します。これらは <https://fabricmc.net/develop/> で照会できます。

言うまでもないですが、パッケージ名と Mod のメインクラスも適宜編集してください。
