---
title: カスタムイベントの登録
description: Learn how to create an item that uses built-in vanilla events.
authors:
  - IMB11
---

アイテム登録だけでは、複雑な効果を持つアイテムを作成することはできません。ここでは、あるイベントに対して何かしらの処理をさせるといった、イベント駆動型の処理をアイテムに定義し、もっと面白いアイテムを作りましょう。

バニラのアイテムイベントを確認する前に、重要なクラスについて理解しましょう。(ただし、おそらく`TypedActionResult`は古いバージョンしか対応していません。ミスだと思われます。)

## InteractionResult {#interactionresult}

An `InteractionResult` tells the game the status of the event, whether it was passed/ignored, failed or successful.

A succesful interaction can also be used to transform the stack in hand.

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
InteractionResult.SUCCESS.heldItemTransformedTo().success(heldStack);
```

## イベント定義メソッドのオーバーライド {#overridable-events}

Itemクラスは、追加機能の設定に関するメソッドを複数持っています。追加機能を設定するには、そのメソッドをオーバーライドすることで可能になります。

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
