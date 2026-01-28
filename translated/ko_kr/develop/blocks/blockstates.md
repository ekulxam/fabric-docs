---
title: 블록 상태
description: 블록 상태가 블록에 시각적 기능을 추가하는 좋은 방법인 이유에 대해 알아보세요.
authors:
  - IMB11
---

블록 상태는 속성의 형태로 한 블록의 모든 정보를 포함하는 Minecraft 세계에서 단일 블록에 등록된 데이터의 조각입니다. 바닐라가 블록 상태에 저장하는 몇 가지 속성의 예시입니다:

- Rotation: 주로 원목이나 기타 자연 블록에 사용됩니다.
- Activated: 레드스톤 장치 및 화로나 훈연기와 같은 블록에 사용됩니다.
- Age: 작물, 식물, 묘목, 켈프 등에 사용됩니다.

세계의 용량을 줄이고, 블록 엔티티 안에 NBT 데이터를 저장할 필요를 없게 하고, TPS 문제를 막는 데 유용하기에 블록 상태가 유용한 이유를 알 수 있을 겁니다!

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## 예시: 기둥 블록 {#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

Minecraft는 이미 빠르게 특정 종류의 블록을 만들 수 있도록 하는 몇 가지의 맞춤 클래스가 이미 있습니다. 이 예시는 "Condensed Oak Log" 블록을 생성하여 `axis` 속성을 사용하여 블록을 생성하는 과정을 보여 줍니다.

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

기둥 블록은 윗면과 옆면으로 된 두 가지의 텍스처가 있습니다. `block/cube_column` 모델을 사용합니다.

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">텍스처</DownloadEntry>

기둥 블록이 수평과 수직, 두 개의 위치가 있기 때문에, 분리된 각각의 모델 파일을 만들어야 합니다:

- `block/cube_column_horizontal` 모델을 확장하는 `condensed_oak_log_horizontal.json`.
- `block/cube_column` 모델을 확장하는 `condensed_oak_log.json`.

`condensed_oak_log_horizontal.json` 파일의 예시:

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

모든 블록 상태 파일 안의 수정자에 대한 더 자세한 보기는 [Minecraft 위키 - 모델 문단 (Block States) (영어)](https://minecraft.wiki/w/Tutorials/Models#Block_states)에 있습니다.

:::

다음으로, 블록 상태 파일을 생성하여야 합니다. 블록 상태 파일은 마법이 일어나는 곳입니다. 기둥 블록은 세 개의 축이 있으므로, 다음 상황에서 특정 모델을 사용할 것입니다:

- `axis=x` - 블록이 X축을 따라 설치되면, 양의 X축 방향을 향하도록 모델을 회전할 것입니다.
- `axis=y` - 블록이 Y축을 따라 설치되면, 기본 수직 모델을 사용할 것입니다.
- `axis=z` - 블록이 Z축을 따라 설치되면, 양의 Z축 방향을 향하도록 모델을 회전할 것입니다.

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

언제나 블록에 대한 번역과 두 모델 중 하나의 부모격이 되는 아이템 모델을 만들어야 할 것입니다.

![게임 안에서의 기둥 블록의 예시](/assets/develop/blocks/blockstates_1.png)

## 사용자 정의 블록 상태 {#custom-block-states}

사용자 지정 블록 상태는 블록이 고유한 속성을 가지고 있을 때 유용합니다. 때때로 바닐라 속성을 재사용하여 만든 블록을 발견할 수도 있습니다.

이 예시는 `activated`라 불리는 고유한 불 (boolean) 속성을 생성할 것입니다. 플레이어가 블록에 오른쪽 클릭할 때, `activated=false`에서 `activated=true`로 변환하여서 이에 따라 텍스처를 변경합니다.

### 속성 만들기 {#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. 빌더에 접근하기 위하여 메서드를 덮어써야 할 것입니다:

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

또한 사용자 정의 블록의 생성자에서 `activated` 속성의 기본 상태를 설정하여야 합니다.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### 속성 시각화하기 {#visualizing-the-property}

이 예시는 플레이어가 블록과 상호작용을 할 때 불 `activated` 속성을 뒤집습니다. We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### 속성 사용하기 {#using-the-property}

블록 상태 파일을 만들기 전, 블록 모델과 같이 블록이 활성화되었을 때와 비활성화되었을 때의 텍스처를 제공하여야 합니다.

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">텍스처</DownloadEntry>

블록 모델의 지식을 이용해 두 가지의 블록 모델을 만듭니다. 하나는 활성화된 상태용, 다른 하나는 비활성화된 상태용입니다. 이렇게 하면 블록 상태 파일을 계속하여 만들 수 있습니다.

새 속성을 만들었으면, 그 속성을 설명하기 위하여 블록에 대한 블록 상태 파일을 업데이트하여야 합니다.

만약 블록에 여러 개의 속성이 있는 경우, 가능한 모든 조합을 고려하여야 합니다. 예시로, `activated`와 `axis`는 6개의 조합으로 이어집니다. (`activated`가 가능한 두 개의 값과, `axis`가 가능한 세 개의 값).

블록이 오직 두 개의 가능한 변형이 있고, 오직 한 개의 속성 (`activated`)가 있으므로, 블록 상태 JSON 파일은 다음과 같을 것입니다.

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

예시 블록이 바다 랜턴이기 때문에, 또한 `activated` 속성이 true (참)일 때 발광하도록 만들어야 합니다. 이는 블록을 등록할 때 생성자로 전달된 블록 설정을 통하여 완료될 수 있습니다.

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

모든 작업을 완료하면, 최종 결과는 다음과 같이 보일 것입니다:

<VideoPlayer src="/assets/develop/blocks/blockstates_3.webm">게임 내에서의 바다 랜턴 블록</VideoPlayer>
