---
title: Предметы еды
description: Узнайте, как добавить FoodComponent к предмету, чтобы сделать его съедобным, и как его настроить.
authors:
  - IMB11
---

Еда — это ключевой аспект выживания в Minecraft, поэтому при создании съедобных предметов вам следует учитывать их использование с другими съедобными предметами.

Если вы не создаете мод с очень мощными предметами, вам следует учесть:

- Насколько сильное чувство голода добавляет или убирает ваш съедобный продукт.
- Какой эффект(ы) зелья оно дает?
- Доступно ли оно на ранней или конечной стадии игры?

## Добавляем компонент еды {#adding-the-food-component}

To add a food component to an item, we can pass it to the `Item.Properties` instance:

```java
new Item.Properties().food(new FoodProperties.Builder().build())
```

На данный момент это просто делает продукт съедобным и ничего более.

The `FoodProperties.Builder` class has some methods that allow you to modify what happens when a player eats your item:

| Метод                | Описание                                                                                  |
| -------------------- | ----------------------------------------------------------------------------------------- |
| `nutrition`          | Устанавливает количество очков голода, которое восполнит ваш предмет.     |
| `saturationModifier` | Устанавливает количество точек насыщенности, которые добавит ваш элемент. |
| `alwaysEdible`       | Позволяет съесть ваш предмет независимо от уровня голода.                 |

When you've modified the builder to your liking, you can call the `build()` method to get the `FoodProperties`.

If you want to add status effects to the player when they eat your food, you will need to add a `Consumable` component alongside the `FoodProperties` component as seen in the following example:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Подобно примеру на странице [Создание вашего первого элемента](./first-item), я буду использовать указанный выше компонент:

@[code transcludeWith=:::poisonous_apple](@/reference/latest/src/main/java/com/example/docs/item/ModItems.java)

Это делает предмет:

- Всегда съедобным, может быть съеден независимо от уровня голода.
- Всегда дающим Отравление II на 6 секунд когда съеден.
