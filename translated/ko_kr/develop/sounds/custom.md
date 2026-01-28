---
title: 사용자 지정 소리 만들기
description: 레지스트리로 새로운 사운드를 추가하고 사용하는 방법을 알아보세요.
authors:
  - JR1811
---

## 오디오 파일 준비하기 {#creating-custom-sounds}

오디오 파일은 정해진 방법으로 처리해야 합니다. OGG Vorbis는 오디오와 같은 멀티미디어 데이터를 위한 오픈 컨테이너 포맷이며, Minecraft 사운드 포맷으로도 사용됩니다. Minecraft가 거리 계산을 처리하는 방식에 대한 문제를 피하려면, 단일 채널(모노) 오디오가 필요합니다.

여러 신형 DAW (Digital Audio Workstation) 소프트위어는 이 파일 포맷을 사용하여 불러오고 내보낼 수 있습니다. 다음 예시에서는 오디오 파일을 올바른 형식으로 변환하기 위해 자유 오픈소스 소프트웨어 '[Audacity](https://www.audacityteam.org/)'가 사용됩니다. 다만, 다른 DAW도 가능합니다.

![Audacity에서 준비되지 않은 오디오 파일](/assets/develop/sounds/custom_sounds_0.png)

이 예시에서, [휘파람 소리](https://freesound.org/people/strongbot/sounds/568995/)는 Audacity로 임포트됩니다. 현재는 이 파일은 채널 2개(스테레오)를 가진  '.wav'로 저장됩니다. 필요 시 사운드를 수정합니다. 이 때 "track head"의 맨 위 drop-down 요소로 채널을 반드시 삭제해야 합니다.

![스테레오 트랙 분할하기](/assets/develop/sounds/custom_sounds_1.png)

![채널 삭제하기](/assets/develop/sounds/custom_sounds_2.png)

오디오 파일을 내보내거나 렌더링할 때, OGG파일을 선택하세요. REAPER와 같은 DAW는 다중 OGG 오디오 레이어 포맷을 지원할 수 있습니다. 이 경우 OGG Vorbis는 작동해야 합니다.

![OGG파일로 내보내기](/assets/develop/sounds/custom_sounds_3.png)

오디오 파일이 모드의 파일 크기를 상당히 늘릴 수 있다는 점 유의하세요. 파일을 수정하고 내보낼 때 프로젝트 크기를 최소화 하려면, 오디오 파일을 압축하면 됩니다.

## 오디오 파일 불러오기 {#loading-the-audio-file}

Add the new `resources/assets/example-mod/sounds` directory for the sounds in your mod, and put the exported audio file `metal_whistle.ogg` in there.

Continue with creating the `resources/assets/example-mod/sounds.json` file if it doesn't exist yet and add your sound to the sound entries.

@[code lang=json](@/reference/latest/src/main/resources/assets/example-mod/sounds.json)

부제의 항목은 더 많은 컨텍스트를 제공할 수 있습니다. The subtitle name is used in the language files in the `resources/assets/example-mod/lang` directory and will be displayed if the in-game subtitle setting is turned on and this custom sound is being played.

## 사용자 정의 소리 등록하기 {#registering-the-custom-sound}

To add the custom sound to the mod, register a SoundEvent in your [mod's initializer](../getting-started/project-structure#entrypoints).

```java
Registry.register(BuiltInRegistries.SOUND_EVENT, Identifier.fromNamespaceAndPath(MOD_ID, "metal_whistle"),
        SoundEvent.createVariableRangeEvent(Identifier.fromNamespaceAndPath(MOD_ID, "metal_whistle")));
```

## 정리하기 {#cleaning-up-the-mess}

얼마나 많은 레지스트리가 있는지에 따라, 관리하기 어려워질 수 있습니다. 이를 예방하기 위해 새로운 헬퍼 클래스를 활용할 수 있습니다.

헬퍼 클래스에 새 메서드를 추가하세요. 하나는, 모든 사운드를 등록하는 메서드, 다른 하나는 클래스를 초기화하는데 사용될 메서드입니다. 이제 새로운 커스텀 `SoundEvent` 정적 클래스 변수를 필요에 따라 추가할 수 있습니다.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/sound/CustomSounds.java)

이렇게 하면, 모드 이니셜라이저에 모든 커스텀 SoundEvent를 등록할 짧은 코드를 구현해야 합니다.

@[code lang=java transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/sound/ExampleModSounds.java)

## 커스텀 SoundEvent 사용하기 {#using-the-custom-soundevent}

커스텀 SoundEvent에 접근하려면 헬퍼 클래스를 사용합니다. Check out the [Playing Sounds](./using-sounds) page to learn how to play sounds.
