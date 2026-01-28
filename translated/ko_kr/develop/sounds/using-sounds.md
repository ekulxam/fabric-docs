---
title: 소리 재생
description: 소리 이벤트를 재생하는 방법에 대해 알아보기
authors:
  - JR1811
---

Minecraft는 선택할 수 있는 소리의 종류가 무진장 많습니다. Mojang이 제공한 모든 기본 소리 이벤트 인스턴스를 확인하려면 `SoundEvents`를 확인하세요.

## 모드 내에서 소리 재생하기 {#using-sounds}

소리를 사용하려는 경우 `playSound()` 메서드를 논리 서버 측에서 실행하는 것을 잊지 마세요!

In this example, the `interactLivingEntity()` and `useOn()` methods for a custom interactive item are used to play a "placing copper block" and a pillager sound.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CustomSoundItem.java)

`playSound()` 메서드는 `LivingEntity` 오브젝트와 함께 사용되었습니다. 오직 SoundEvent에서만 소리의 크기와 음높이를 지정하면 됩니다. You can also use the `playSound()` method from the level instance to have a higher level of control.

@[code lang=java transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/item/custom/CustomSoundItem.java)

### SoundEvent와 SoundCategory {#soundevent-and-soundcategory}

SoundEvent는 어느 소리를 낼지 정의합니다. 또한 [나만의 SoundEvent를 등록](./custom)해 나만의 소리를 추가할 수 있습니다.

Minecraft는 게임 내 설정에 몇몇 개의 오디오 슬라이더가 있습니다. `SoundCategory` 열거형은 어느 슬라이더가 소리의 크기를 조절하는 데 할당될지 결정합니다.

### 소리의 크기와 높이 {#volume-and-pitch}

소리 매개변수는 약간 오해의 소지가 있습니다. `0.0f - 1.0f` 범위 내에서는 실제 소리의 크기를 조정할 수 있습니다. 만일 숫자가 이보다 커진다면 `1.0f`의 소리 크기만 사용될 것이며 소리가 들리는 거리만 조절될 겁니다. 대략적인 블록의 거리는`소리 크기 * 16`으로 계산할 수 있습니다.

음높이 매개변수는 음높이 값을 증가 또는 감소시키고, 또한 소리의 재생 길이를 변경합니다. 큰 숫자는 음높이와 속도를 높이는 반면 `(0.5f - 1.0f)` 범위 내에서는 음높이와 속도가 줄어듭니다. `0.5f` 이하의 숫자는 음높이 `0.5f`와 같은 음높이입니다.
