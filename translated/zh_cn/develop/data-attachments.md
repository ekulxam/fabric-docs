---
title: 数据附件
description: 涵盖 Fabric 新数据附件 API 的基本用法的指南。
authors:
  - cassiancc
  - DennisOchulor
---

数据附加 API 是 Fabric API 中一项近期推出的实验性功能。 它允许开发者轻松地将任意数据附加到实体、方块实体、Level 和区块。 附加的数据可以通过 [Codec](./codecs) 和流 [Codec](./codecs) 进行存储和同步，因此在使用前你应该熟悉这些 Codec。

## 创建数据附件 {#creating-attachments}

首先，你需要调用 `AttachmentRegistry.create` 方法。 以下示例创建一个基本数据附件，该附件不会在重启后同步或保留。

@[code lang=java transcludeWith=:::string](@/reference/latest/src/main/java/com/example/docs/attachment/ExampleModAttachments.java)

`AttachmentRegistry` 包含一些用于创建基本数据附件的方法，包括：

- `AttachmentRegistry.create()`：创建一个数据附件。 重启游戏会清除附件。
- `AttachmentRegistry.createPersistent()`：创建一个在游戏重启后仍然有效的数据附件。
- `AttachmentRegistry.createDefaulted()`：创建一个带有默认值的数据附件，你可以使用 `getAttachedOrCreate` 读取该默认值。 重启游戏会清除附件。

通过应用[方法链模式](https://en.wikipedia.org/wiki/Method_chaining)，可以使用 `create` 的 `builder` 参数来复制和进一步自定义每个方法的行为。

### Syncing a Data Attachment {#syncing-attachments}

If you need a Data Attachment to both be persistent and synced between server and clients, you can set that behavior using the `create` method, which allows configuration through a `builder` chain. For example:

@[code lang=java transcludeWith=:::pos](@/reference/latest/src/main/java/com/example/docs/attachment/ExampleModAttachments.java)

The example above synced to every player, but that might not fit your use case. Here are some other default predicates, but you can also build your own by referencing the `AttachmentSyncPredicate` class.

- `AttachmentSyncPredicate.all()`: Syncs the Attachment with all clients.
- `AttachmentSyncPredicate.targetOnly()`: Syncs the Attachment only with the target it is attached to. Note that the syncing can only happen if the target is a player.
- `AttachmentSyncPredicate.allButTarget()`: Syncs the Attachment with every client except the target it is attached to. Note that the exception can only apply if the target is a player.

### Persisting Data Attachments {#persisting-attachments}

Data Attachments can also be set to persist across game restarts by calling the `persistent` method on the builder chain. It takes in a `Codec` so that the game knows how to serialize the data.

They can be set to perdure even after the death or [conversion](https://minecraft.wiki/w/Mob_conversion) of the target with the `copyOnDeath` method.

@[code lang=java transcludeWith=:::persistent](@/reference/latest/src/main/java/com/example/docs/attachment/ExampleModAttachments.java)

## Reading From a Data Attachment {#reading-attachments}

Methods to read from a Data Attachment have been injected onto the `Entity`, `BlockEntity`, `ServerLevel` and `ChunkAccess` classes. Using it is as simple as calling one of the methods, which return the value of the attached data.

```java
// Checks if the given AttachmentType has attached data, returning a boolean.
entity.hasAttached(EXAMPLE_STRING_ATTACHMENT);

// Gets the data associated with the given AttachmentType, or `null` if it doesn't exist.
entity.getAttached(EXAMPLE_STRING_ATTACHMENT);

// Gets the data associated with the given AttachmentType, throwing a `NullPointerException` if it doesn't exist.
entity.getAttachedOrThrow(EXAMPLE_STRING_ATTACHMENT);

// Gets the data associated with the given AttachmentType, setting the value if it doesn't exist.
entity.getAttachedOrSet(EXAMPLE_STRING_ATTACHMENT, "basic");
entity.getAttachedOrSet(EXAMPLE_BLOCK_POS_ATTACHMENT, new BlockPos(0, 0, 0););

// Gets the data associated with the given AttachmentType, returning the provided value if it doesn't exist.
entity.getAttachedOrElse(EXAMPLE_STRING_ATTACHMENT, "basic");
entity.getAttachedOrElse(EXAMPLE_BLOCK_POS_ATTACHMENT, new BlockPos(0, 0, 0););
```

## Writing To a Data Attachment {#writing-attachments}

Methods to write to a Data Attachment have been injected onto the `Entity`, `BlockEntity`, `ServerLevel` and `ChunkAccess` classes. Calling one of the following methods will update the value of the attached data, and return the previous value (or `null` if there wasn't one).

```java
// Sets the data associated with the given AttachmentType, returning the previous value.
entity.setAttached(EXAMPLE_STRING_ATTACHMENT, "new value");

// Modifies the data associated with the given AttachmentType in place, returning the currently attached value. Note that currentValue is null if there is no previously attached data.
entity.modifyAttached(EXAMPLE_STRING_ATTACHMENT, currentValue -> "The length was " + (currentValue == null ? 0 : currentValue.length()));

// Removes the data associated with the given AttachmentType, returning the previous value.
entity.removeAttached(EXAMPLE_STRING_ATTACHMENT);
```

::: warning

You should always use values with immutable types for Data Attachments, and you should also update them with API methods only. Doing otherwise may cause the Data Attachment to not persist or sync properly.

:::

## Larger Attachments {#larger-attachments}

Although Data Attachments could store any form of data for which a Codec can be written, they shine when syncing individual values. This is because a Data Attachment is immutable: modifying part of its value (for example a single field of an object) means replacing it entirely, triggering a full sync to every client tracking it.

Instead, you could achieve more intricate Attachments by splitting them into multiple fields, and organizing them with a helper class. For example, if you need two fields related to a player's stamina, you may build something like this:

@[code lang=java transcludeWith=:::stamina](@/reference/latest/src/main/java/com/example/docs/attachment/Stamina.java)

This helper class can then be used like so:

```java
Player player = getPlayer();
Stamina.get(player).getCurrentStamina();
```
