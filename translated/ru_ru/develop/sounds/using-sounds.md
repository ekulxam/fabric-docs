---
title: Воспроизведение SoundEvents
description: Научитесь воспроизводить звуковые события.
authors:
  - JR1811
---

Minecraft имеет большой выбор звуков которые вы можете воспроизвести. Просмотрите класс `SoundEvents`, чтобы увидеть все ванильные звуки события, предоставленные Mojang.

## Воспроизведение звуков в вашем моде {#using-sounds}

Обязательно вызовите метод `playSound()` на логической стороне сервера когда воспроизводятся звуки!

In this example, the `interactLivingEntity()` and `useOn()` methods for a custom interactive item are used to play a "placing copper block" and a pillager sound.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/item/custom/CustomSoundItem.java)

Метод`playSound()` используется вместе с объектом `LivingEntity`. Необходимо указать только SoundEvent, громкость и высоту тона. You can also use the `playSound()` method from the level instance to have a higher level of control.

@[code lang=java transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/item/custom/CustomSoundItem.java)

### SoundEvent и SoundCategory {#soundevent-and-soundcategory}

SoundEvent определяет, какой звук будет воспроизводиться. Вы также можете [зарегистрировать собственные SoundEvents](./custom), чтобы включить собственный звук.

В настройках Minecraft есть несколько ползунков звука. Перечисление `SoundCategory` используется для определения того, какой ползунок будет регулировать громкость звука.

### Громкость и высота {#volume-and-pitch}

Параметр громкости может немного вводить в заблуждение. В диапазоне «0,0f - 1,0f» можно изменять фактическую громкость звука. Если число превысит это значение, будет использоваться громкость `1.0f`, и будет регулироваться только расстояние, на котором слышен ваш звук. Расстояние между блоками можно приблизительно рассчитать по формуле `звук * 16`.

Параметр высоты тона увеличивает или уменьшает значение высоты тона, а также изменяет длительность звука. В диапазоне «(0,5f - 1,0f)» высота тона и скорость уменьшаются, тогда как большие числа увеличивают высоту тона и скорость. Числа ниже «0,5f» сохранят значение высоты тона «0,5f».
