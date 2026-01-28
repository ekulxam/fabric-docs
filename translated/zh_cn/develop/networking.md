---
title: 网络通信
description: 使用 Fabric API 进行网络通信的一般指南。
authors:
  - Daomephsta
  - dicedpixels
  - Earthcomputer
  - FlooferLand
  - FxMorin
  - i509VCB
  - modmuss50
  - natanfudge
  - NetUserGet
  - NShak
  - parzivail
  - skycatminepokie
  - SolidBlock-cn
  - Voleil
  - Wxffel
  - YTG123-Mods
  - zulrang
---

Minecraft 中的网络用于使客户端和服务端可以相互通信。 网络通信是个广泛的话题，
因此本页面分为几个类别。

## 为什么网络很重要？ {#why-is-networking-important}

数据包是 Minecraft 网络的核心概念。
数据包由任意数据组成，可以从服务器发送到客户端，也可以从客户端发送到服务器。
请参阅下面的图表，它直观地展示了 Fabric 中的网络架构：

![双端架构](/assets/develop/networking/sides.png)

请留意数据包是如何充当服务器和客户端之间的桥梁的；这是因为你在游戏中所做的几乎一切都以某种方式涉及网络。
例如，当你发送一条聊天消息时，包含内容的数据包会发送到服务器。
然后，服务器会向所有其他客户端发送包含你消息的另一个数据包。

要记住的一件重要事情是始终有一个服务器在运行，即使在单人游戏和局域网中也是如此。 即使没有其他人和你一起玩，数据包仍用于客户端和服务器之间的通信。 在谈论网络中的端时，我们常用术语“**逻辑**客户端”和“**逻辑**服务器”。 集成的单人/局域网服务器和专用服务器都是**逻辑**服务器，但只有专用服务器可以被视为**物理**服务器。

当客户端和服务器之间的状态没有同步时，可能会遇到服务器或其他客户端不承认另一个客户端正在做的事情的问题。 这通常被称为“不同步”。 在编写自己的模组时，你可能需要发送数据包来保持服务器和所有客户端的状态同步。

## 网络通信简介 {#an-introduction-to-networking}

### 定义有效载荷 {#defining-a-payload}

::: info

A payload is the data that is sent within a packet.

:::

This can be done by creating a Java `Record` with a `BlockPos` parameter that implements `CustomPacketPayload`.

@[code lang=java transcludeWith=:::summon_Lightning_payload](@/reference/latest/src/main/java/com/example/docs/networking/basic/SummonLightningS2CPayload.java)

At the same time, we've defined:

- An `Identifier` used to identify our packet's payload. For this example our identifier will be
  `example-mod:summon_lightning`.

@[code lang=java transclude={13-13}](@/reference/latest/src/main/java/com/example/docs/networking/basic/SummonLightningS2CPayload.java)

- A public static instance of `CustomPayload.Id` to uniquely identify this custom payload. We will be referencing this
  ID in both our common and client code.

@[code lang=java transclude={14-14}](@/reference/latest/src/main/java/com/example/docs/networking/basic/SummonLightningS2CPayload.java)

- A public static instance of a `StreamCodec` so that the game knows how to serialize/deserialize the contents of the
  packet.

@[code lang=java transclude={15-15}](@/reference/latest/src/main/java/com/example/docs/networking/basic/SummonLightningS2CPayload.java)

We have also overridden `type` to return our payload ID.

@[code lang=java transclude={17-20}](@/reference/latest/src/main/java/com/example/docs/networking/basic/SummonLightningS2CPayload.java)

### Registering a Payload {#registering-a-payload}

Before we send a packet with our custom payload, we need to register it on both physical sides.

::: info

`S2C` and `C2S` are two common suffixes that mean _Server-to-Client_ and _Client-to-Server_ respectively.

:::

This can be done in our **common initializer** by using `PayloadTypeRegistry.playS2C().register` which takes in a
`CustomPayload.Id` and a `StreamCodec`.

@[code lang=java transclude={25-25}](@/reference/latest/src/main/java/com/example/docs/networking/basic/ExampleModNetworkingBasic.java)

A similar method exists to register client-to-server payloads: `PayloadTypeRegistry.playC2S().register`.

### Sending a Packet to the Client {#sending-a-packet-to-the-client}

To send a packet with our custom payload, we can use `ServerPlayNetworking.send` which takes in a `ServerPlayerEntity`
and a `CustomPayload`.

Let's start by creating our Lightning Tater item. You can override `use` to trigger an action when the item is used.
In this case, let's send packets to the players in the server level.

@[code lang=java transcludeWith=:::lightning_tater_item](@/reference/latest/src/main/java/com/example/docs/networking/basic/LightningTaterItem.java)

Let's examine the code above.

We only send packets when the action is initiated on the server, by returning early with a `isClient` check:

@[code lang=java transclude={22-24}](@/reference/latest/src/main/java/com/example/docs/networking/basic/LightningTaterItem.java)

We create an instance of the payload with the user's position:

@[code lang=java transclude={26-26}](@/reference/latest/src/main/java/com/example/docs/networking/basic/LightningTaterItem.java)

Finally, we get the players in the server level through `PlayerLookup` and send a packet to each player.

@[code lang=java transclude={28-30}](@/reference/latest/src/main/java/com/example/docs/networking/basic/LightningTaterItem.java)

::: info

Fabric API provides `PlayerLookup`, a collection of helper functions that will look up players in a server.

A term frequently used to describe the functionality of these methods is "_tracking_". It means that an entity or a
chunk
on the server is known to a player's client (within their view distance) and the entity or block entity should notify
tracking clients of changes.

Tracking is an important concept for efficient networking, so that only the necessary players are notified of changes by
sending packets.

:::

### Receiving a Packet on the Client {#receiving-a-packet-on-the-client}

To receive a packet sent from a server on the client, you need to specify how you will handle the incoming packet.

This can be done in the **client initializer**, by calling `ClientPlayNetworking.registerGlobalReceiver` and passing a
`CustomPayload.Id` and a `PlayPayloadHandler`, which is a Functional Interface.

In this case, we'll define the action to trigger within the implementation of `PlayPayloadHandler` implementation (as a
lambda expression).

@[code lang=java transcludeWith=:::client_global_receiver](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

Let's examine the code above.

We can access the data from our payload by calling the Record's getter methods. In this case `payload.pos()`. Which then
can be used to get the `x`, `y` and `z` positions.

@[code lang=java transclude={32-32}](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

Finally, we create a `LightningBolt` and add it to the level.

@[code lang=java transclude={33-38}](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

Now, if you add this mod to a server and when a player uses our Lightning Tater item, every player will see lightning
striking at the user's position.

<VideoPlayer src="/assets/develop/networking/summon-lightning.webm">Summon lightning using Lightning Tater</VideoPlayer>

### Sending a Packet to the Server {#sending-a-packet-to-the-server}

Just like sending a packet to the client, we start by creating a custom payload. This time, when a player uses a
Poisonous Potato on a living entity, we request the server to apply the Glowing effect to it.

@[code lang=java transcludeWith=:::give_glowing_effect_payload](@/reference/latest/src/main/java/com/example/docs/networking/basic/GiveGlowingEffectC2SPayload.java)

We pass in the appropriate codec along with a method reference to get the value from the Record to build this codec.

Then we register our payload in our **common initializer**. However, this time as _Client-to-Server_ payload by using
`PayloadTypeRegistry.playC2S().register`.

@[code lang=java transclude={26-26}](@/reference/latest/src/main/java/com/example/docs/networking/basic/ExampleModNetworkingBasic.java)

To send a packet, let's add an action when the player uses a Poisonous Potato. We'll be using the `UseEntityCallback`
event to
keep things concise.

We register the event in our **client initializer**, and we use `isClientSide()` to ensure that the action is only triggered
on the logical client.

@[code lang=java transcludeWith=:::use_entity_callback](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

We create an instance of our `GiveGlowingEffectC2SPayload` with the necessary arguments. In this case, the network ID
of
the targeted entity.

@[code lang=java transclude={51-51}](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

Finally, we send a packet to the server by calling `ClientPlayNetworking.send` with the instance of our
`GiveGlowingEffectC2SPayload`.

@[code lang=java transclude={52-52}](@/reference/latest/src/client/java/com/example/docs/network/basic/ExampleModNetworkingBasicClient.java)

### Receiving a Packet on the Server {#receiving-a-packet-on-the-server}

This can be done in the **common initializer**, by calling `ServerPlayNetworking.registerGlobalReceiver` and passing a
`CustomPayload.Id` and a `PlayPayloadHandler`.

@[code lang=java transcludeWith=:::server_global_receiver](@/reference/latest/src/main/java/com/example/docs/networking/basic/ExampleModNetworkingBasic.java)

::: info

It is important that you validate the content of the packet on the server side.

In this case, we validate if the entity exists based on its network ID.

@[code lang=java transclude={30-30}](@/reference/latest/src/main/java/com/example/docs/networking/basic/ExampleModNetworkingBasic.java)

Additionally, the targeted entity has to be a living entity, and we restrict the range of the target entity from the
player to 5.

@[code lang=java transclude={32-32}](@/reference/latest/src/main/java/com/example/docs/networking/basic/ExampleModNetworkingBasic.java)

:::

Now when any player tries to use a Poisonous Potato on a living entity, the glowing effect will be applied to it.

<VideoPlayer src="/assets/develop/networking/use-poisonous-potato.webm">Glowing effect is applied when a Poisonous Potato is used on a living entity</VideoPlayer>
