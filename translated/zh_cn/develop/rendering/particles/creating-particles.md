---
title: 创建自定义粒子
description: 学习如何使用 Fabric API 创建自定义粒子。
authors:
  - Superkat32
---

粒子是一种强大的工具， 可以为美丽的场景增添氛围，也可以为你的 boss 战添加紧张感。 让我们创建一个自定义粒子吧！

## 注册自定义粒子{#register-a-custom-particle}

我们会添加新的火花粒子，模仿末地烛的粒子移动。

首先，需要在你的[模组初始化器](../../getting-started/project-structure#entrypoints)中，使用你有模组 ID，注册 `ParticleType`。

@[code lang=java transcludeWith=#particle_register_main](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

小写字母“sparkle_particle”是粒子纹理的 JSON 路径。 稍后就会以这个名字，创建新的 JSON 文件。

## 客户端注册{#client-side-registration}

在模组的初始化器中注册粒子后，还需要在客户端的初始化器中注册粒子。

@[code lang=java transcludeWith=#particle_register_client](@/reference/latest/src/client/java/com/example/docs/ExampleModClient.java)

在这个例子中，我们在客户端注册我们的粒子。 使用末地烛粒子的 factory，给予粒子一些移动。 这意味着，我们的粒子就会像末地烛那样移动。

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
