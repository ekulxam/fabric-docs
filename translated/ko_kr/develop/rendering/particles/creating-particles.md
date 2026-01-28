---
title: 사용자 정의 입자 만들기
description: Fabric API를 통해 사용자 정의 입자를 만드는 방법을 알아보세요.
authors:
  - Superkat32
---

입자는 강력한 도구입니다. 아름다운 장면에 분위기를 더할 수도 있고, 보스와의 전투에 긴장감을 더할 수도 있습니다. 그럼 직접 한번 만들어 봅시다!

## 사용자 정의 입자 등록하기

이 튜토리얼에서는 엔드 막대기 입자처럼 움직이는 새로운 스파클 입자를 추가해 볼 예정입니다.

먼저, 모드 초기화 클래스에 모드 ID로 `ParticleType` 을 등록해 봅시다.

@[code lang=java transcludeWith=#particle_register_main](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

소문자로 "sparkle_particle"은 이후 입자의 텍스쳐를 위한 JSON 파일의 이름이 되게 됩니다. 곧 JSON 파일을 어떻게 생성하는지 알아볼 것입니다.

## 클라이언트측에서 등록하기

`ModInitializer` 엔트리포인트에 입자를 등록했다면, `ClientModInitializer`의 엔트리포인트에도 입자를 등록해야 합니다.

@[code lang=java transcludeWith=#particle_register_client](@/reference/latest/src/client/java/com/example/docs/ExampleModClient.java)

위 예시는 입자를 클라이언트측에 등록하는 방법입니다. 이제 엔드 막대기 입자 팩토리를 통해 입자에 움직임을 줘보겠습니다. 이렇게 하면 입자가 엔드 막대기 입자와 똑같이 움직이게 됩니다.

::: tip

You can see all the particle factories by looking at all the implementations of the `ParticleFactory` interface. This is helpful if you want to use another particle's behaviour for your own particle.

- IntelliJ's hotkey: <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>B</kbd>
- Visual Studio Code's hotkey: <kbd>Ctrl</kbd>+<kbd>F12</kbd>

:::

## Creating a JSON File and Adding Textures {#creating-a-json-file-and-adding-textures}

You will need to create 2 folders in your `resources/assets/example-mod/` folder.

| Folder Path          | Explanation                                                                                          |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| `/textures/particle` | The `particle` folder will contain all the textures for all of your particles.       |
| `/particles`         | The `particles` folder will contain all of the json files for all of your particles. |

For this example, we will have only one texture in `textures/particle` called "sparkle_particle_texture.png".

Next, create a new JSON file in `particles` with the same name as the JSON path that you used when registering your ParticleType. For this example, we will need to create `sparkle_particle.json`. This file is important because it lets Minecraft know which textures our particle should use.

@[code lang=json](@/reference/latest/src/main/resources/assets/example-mod/particles/sparkle_particle.json)

::: tip

You can add more textures to the `textures` array to create a particle animation. The particle will cycle through the textures in the array, starting with the first texture.

:::

## Testing the New Particle {#testing-the-new-particle}

Once you have completed the JSON file and saved your work, you are good to load up Minecraft and test everything out!

You can see if everything has worked by typing the following command:

```mcfunction
/particle example-mod:sparkle_particle ~ ~1 ~
```

![Showcase of the particle](/assets/develop/rendering/particles/sparkle-particle-showcase.png)

::: info

The particle will spawn inside the player with this command. You will likely need to walk backwards to actually see it.

:::

Alternatively, you can also use a command block to summon the particle with the exact same command.
