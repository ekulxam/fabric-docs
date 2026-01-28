---
title: Crear Sonidos Personalizados
description: Aprende a agregar y usar un nuevo sonido con el registro.
authors:
  - JR1811
---

## Preparando el archivo del sonido

Tus archivos de sonido tienen que estar formateados de una manera específica. OGG Vorbis es un formato de contenedor abierto para datos multimedia, como audio, y es usado para los archivos de sonido de Minecraft. Para evitar problemas con la manera en que Minecraft maneja el distanciamiento, tu audio tiene que tener solo un canal (Mono).

Muchos software de DAWs (Estaciones de Trabajo de Audio Digitales) modernos pueden importar y exportar usando este formato de archivo. En el siguiente ejemplo el software gratuito y de fuente abierta "[Audacity](https://www.audacityteam.org/)" será usado para traer el archivo de sonido al formato correcto, aunque cualquier otro DAW debería ser suficiente también.

![archivo de sonido sin preparar en Audacity](/assets/develop/sounds/custom_sounds_0.png)

En este ejemplo, un sonido de un [silbido](https://freesound.org/people/strongbot/sounds/568995/) es importado a Audacity. Actualmente está guardado como un archivo ".wav" y tiene dos canales de audio (Stereo). Edita el sonido a tu gusto y asegúrate de eliminar uno de los canales usando el menú desplegable encima del "track head" (cabeza de pista).

![eliminando uno de los canales](/assets/develop/sounds/custom_sounds_2.png)

![dividiendo la pista Estereo](/assets/develop/sounds/custom_sounds_1.png)

Al exportar o renderizar el archivo de audio, asegúrate de elegir el formato de archivo OGG. Algunos DAWs, como REAPER, pueden soportar múltiples capas de audio de OGG. En este caso, OGG Vorbis debería funcionar bien.

![exportar como archivo OGG](/assets/develop/sounds/custom_sounds_3.png)

También ten en cuenta que los archivos de audio pueden aumentar el tamaño de tu mod drásticamente. Si es necesario, comprime el audio cuando lo edites y exportes el archivo para mantener el tamaño de tu producto final a un mínimo.

## Cargando El Archivo de Audio

Add the new `resources/assets/example-mod/sounds` directory for the sounds in your mod, and put the exported audio file `metal_whistle.ogg` in there.

Continue with creating the `resources/assets/example-mod/sounds.json` file if it doesn't exist yet and add your sound to the sound entries.

@[code lang=json](@/reference/latest/src/main/resources/assets/example-mod/sounds.json)

La entrada de subtítulo provee más contexto para el jugador. The subtitle name is used in the language files in the `resources/assets/example-mod/lang` directory and will be displayed if the in-game subtitle setting is turned on and this custom sound is being played.

## Registrando el Sonido Personalizado

To add the custom sound to the mod, register a SoundEvent in your [mod's initializer](../getting-started/project-structure#entrypoints).

```java
Registry.register(BuiltInRegistries.SOUND_EVENT, Identifier.fromNamespaceAndPath(MOD_ID, "metal_whistle"),
        SoundEvent.createVariableRangeEvent(Identifier.fromNamespaceAndPath(MOD_ID, "metal_whistle")));
```

## Cleaning up the Mess {#cleaning-up-the-mess}

Dependiendo de cuantas entradas de Registro hayan, las cosas pueden enredarse rápidamente. Para evitar eso, podemos hacer uso de una nueva clase ayudante.

Agrega dos nuevos métodos a la nueva clase ayudante creada. Uno, que registre todos los sonidos y uno que es usado para inicializar esta clase en primer lugar. Después de eso, puedes agregar nuevos miembros estáticos de `SoundEvent` cómodamente como sea necesario.

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/sound/CustomSounds.java)

De esta manera, la clase implementadora de `ModInitializer` solo tiene que implementar una línea para registrar todos los SoundEvents personalizados.

@[code lang=java transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/sound/ExampleModSounds.java)

## Usando el SoundEvent Personalizado

Use la clase ayudante para acceder el SoundEvent personalizado. Check out the [Playing Sounds](./using-sounds) page to learn how to play sounds.
