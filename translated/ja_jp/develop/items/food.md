---
title: 食料アイテム
description: Learn how to add a FoodComponent to an item to make it edible, and configure it.
authors:
  - IMB11
---

食料は、マインクラフトのサバイバルモードにおいて核となる要素です。そのため、食料アイテムを作成する際には、他の食料アイテムとの兼ね合いを考慮する必要があります。

ゲームバランスを崩すような特殊なmodを作るのでなければ、以下の点などを考慮しましょう。

- その食料アイテムがどのくらい満腹度を増減させるか。
- どのようなステータス効果を付与するか。
- アイテムが入手可能になるのは、ゲームの序盤か終盤か。

## 食料コンポーネントを追加 {#adding-the-food-component}

To add a food component to an item, we can pass it to the `Item.Properties` instance:

```java
new Item.Properties().food(new FoodProperties.Builder().build())
```

上の処理では、アイテムが食べられるようになっただけで、それ以上の情報はありません。

The `FoodProperties.Builder` class has some methods that allow you to modify what happens when a player eats your item:

| Method               | Description                  |
| -------------------- | ---------------------------- |
| `nutrition`          | アイテムを食べた際に回復する満腹度の量を設定します。   |
| `saturationModifier` | アイテムを食べた際に回復する隠し満腹度の量を設定します。 |
| `alwaysEdible`       | 満腹でも食べられるかどうかを設定します。         |

When you've modified the builder to your liking, you can call the `build()` method to get the `FoodProperties`.

If you want to add status effects to the player when they eat your food, you will need to add a `Consumable` component alongside the `FoodProperties` component as seen in the following example:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

食料アイテムをゲーム内に登録するには、[初めてのアイテム作成](./first-item)で行った方法と同様に、registerメソッドを用いて登録します。

@[code transcludeWith=:::poisonous_apple](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

このアイテムは以下の特徴を持ちます。

- 現在の満腹度に関係なく食事が可能。
- 食べた場合、毒IIのステータス効果が 6 秒間付与される。

<VideoPlayer src="/assets/develop/items/food_0.webm">Eating the Poisonous Apple</VideoPlayer>
