---
title: 음식 아이템
description: 아이템을 섭취할 수 있고, 다양한 효과를 구성할 수 있게 FoodComponent에 추가하는 방법을 알아보세요.
authors:
  - IMB11
---

음식은 Minecraft 생존의 핵심 요소로, 섭취 가능한 아이템을 만들 때에는 다른 아이템의 용도를 고려해야 합니다.

모드를 이용하여 사기적인 아이템을 만드는 것이 아니라면, 다음을 고려해야 합니다:

- 배고픔 수치를 얼마나 추가하거나 빼낼 것인가
- 어떤 물약 효과를 부여할 것인가
- 어느 시기에 획득할 수 있는가

## 음식 요소 추가하기 {#adding-the-food-component}

To add a food component to an item, we can pass it to the `Item.Properties` instance:

```java
new Item.Properties().food(new FoodProperties.Builder().build())
```

지금으로선 단지 아이템을 먹는 기능 그 이상 이하도 존재하지 않습니다.

The `FoodProperties.Builder` class has some methods that allow you to modify what happens when a player eats your item:

| 메소드                  | 설명                                        |
| -------------------- | ----------------------------------------- |
| `nutrition`          | 아이템이 채울 배고픔의 양을 설정합니다.    |
| `saturationModifier` | 아이템이 채울 포만감의 양을 설정합니다.    |
| `alwaysEdible`       | 배고픔에 상관없이 항상 먹을 수 있게 합니다. |

When you've modified the builder to your liking, you can call the `build()` method to get the `FoodProperties`.

If you want to add status effects to the player when they eat your food, you will need to add a `Consumable` component alongside the `FoodProperties` component as seen in the following example:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

[첫 번째 아이템 만들기](./first-item)의 예시와 유사하게, 위의 구성 요소를 사용하여 아이템을 추가해 봅시다:

@[code transcludeWith=:::poisonous_apple](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

이렇게 하면 아이템에 다음과 같은 효과가 적용됩니다:

- 배고픔에 상관없이 항상 먹을 수 있습니다.
- 항상 먹을 때마다 독 II를 6초 부여합니다.

<VideoPlayer src="/assets/develop/items/food_0.webm">독사과를 섭취하는 모습</VideoPlayer>
