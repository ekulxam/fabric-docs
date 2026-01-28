---
title: 自定义魔咒效果
description: 学习如何创建自己的魔咒效果。
authors:
  - CelDaemon
  - krizh-p
---

从 1.21 开始，Minecraft 中的自定义魔咒通过”数据驱动“的方式添加。 这让添加一些简单的魔咒（如增加攻击伤害）变得更容易，但创建复杂的魔咒则更具挑战性。 这个过程包括将魔咒分解成 _效果组件_。

效果组件包含定义魔咒特殊效果的代码。 Minecraft 原版已支持一些默认效果，例如物品损害值、击退和经验。

::: tip

Be sure to check if the default Minecraft effects satisfy your needs by visiting [the Minecraft Wiki's Enchantment Effect Components page](https://minecraft.wiki/w/Enchantment_definition#Effect_components). This guide assumes you understand how to configure "simple" data-driven enchantments and focuses on creating custom enchantment effects that aren't supported by default.

:::

## Custom Enchantment Effects {#custom-enchantment-effects}

Start by creating an `enchantment` folder, and within it, create an `effect` folder. Within that, we'll create the `LightningEnchantmentEffect` record.

Next, we can create a constructor and override the `EnchantmentEntityEffect` interface methods. We'll also create a `CODEC` variable to encode and decode our effect; you can read more about [Codecs here](../codecs).

The bulk of our code will go into the `apply()` event, which is called when the criteria for your enchantment to work is met. We'll later configure this `Effect` to be called when an entity is hit, but for now, let's write simple code to strike the target with lightning.

@[code transcludeWith=#entrypoint](@/reference/latest/src/main/java/com/example/docs/enchantment/effect/LightningEnchantmentEffect.java)

Here, the `amount` variable indicates a value scaled to the level of the enchantment. We can use this to modify how effective the enchantment is based on level. In the code above, we are using the level of the enchantment to determine how many lightning strikes are spawned.

## Registering the Enchantment Effect {#registering-the-enchantment-effect}

Like every other component of your mod, we'll have to add this `EnchantmentEffect` to Minecraft's registry. To do so, add a class `ModEnchantmentEffects` (or whatever you want to name it) and a helper method to register the enchantment. Be sure to call the `registerModEnchantmentEffects()` in your main class, which contains the `onInitialize()` method.

@[code transcludeWith=#entrypoint](@/reference/latest/src/main/java/com/example/docs/enchantment/ModEnchantmentEffects.java)

## Creating the Enchantment {#creating-the-enchantment}

Now we have an enchantment effect! The final step is to create an enchantment that applies our custom effect. While this can be done by creating a JSON file similar to those in datapacks, this guide will show you how to generate the JSON dynamically using Fabric's data generation tools. To begin, create an `ExampleModEnchantmentGenerator` class.

Within this class, we'll first register a new enchantment, and then use the `configure()` method to create our JSON programmatically.

@[code transcludeWith=#entrypoint](@/reference/latest/src/client/java/com/example/docs/datagen/ExampleModEnchantmentGenerator.java)

Before proceeding, you should ensure your project is configured for data generation; if you are unsure, [view the respective docs page](../data-generation/setup).

Lastly, we must tell our mod to add our `EnchantmentGenerator` to the list of data generation tasks. To do so, simply add the `EnchantmentGenerator` to this inside of the `onInitializeDataGenerator` method.

@[code transcludeWith=:::custom-enchantments:register-generator](@/reference/latest/src/client/java/com/example/docs/datagen/ExampleModDataGenerator.java)

Now, when you run your mod's data generation task, enchantment JSONs will be generated inside the `generated` folder. An example can be seen below:

@[code](@/reference/latest/src/main/generated/data/example-mod/enchantment/thundering.json)

You should also add translations to your `en_us.json` file to give your enchantment a readable name:

```json
"enchantment.example-mod.thundering": "Thundering",
```

You should now have a working custom enchantment effect! Test it by enchanting a weapon with the enchantment and hitting a mob. An example is given in the following video:

<VideoPlayer src="/assets/develop/enchantment-effects/thunder.webm">Using the Thundering Enchantment</VideoPlayer>
