---
title: Stany bloku
description: Dowiedz się, dlaczego stany są idealnym sposobem na dodanie funkcji wizualnych do twoich bloków.
authors:
  - IMB11
---

Stan bloku to fragment danych dołączonych do danego bloku w świecie Minecrafta zawierający informacje o jego własnościach. Parę przykładów własności, które vanilla trzyma w stanach:

- Obrót: Głównie używane przy kłodach drzew i innnych naturalnych blokach.
- Aktywacja: W większości używane przy urządzeniach redstonowych, ale także przy blokach takich jak piec czy wędzarka.
- Wiek: Używany w uprawach, roślinach, sadzonkach, wodorostach itp.

Widzisz już pewnie, dlaczego są potrzebne - nie trzeba przez to trzymać ich danych w NBT w bycie bloku (block entity) - obniża to rozmiar świata i zmniejsza problemy z wydajnością!

Blockstate definitions are found in the `assets/example-mod/blockstates` folder.

## Przykład: Blok filaru {#pillar-block}

<!-- Note: This example could be used for a custom recipe types guide, a condensor machine block with a custom "Condensing" recipe? -->

Minecraft ma już niektóre specjalne klasy pozwalające ci szybko stworzyć bloki różnego typu - ten przykład opiera się na tworzeniu bloku z własnością `axis` - oś - tworząc "Skondensowaną kłodę dębową".

The vanilla `RotatedPillarBlock` class allows the block to be placed in the X, Y or Z axis.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

Filary mają dwie tekstury, górną i boczną - używają modelu `block/cube_column`.

As always, with all block textures, the texture files can be found in `assets/example-mod/textures/block`

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_0_large.png" downloadURL="/assets/develop/blocks/condensed_oak_log_textures.zip">Tekstury</DownloadEntry>

Ponieważ filar ma dwie pozycje — pionową i poziomą, musimy utworzyć dwa osobne pliki modelu:

- `condensed_oak_log_horizontal.json`, który rozszerza model `block/cube_column_horizontal`.
- `condensed_oak_log.json`, który rozszerza model `block/cube_column`.

Przykład pliku `condensed_oak_log_horizontal.json`:

@[code](@/reference/latest/src/main/generated/assets/example-mod/models/block/condensed_oak_log_horizontal.json)

::: info

Remember, blockstate files can be found in the `assets/example-mod/blockstates` folder, the name of the blockstate file should match the block ID used when registering your block in the `ModBlocks` class. For instance, if the block ID is `condensed_oak_log`, the file should be named `condensed_oak_log.json`.

Więcej modyfikatorów znajdziesz na stronie [Minecraft Wiki - Models (Block States)](https://minecraft.wiki/w/Tutorials/Models#Block_states) (ang.)

:::

Następnie musimy stworzyć plik stanu bloku ⁠- to tam dzieje się cała magia. Bloki filarów mają trzy osie, więc użyjemy specyficznych modeli dla tych sytuacji:

- `axis=x` - Kiedy blok jest postawiony wzdłuż osi X, obrócimy model, aby wskazywał w kierunku dodatnich X.
- `axis=y` - Kiedy blok jest postawiony wzdłuż osi Y, użyjemy zwykłego pionowego modelu.
- `axis=z` - Kiedy blok jest postawiony wzdłuż osi Z, obrócimy model, aby wskazywał w kierunku dodatnich Z.

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/condensed_oak_log.json)

Jak zawsze, musisz stworzyć tłumaczenie dla twojego bloku oraz model przedmiotu nadrzędny do któregokolwiek z tych dwóch.

![Przykład bloku filaru w grze](/assets/develop/blocks/blockstates_1.png)

## Niestandardowe stany bloku {#custom-block-states}

Niestandardowe stany bloku są idealne do użycia w sytuacji, gdzie twój blok ma unikatowe własności - czasami okazuje się, że twój blok może wykorzystywać własności z vanilli.

Ten przykład tworzy unikatową własność booleanową zwaną `activated` - gdy gracz klika blok prawym przyciskiem myszy, stan `activated` zmieni się z `false` na `true` i zmieni swoją teksturę.

### Tworzenie właściwości {#creating-the-property}

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.create` method.

@[code transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Next, we have to append the property to the blockstate manager in the `createBlockStateDefinition` method. Musisz nadpisać metodę, by mieć dostęp do budowniczego:

@[code transcludeWith=:::2](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

Musisz także ustalić domyślny stan własności `activated` w konstruktorze swojego bloku.

@[code transcludeWith=:::3](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Wykorzystywanie własności {#using-the-property}

Ten przykład przełącza własność `activated` gdy gracz wchodzi w interakcję z blokiem. We can override the `useWithoutItem` method for this:

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

### Wizualizacja własności {#visualizing-the-property}

Zanim stworzysz plik stanu bloku, musisz podać tekstury dla aktywnego i nieaktywnego stanu bloku oraz model bloku.

<DownloadEntry visualURL="/assets/develop/blocks/blockstates_2_large.png" downloadURL="/assets/develop/blocks/prismarine_lamp_textures.zip">Tekstury</DownloadEntry>

Użyj swojej znajomości modeli bloków, by stworzyć dwa modele dla bloku: jeden dla stanu włączonego i jeden dla stanu wyłączonego. Dopiero potem możesz rozpocząć pracę nad plikiem stanu bloku.

Skoro stworzyliśmy nową własność, musimy aktualizować plik stanu bloku, by ją uwzględnić.

Jeżeli masz kilka własności w jednym bloku, musisz uwzględnić je wszystkie. Na przykład, `activated` i `axis` razem tworzą 6 kombinacji (dla dwóch możliwych stanów `activated` i trzech możliwych stanów `axis`).

Skoro ten blok ma tylko jedną własność (`activated`), JSON stanu bloku będzie wyglądał mniej więcej tak:

@[code](@/reference/latest/src/main/generated/assets/example-mod/blockstates/prismarine_lamp.json)

::: tip

Don't forget to add a [Client Item](../items/first-item#creating-the-client-item) for the block so that it will show in the inventory!

:::

Skoro przykładowym blogiem jest latarnia, musimy jeszcze sprawić, by emitowała światło, gdy własność `activated` przyjmuje wartość `true`. Można to zrobić za pomocą ustawień bloku przekazywanych konstruktorowi podczas rejestrowania bloku.

You can use the `lightLevel` method to set the light level emitted by the block, we can create a static method in the `PrismarineLampBlock` class to return the light level based on the `activated` property, and pass it as a method reference to the `lightLevel` method:

@[code transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/block/custom/PrismarineLampBlock.java)

@[code transcludeWith=:::4](@/reference/latest/src/main/java/com/example/docs/block/ModBlocks.java)

<!-- Note: This block can be a great starter for a redstone block interactivity page, maybe triggering the blockstate based on redstone input? -->

Końcowy wynik powinien wyglądać mniej więcej tak:

<VideoPlayer src="/assets/develop/blocks/blockstates_3.webm">Blok pryzmarynowej lampy w grze</VideoPlayer>
