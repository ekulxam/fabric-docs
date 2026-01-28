---
title: 自定义物品交互
description: 学习如何创建使用内置原版事件的物品
authors:
  - IMB11
---

基础物品的功能是有限的——你还是需要个能在使用时与世界交互的物品。

有些关键的类你必须理解，然后才能看看原版的物品事件。

## InteractionResult {#interactionresult}

`InteractionResult` 告诉游戏事件的状态，即事件是否被通过/忽略、失败或成功。

一次成功的交互也可以用来改变手中的堆叠。

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
InteractionResult.SUCCESS.heldItemTransformedTo().success(heldStack);
```

## 可重写事件 {#overridable-events}

`Item` 类有许多方法可以被重写，从而为物品添加额外的功能。

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
