---
title: 사용자 지정 아이템 상호 작용
description: 바닐라에 내장된 이벤트를 사용하는 아이템을 추가하는 방법을 알아보세요.
authors:
  - IMB11
---

결국에는 사용했을 때 세계와 상호 작용하는 아이템이 필요할 것입니다.

바닐라 아이템 이벤트를 알아보기 전에 먼저 이해해야 할 몇 가지 주요 클래스가 있습니다.

## InteractionResult {#interactionresult}

An `InteractionResult` tells the game the status of the event, whether it was passed/ignored, failed or successful.

A succesful interaction can also be used to transform the stack in hand.

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
InteractionResult.SUCCESS.heldItemTransformedTo().success(heldStack);
```

## 오버라이드 가능한 이벤트 {#overridable-events}

다행히도, 아이템 클래스에는 아이템에 추가 기능을 만들 때 오버라이드할 수 있는 여러 메소드가 있습니다.

::: info

A great example of these events being used can be found in the [Playing SoundEvents](../sounds/using-sounds) page, which uses the `useOn` event to play a sound when the player right clicks a block.

:::

| Method                 | Information                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| `hurtEnemy`            | Ran when the player hits an entity.                     |
| `mineBlock`            | Ran when the player mines a block.                      |
| `inventoryTick`        | Ran every tick whilst the item is in an inventory.      |
| `onCraftedPostProcess` | Ran when the item is crafted.                           |
| `useOn`                | Ran when the player right clicks a block with the item. |
| `use`                  | Ran when the player right clicks the item.              |

## The `use()` Event {#use-event}

Let's say you want to make an item that summons a lightning bolt in front of the player - you would need to create a custom class.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/item/custom/LightningStick.java)

The `use` event is probably the most useful out of them all - you can use this event to spawn our lightning bolt, you should spawn it 10 blocks in front of the players facing direction.

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/item/custom/LightningStick.java)

As usual, you should register your item, add a model and texture.

As you can see, the lightning bolt should spawn 10 blocks in front of you - the player.

<VideoPlayer src="/assets/develop/items/custom_items_0.webm">Using the Lightning Stick</VideoPlayer>
