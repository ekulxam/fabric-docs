---
title: 이벤트
description: Fabric API가 제공하는 이벤트에 대한 가이드.
authors:
  - Daomephsta
  - dicedpixels
  - Draylar
  - JamiesWhiteShirt
  - Juuxel
  - liach
  - mkpoli
  - natanfudge
  - PhoenixVX
  - SolidBlock-cn
  - YanisBft
authors-nogithub:
  - stormyfabric
---

Fabric API는 게임에서 발생 가능한 이벤트에 관여할 수 있도록 만드는 시스템입니다.

이벤트란, 동일한 후크를 구현한 모드 사이의 성능과 호환성을 향상시키고, 일반적인 상황을 만족시키도록 하는 후크입니다. 이벤트는 일부 경우에 한하여 mixin을 대체할 수 있습니다.

Fabric API는 다수의 개발자가 후킹할 수 있는 Minecraft 코드베이스의 핵심 영역을 위한 이벤트를 제공합니다.

이벤트는 _callbacks_ 를 호출하고 저장하는 `net.fabricmc.fabric.apt.event.Event`의 인스턴스로 표현됩니다. 보통, 콜백을 위한 단일 이벤트 인스턴스가 있습니다. 이 콜백은 콜백 인터페이스의 정적 필드인 `EVENT`에 저장되지만, 다른 형태로 존재할 수 있습니다. 예를 들어, `ClientTickEvents`는 몇가지 관련된 이벤트를 그룹화합니다.

## 콜백 {#callbacks}

Callback은 이벤트에 매개변수로 넘겨지는 코드 조각을 의미합니다. 게임에서 이벤트가 발생하면, 주어진 코드가 실행됩니다.

### 콜백 인터페이스 {#callback-interfaces}

Each event has a corresponding callback interface. Callbacks are registered by calling `register()` method on an event instance, with an instance of the callback interface as the argument.

## 이벤트 리스닝(Listening to Events) {#listening-to-events}

아래 예제는 도구를 들지 않은 플레이어가 블록을 가격할 때 피해를 입히는 `AttackBlockCallback`을 등록합니다.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/event/ExampleModEvents.java)

### 기존 노획물 목록에 아이템 추가하기 {#adding-items-to-existing-loot-tables}

노획물 목록에 아이템을 추가하고 싶은 상황이 있다고 가정해 봅시다. 이를 테면, 바닐라 블록이나 엔티티에 커스텀 드롭을 추가하는 것을 의미합니다.

가장 간단한 방법은, 노획물 목록 파일을 바꿔버리는 것인데, 이는 다른 모드를 망칠 수 있습니다. 만약 노획물 목록 파일을 바꾸고 싶다면 어떨까요? 노획물 목록을 덮어쓰지 않고 아이템을 추가하는 방법을 살펴볼 것입니다.

이 문서에서는 석탄 전리품 테이블에 달걀을 추가하는 방법을 설명합니다.

#### 노획물 목록을 불러오는 경우 {#listening-to-loot-table-loading}

Fabric API에는 노획물 목록을 불러왔을 경우 호출될 `LootTableEvents.MODIFY`이벤트가 있습니다. 콜백은 [모드 이니셜라이저](./getting-started/project-structure#entrypoints)에 등록될 수 있습니다. 아래 코드는 현재 노획물 목록이 석탄인지 확인합니다.

@[code lang=java transclude={38-40}](@/reference/latest/src/main/java/com/example/docs/event/ExampleModEvents.java)

#### 노획물 목록에 아이템 추가하기 {#adding-items-to-the-loot-table}

In loot tables, items are stored in _loot pool entries_, and entries are stored in _loot pools_. To add an item, we'll need to add a pool with an item entry to the loot table.

노획물 목록에 `LootPool#builder`를 만들고, 추가할 수 있습니다.

Our pool doesn't have any items either, so we'll make an item entry using `ItemEntry#builder` and add it to the pool.

@[code highlight={6-7} transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/event/ExampleModEvents.java)

## 커스텀 이벤트 {#custom-events}

게임의 일부 영역은 Fabric API가 제공하는 후크를 가지고 있지 않기 때문에, mixin이나 커스텀 이벤트를 만들어야 합니다.

양털을 가위로 자를 때 호출되는 이벤트를 만듭니다. 이벤트를 만드는 과정은:

- 콜백 인터페이스 이벤트 만들기
- mixin에서 이벤트 호출하기
- 테스트 구현 만들기

### 이벤트 콜백 인터페이스 만들기 {#creating-the-event-callback-interface}

The callback interface describes what must be implemented by event listeners that will listen to your event. The callback interface also describes how the event will be called from our mixin. It is conventional to place an `Event` object as a field in the callback interface, which will identify our actual event.

For our `Event` implementation, we will choose to use an array-backed event. The array will contain all event listeners that are listening to the event.

이 구현은 이벤트 리스너를 순서대로 호출합니다. 이 작업은 `ActionResult.PASS`를 반환하지 않은 이벤트가 존재할 때까지 실행됩니다. This means that a listener can say "_cancel this_", "_approve this_" or "_don't care, leave it to the next event listener_" using its return value.

Using `ActionResult` as a return value is a conventional way to make event handlers cooperate in this fashion.

You'll need to create an interface that has an `Event` instance and method for response implementation. A basic setup for our sheep shear callback is:

@[code lang=java transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/event/SheepShearCallback.java)

더 자세히 살펴보자면, Invoker가 호출될 경우 모든 리스너를 반복합니다:

@[code lang=java transclude={21-22}](@/reference/latest/src/main/java/com/example/docs/event/SheepShearCallback.java)

We then call our method (in this case, `interact`) on the listener to get its response:

@[code lang=java transclude={33-33}](@/reference/latest/src/main/java/com/example/docs/event/SheepShearCallback.java)

If the listener says we have to cancel (`ActionResult.FAIL`) or fully finish (`ActionResult.SUCCESS`), the callback returns the result and finishes the loop. `ActionResult.PASS` moves on to the next listener, and in most cases should result in success if there are no more listeners registered:

@[code lang=java transclude={25-30}](@/reference/latest/src/main/java/com/example/docs/event/SheepShearCallback.java)

We can add Javadoc comments to the top of callback classes to document what each `ActionResult` does. In our case, it might be:

@[code lang=java transclude={9-16}](@/reference/latest/src/main/java/com/example/docs/event/SheepShearCallback.java)

### Mixin에서 이벤트 호출하기 {#triggering-the-event-from-a-mixin}

We now have the basic event skeleton, but we need to trigger it. Because we want to have the event called when a player attempts to shear a sheep, we call the event `invoker` in `SheepEntity#interactMob` when `sheared()` is called (i.e. sheep can be sheared, and the player is holding shears):

@[code lang=java transcludeWith=:::](@/reference/latest/src/main/java/com/example/docs/mixin/event/SheepEntityMixin.java)

### 테스트 구현 만들기 {#creating-a-test-implementation}

Now we need to test our event. You can register a listener in your initialization method (or another area, if you prefer) and add custom logic there. Here's an example that drops a diamond instead of wool at the sheep's feet:

@[code lang=java transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/event/ExampleModEvents.java)

이제 게임에서 가위로 양털을 깎으면, 양털 대신 다이아몬드가 떨어질 겁니다.
