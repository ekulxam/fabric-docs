---
title: Tạo một dự án
description: Hưỡng dẫn từng bước cách tạo một dự mod bằng cách sử dụng Fabric template mod generator.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric cung cấp một vài cách tạo mod project đơn giản bằng cách sử dụng Fabric Template Mod Generator (Bộ tạo mẫu Fabric Mod) - nếu bạn muốn, bạn cũng có thể tạo một dự án mới bằng các sử dụng các repository mẫu từ những mod khác, và cũng nên tham khảo phần [Tạo dự án bằng cách thủ công](#manual-project-creation).

## Khởi tạo một dự án

Bạn có thể sử dụng  [Fabric Template Mod Generator](https://fabricmc.net/develop/template/) để tạo sẵn một dự án mới tính cho mod của bạn - ở đây nó sẽ cho bạn điền một số trường thông tin, như là tên mod, tên package, và phiên bản minecraft mà bạn muốn phát triển.

Tên package nên được đặt bằng chữ in thường, được ngăn cách bởi những dấu chấm, và tên phải riêng biệt để tránh tình trạng trùng với các package của những lập trình viên khác  It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![Bản xem trước của chức năng khởi tạo](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Phần Advanced options](/assets/develop/getting-started/template-generator-advanced.png)

Sau khi bạn đã điền đầy đủ thông tin, nhấn vào nút `Generate`, bộ khởi tạo sẽ tạo một dự án mới cho bạn với định dạng là một tệp zip.

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## Manual Project Creation {#manual-project-creation}

:::info PREREQUISITES

You will need [Git](https://git-scm.com/) installed in order to clone the example mod repository.

:::

If you cannot use the Fabric Template Mod Generator, you can create a new project manually by following these steps.

Firstly, clone the example mod repository using Git:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. If the `.git` folder does not appear, you should enable the display of hidden files in your file manager.

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

Again, as previously mentioned, if you receive a notification talking about a Gradle build script, you should click the `Import Gradle Project` button.

### Modifying the Template {#modifying-the-template}

Once the project has been imported, you should modify the project's details to match your mod's details:

- Modify the project's `gradle.properties` file to change the `maven_group` and `archive_base_name` properties to match your mod's details.
- Modify the `fabric.mod.json` file to change the `id`, `name`, and `description` properties to match your mod's details.
- Make sure to update the versions of Minecraft, the mappings, the Loader and the Loom - all of which can be queried through <https://fabricmc.net/develop/> - to match the versions you wish to target.

You can obviously change the package name and the mod's main class to match your mod's details.
