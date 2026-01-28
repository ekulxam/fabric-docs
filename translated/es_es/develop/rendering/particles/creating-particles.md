---
title: Creando Partículas Personalizadas
description: Aprende a crear partículas personalizadas usando el Fabric API.
authors:
  - Superkat32
---

Las partículas son una poderosa herramienta. Pueden agregar atmósfera a una hermosa escena, o agregar tensión durante una pelea contra un jefe. ¡Agreguemos una!

## Registrar una partícula personalizada

Agregaremos una nueva partícula brillante cuyo movimiento se asemejara al de la partícula de una vara del end.

Primero tenemos que registrar un `ParticleType` (Tipo de Partícula) en nuestra clase inicializadora de mod usando nuestro _mod id_.

@[code lang=java transcludeWith=#particle_register_main](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

El "sparkle_particle" en letras minúsculas es la dirección JSON para la textura de la partícula. Crearás un archivo JSON con ese mismo nombre más adelante.

## Registración en el Cliente

Después de registrar la partícula en la entrada de `ModInitializer`, también necesitarás registrar la partícula en la entrada de `ClientModInitializer` (Inicializador del Mod de Cliente).

@[code lang=java transcludeWith=#particle_register_client](@/reference/latest/src/client/java/com/example/docs/ExampleModClient.java)

En este ejemplo, estamos registrando nuestra partícula en el lado del cliente. Después estamos dándole movimiento usando la fábrica de partículas de la vara del end. Esto significa que nuestra partícula se moverá exactamente como una partícula de vara del end.

::: tip

You can see all the particle factories by looking at all the implementations of the `ParticleFactory` interface. This is helpful if you want to use another particle's behaviour for your own particle.

- IntelliJ's hotkey: <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>B</kbd>
- Visual Studio Code's hotkey: <kbd>Ctrl</kbd>+<kbd>F12</kbd>

:::

## Creating a JSON File and Adding Textures {#creating-a-json-file-and-adding-textures}

You will need to create 2 folders in your `resources/assets/example-mod/` folder.

| Folder Path          | Explanation                                                                                          |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| `/textures/particle` | The `particle` folder will contain all the textures for all of your particles.       |
| `/particles`         | The `particles` folder will contain all of the json files for all of your particles. |

For this example, we will have only one texture in `textures/particle` called "sparkle_particle_texture.png".

Next, create a new JSON file in `particles` with the same name as the JSON path that you used when registering your ParticleType. For this example, we will need to create `sparkle_particle.json`. This file is important because it lets Minecraft know which textures our particle should use.

@[code lang=json](@/reference/latest/src/main/resources/assets/example-mod/particles/sparkle_particle.json)

::: tip

You can add more textures to the `textures` array to create a particle animation. The particle will cycle through the textures in the array, starting with the first texture.

:::

## Testing the New Particle {#testing-the-new-particle}

Once you have completed the JSON file and saved your work, you are good to load up Minecraft and test everything out!

You can see if everything has worked by typing the following command:

```mcfunction
/particle example-mod:sparkle_particle ~ ~1 ~
```

![Showcase of the particle](/assets/develop/rendering/particles/sparkle-particle-showcase.png)

::: info

The particle will spawn inside the player with this command. You will likely need to walk backwards to actually see it.

:::

Alternatively, you can also use a command block to summon the particle with the exact same command.
