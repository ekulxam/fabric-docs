---
title: IntelliJ IDEA로 게임 실행하기
description: Learn how to utilize the various launch profiles to start and debug your mods in a live game environment through IntelliJ IDEA.
authors:
  - IMB11
  - Tenneb22
prev:
  text: Opening a Project in IntelliJ IDEA
  link: ./opening-a-project
next:
  text: Generating Sources in IntelliJ IDEA
  link: ./generating-sources
---

Fabric Loom provides a variety of launch profiles to help you start and debug your mods in a live game environment. This guide will cover the various launch profiles and how to use them to debug and playtest your mods.

## Launch Profiles {#launch-profiles}

만약 여러분이 IntelliJ IDEA를 사용 중이라면, 화면 오른쪽 상단에서 실행 프로필을 찾을 수 있습니다. 드롭다운 메뉴를 클릭하여 사용 가능한 실행 프로필 목록을 확인하세요.

목록에 있는 클라이언트 및 서버 프로필을 선택하여 클라이언트 모드 또는 디버그 모드 중 원하는 방식으로 구동할 수 있습니다:

![실행 프로필들](/assets/develop/getting-started/launch-profiles.png)

## Gradle 작업 {#gradle-tasks}

만약 여러분이 터미널을 사용 중이라면, 아래의 Gradle 명령을 이용하여 게임을 실행할 수 있습니다:

- `./gradlew runClient` - 클라이언트 모드로 게임을 실행합니다.
- `./gradlew runServer` - 서버 모드로 게임을 실행합니다.

이 방식의 유일한 문제점은 코드를 쉽게 디버깅할 수 없다는 점입니다. 만약 여러분이 코드를 디버깅하고 싶다면, IntelliJ IDEA의 실행 프로필을 이용하거나 IDE의 Gradle 통합 기능을 사용해야 합니다.

## 클래스 실시간 수정 {#hotswapping-classes}

여러분이 게임을 디버그 모드로 실행할 때, 클래스들을 게임을 다시 시작하지 않고도
실시간으로 수정할 수 있습니다. 이 방식은 코드의 변경점들을 빠르게 테스트하는데 유용합니다.

You're still quite limited though:

- You can't add or remove methods
- You can't change method parameters
- You can't add or remove fields

However, by using the [JetBrains Runtime](https://github.com/JetBrains/JetBrainsRuntime), you are able to circumvent most of the limitations, and even add or remove classes and methods. This should allow most changes to take effect without restarting the game.

Don't forget to add the following to the VM Arguments option in your Minecraft run configuration:

```text:no-line-numbers
-XX:+AllowEnhancedClassRedefinition
```

## Hotswapping Mixins {#hotswapping-mixins}

If you're using Mixins, you can hotswap your Mixin classes without restarting the game. This is useful for quickly testing changes to your Mixins.

You will need to install the Mixin Java agent for this to work though.

### 1. Locate the Mixin Library Jar {#1-locate-the-mixin-library-jar}

In IntelliJ IDEA, you can find the mixin library jar in the "External Libraries" section of the "Project" section:

![Mixin Library](/assets/develop/getting-started/mixin-library.png)

You will need to copy the jar's "Absolute Path" for the next step.

### 2. Add the `-javaagent` VM Argument {#2-add-the--javaagent-vm-argument}

In your "Minecraft Client" and or "Minecraft Server" run configuration, add the following to the VM Arguments option:

```text:no-line-numbers
-javaagent:"path to mixin library jar here"
```

![VM Arguments Screenshot](/assets/develop/getting-started/vm-arguments.png)

Now, you should be able to modify the contents of your mixin methods during debugging and have the changes take effect without restarting the game.
