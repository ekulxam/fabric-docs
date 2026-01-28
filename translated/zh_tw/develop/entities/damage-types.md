---
title: 傷害類型
description: 了解如何新增自訂傷害類型。
authors:
  - dicedpixels
  - hiisuuii
  - MattiDragon
---

傷害類型定義實體可以承受的傷害類型。 自 Minecraft 1.19.4 起，新傷害類型的建立已
成為資料驅動，這表示它們是使用 JSON 檔案建立的。

## Creating a Damage Type {#creating-a-damage-type}

讓我們建立一個名為「_馬鈴薯_」的自訂傷害類型。 首先為您的自訂傷害建立一個 JSON 檔案。 這個檔案將放在您模組的 `data` 目錄中的 `damage_type` 子目錄。

```text:no-line-numbers
resources/data/example-mod/damage_type/tater.json
```

結構如下：

@[code lang=json](@/reference/latest/src/main/generated/data/example-mod/damage_type/tater.json)

這個傷害類型會使玩家在每次受到傷害時，增加0.1[飢餓消耗度](https://minecraft.wiki/w/Hunger#Exhaustion_level_increase)，此傷害會由活物、非玩家來源造成（例如：一個方塊）。 此外，傷害會隨著世界難易度增加。

::: info

有關所有可能的鍵和值，請參閱 [Minecraft Wiki](https://zh.minecraft.wiki/w/伤害类型/JSON格式)。

:::

### Accessing Damage Types Through Code {#accessing-damage-types-through-code}

When we need to access our custom damage type through code, we will use it's `ResourceKey` to build an instance
of `DamageSource`.

The `ResourceKey` can be obtained as follows:

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/damage/ExampleModDamageTypes.java)

### Using Damage Types {#using-damage-types}

為了演示自訂傷害類型的使用，我們將使用一個名為「_馬鈴薯方塊_」的自訂方塊。 Let's make is so that
when a living entity steps on a _Tater Block_, it deals _Tater_ damage.

You can override `stepOn` to inflict this damage.

We start by creating a `DamageSource` of our custom damage type.

@[code lang=java transclude={22-26}](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

Then, we call `entity.damage()` with our `DamageSource` and an amount.

@[code lang=java transclude={27-27}](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

The complete block implementation:

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

Now whenever a living entity steps on our custom block, it'll take 5 damage (2.5 hearts) using our custom damage type.

### Custom Death Message {#custom-death-message}

You can define a death message for the damage type in the format of `death.attack.message_id` in our
mod's `en_us.json` file.

```json
{
  "death.attack.tater": "%1$s died from Tater damage!"
}
```

因我們的傷害類型死亡後，您將看到以下死亡訊息：

![玩家物品欄中的效果](/assets/develop/tater-damage-death.png)

### Damage Type Tags {#damage-type-tags}

部分傷害類型能無視盔甲、狀態效果等等。 標籤用於控制這些類型的屬性傷害類型。

您可以在 `data/minecraft/tags/damage_type` 中找到現有的傷害類型標籤。

::: info

Refer to the [Minecraft Wiki](https://minecraft.wiki/w/Tag#Damage_types) for a comprehensive list of damage type
tags.

:::

讓我們將我們的馬鈴薯傷害類型新增到 `bypasses_armor` 傷害類型標籤中。

要將我們的傷害類型新增到這些標籤之一，需要在 `minecraft` 命名空間下建立一個 JSON 檔案。

```text:no-line-numbers
data/minecraft/tags/damage_type/bypasses_armor.json
```

內容如下：

@[code lang=json](@/reference/latest/src/main/generated/data/minecraft/tags/damage_type/bypasses_armor.json)

透過將 `replace` 鍵設為 `false` 來確保您的標籤不會替換現有標籤。
