---
title: Types de dégâts
description: Apprenez comment ajouter des types de dégâts personnalisés.
authors:
  - dicedpixels
  - hiisuuii
  - MattiDragon
---

Les types de dégâts définissent les types de dégâts que les entités peuvent prendre. Depuis Minecraft 1.19.4, la création d'un nouveau type de dégât est "data-driven", ce qui signifie qu'ils sont créés avec des fichiers JSON.

## Creating a Damage Type {#creating-a-damage-type}

Let's create a custom damage type called _Tater_. We start by creating a JSON file for your custom damage. This file
will
be placed in your mod's `data` directory, in a subdirectory named `damage_type`.

```text:no-line-numbers
resources/data/example-mod/damage_type/tater.json
```

It has the following structure:

@[code lang=json](@/reference/latest/src/main/generated/data/example-mod/damage_type/tater.json)

This custom damage type causes 0.1 increase
in [hunger exhaustion](https://minecraft.wiki/w/Hunger#Exhaustion_level_increase) each time a player takes damage, when
the damage is caused by a living, non-player source (e.g., a block). En outre, le montant des dégâts infligés s'adapte à la difficulté du monde

::: info

Référez-vous au [Wiki Minecraft](https://fr.minecraft.wiki/w/Type_de_d%C3%A9g%C3%A2ts#Format_JSON) pour connaître toutes les clés et valeurs possibles.

:::

### Accessing Damage Types Through Code {#accessing-damage-types-through-code}

When we need to access our custom damage type through code, we will use it's `ResourceKey` to build an instance
of `DamageSource`.

The `ResourceKey` can be obtained as follows:

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/damage/ExampleModDamageTypes.java)

### Using Damage Types {#using-damage-types}

To demonstrate the use of custom damage types, we will use a custom block called _Tater Block_. Let's make is so that
when a living entity steps on a _Tater Block_, it deals _Tater_ damage.

You can override `stepOn` to inflict this damage.

We start by creating a `DamageSource` of our custom damage type.

@[code lang=java transclude={22-26}](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

Then, we call `entity.damage()` with our `DamageSource` and an amount.

@[code lang=java transclude={27-27}](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

The complete block implementation:

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/damage/TaterBlock.java)

Now whenever a living entity steps on our custom block, it'll take 5 damage (2.5 hearts) using our custom damage type.

### Custom Death Message {#custom-death-message}

You can define a death message for the damage type in the format of `death.attack.message_id` in our
mod's `en_us.json` file.

```json
{
  "death.attack.tater": "%1$s died from Tater damage!"
}
```

Upon death from our damage type, you'll see the following death message:

![Effect in player inventory](/assets/develop/tater-damage-death.png)

### Damage Type Tags {#damage-type-tags}

Some damage types can bypass armor, bypass status effects, and such. Tags are used to control these kinds of properties
of damage types.

You can find existing damage type tags in `data/minecraft/tags/damage_type`.

::: info

Refer to the [Minecraft Wiki](https://minecraft.wiki/w/Tag#Damage_types) for a comprehensive list of damage type
tags.

:::

Let's add our Tater damage type to the `bypasses_armor` damage type tag.

To add our damage type to one of these tags, we create a JSON file under the `minecraft` namespace.

```text:no-line-numbers
data/minecraft/tags/damage_type/bypasses_armor.json
```

With the following content:

@[code lang=json](@/reference/latest/src/main/generated/data/minecraft/tags/damage_type/bypasses_armor.json)

Ensure your tag does not replace the existing tag by setting the `replace` key to `false`.
