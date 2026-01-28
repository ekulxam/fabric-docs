---
title: Interazioni tra Oggetti Personalizzate
description: Impara come si crea un oggetto che usa gli eventi vanilla integrati.
authors:
  - IMB11
---

Gli oggetti basilari non possono arrivare lontano - prima o poi ti servirà un oggetto che interagisce con il mondo quando lo si usa.

Ci sono alcune classi chiave che devi comprendere prima di dare un'occhiata agli eventi degli oggetti vanilla.

## InteractionResult {#interactionresult}

An `InteractionResult` tells the game the status of the event, whether it was passed/ignored, failed or successful.

A succesful interaction can also be used to transform the stack in hand.

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
InteractionResult.SUCCESS.heldItemTransformedTo().success(heldStack);
```

## Event con Override {#overridable-events}

Fortunatamente, la classe Item ha molti metodi di cui si può fare override per aggiungere funzionalità ai tuoi oggetti.

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
