---
title: 프로젝트 생성하기
description: Fabric 템플릿 모드 생성기로 새 모드를 생성하는 방법에 대한 단계별 설명서입니다.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

Fabric은 Fabric 템플릿 모드 생성기를 사용하여 간단하게 새로운 모드를 생성하는 방법을 제공하고 있습니다. 원하는 경우, 예제 모드 리포지토리를 사용하여 직접 새 프로젝트를 생성할 할 수 있습니다. 자세한 내용은 [직접 프로젝트 생성](#manual-project-creation)을 참조하십시오.

## 프로젝트 생성하기 {#generating-a-project}

[Fabric 템플릿 모드 생성기](https://fabricmc.net/develop/template/)를 사용하여 새 프로젝트를 생성할 수 있습니다. 모드 이름, 패키지 이름, 그리고 Minecraft 버전 등의 필수 항목을 입력해야 합니다.

패키지의 이름은 소문자여야 하며, 점으로 분류되고, 다른 개발자의 패키지와의 충돌을 피하기 위해 고유해야 합니다. It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![생성기 미리보기](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![고급 옵션 섹션](/assets/develop/getting-started/template-generator-advanced.png)

필수 입력란을 모두 입력했다면, `생성` 버튼을 클릭하여 zip 파일 형식으로 새 프로젝트를 다운로드할 수 있습니다.

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## 직접 프로젝트 생성 {#manual-project-creation}

:::info PREREQUISITES

리포지토리를 복사하려면 [Git](https://git-scm.com/)이 설치되어 있어야 합니다.

:::

Fabric 템플릿 모드 생성기를 사용할 수 없는 경우, 아래 과정으로 직접 프로젝트를 생성할 수 있습니다.

먼저 다음 명령어를 실행하여 Git으로 리포지토리를 복제합니다:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. 만약 파일 탐색기에서 `.git` 폴더가 보이지 않는다면, 파일 탐색기 설정에서 숨긴 항목 표시를 활성화해야 할 수 있습니다.

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

위에서 언급했듯이, 좌측 아래에 Gradle 빌드 스크립트에 대한 알림이 표시되면, `Import Gradle Project (Gradle 프로젝트 가져오기)`를 선택해야 합니다.

### 템플릿 수정하기 {#modifying-the-template}

프로젝트가 모두 가져와졌다면, 다음 단계를 통해 프로젝트의 세부 정보를 만들고자 하는 모드와 맞게 수정해야 합니다.

- 프로젝트의 `gradle.properties` 파일의 `maven_group` 및 `archive_base_name` 속성을 모드와 일치하도록 수정합니다.
- `fabric.mod.json` 파일의 `id`, `name` 및 `description` 속성을 모드와 일치하도록 수정합니다.
- Minecraft 버전, 매핑, 로더와 Loom 버전이 지원하고자 하는 버전과 일치하는지 확인합니다. (<https://fabricmc.net/develop/> 에서 확인할 수 있습니다)

또 모드의 메인 클래스와 패키지 이름이 모드와 일치하게 변경할 수 있습니다.
