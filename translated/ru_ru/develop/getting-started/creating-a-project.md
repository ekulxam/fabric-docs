---
title: Создание проекта
description: Пошаговое руководство по созданию нового проекта мода с использованием генератора шаблонов Fabric.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric предоставляет лёгкий путь создания новых проектов мода используя генератор шаблонов модов Fabric, если вы хотите, вы можете создать новый проект используя репозиторий примерного мода, вам следует перейти на [Создание проекта вручную](#manual-project-creation).

## Генерирование проекта {#generating-a-project}

Вы можете воспользоваться [генератором шаблонов модов Fabric](https://fabricmc.net/develop/template/), чтобы сгенерировать новый проект для своего мода. Вам придётся заполнить обязательные поля, такие как название мода, название пакета и версию Minecraft, для которой вы хотите разрабатывать.

Имя пакета должно быть написано строчными буквами, разделено точками и быть уникальным, чтобы избежать конфликтов с пакетами других программистов. Обычно он оформляется в виде обратного интернет-домена, например `com.example.example-mod`.

:::warning ВАЖНО

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

Например, если ваш ID мода был **`my-cool-mod`**, вместо _`resources/assets/example-mod`_ используйте **`resources/assets/my-cool-mod`**.

:::

![Предварительный просмотр генератора](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Секция расширенных настроек](/assets/develop/getting-started/template-generator-advanced.png)

После того как вы заполнили обязательные поля, нажмите на кнопку `Generate`, и генератор для вас создадит новый проект в форме zip-файла.

Вам следует распаковать этот zip-файл в выбранное вами место, а затем открыть распакованную папку в вашей IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Избегайте каталогов облачных хранилищ (например, Microsoft OneDrive)
- Избегайте символов, отличных от ASCII (например, эмодзи, подчеркнутые буквы)
- Избегайте пробелов

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## Создание проекта вручную {#manual-project-creation}

:::info ТРЕБОВАНИЯ

Для клонирования репозитория примера мода вам потребуется установленный [Git](https://git-scm.com/).

:::

Если вы не можете использовать Fabric Template Mod Generator, вы можете создать новый проект вручную, следуя этим шагам.

Первое, клонируйте репозиторий примера мода используя Git:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

Затем следует удалить папку `.git` из клонированного репозитория, а затем открыть проект. Если папка `.git` не отображается, вам следует включить отображение скрытых элементов в вашем файловом менеджере.

После того как вы откроете проект в вашей IDE, она должна автоматически загрузить конфигурацию Gradle проекта и выполнить необходимые задачи по настройке.

Опять же, как описывалось выше, если вы получите уведомление, говорящее о скрипте сборки Gradle, вам следует нажать кнопку `Import Gradle Project`.

### Изменение Шаблона {#modifying-the-template}

После импорта проекта вам следует изменить детали проекта, чтобы они соответствовали деталям вашего мода:

- Измените файл `gradle.properties` в вашем проекте, чтобы изменить свойства `maven_group` и `archive_base_name` чтобы соответствовать деталям вашего мода.
- Измените файл `fabric.mod.json`, чтобы изменить свойства `id`, `name`, и `description`, чтобы соответствовать деталям вашего мода.
- Убедитесь, что вы обновили версии Minecraft, маппингов, Loader и Loom — все из которых можно запросить через <https://fabricmc.net/develop/> — чтобы они соответствовали версиям, которые вы хотите поддерживать.

Вы конечно же можете изменить имя пакета и основной класс мода, чтобы соответствовать деталям вашего мода.
