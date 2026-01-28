---
title: Автоматизированное тестирование
description: Руководство по написанию автоматических тестов с помощью Fabric Loader JUnit.
authors:
  - kevinthegreat1
---

На этой странице объясняется, как написать код для автоматического тестирования частей вашего мода. Существует два способа автоматического тестирования вашего мода: модульные тесты с помощью Fabric Loader JUnit или игровые тесты с помощью фреймворка Gametest из Minecraft.

Модульные тесты следует использовать для тестирования компонентов вашего кода, таких как методы и вспомогательные классы, в то время как игровые тесты запускают реальный клиент и сервер Minecraft для запуска ваших тестов, что делает их подходящими для тестирования функций и игрового процесса.

## Модульное тестирование {#unit-testing}

Поскольку моддинг Minecraft основан на инструментах модификации байт-кода во время выполнения, таких как Mixin, простое добавление и использование JUnit обычно не работает. Вот почему Fabric предоставляет Fabric Loader JUnit — плагин JUnit, который позволяет проводить модульное тестирование в Minecraft.

### Настройка Fabric Loader JUnit {#setting-up-fabric-loader-junit}

Сначала нам необходимо добавить Fabric Loader JUnit в среду разработки. Добавьте следующее в блок зависимостей в `build.gradle`:

@[code transcludeWith=:::automatic-testing:1](@/reference/build.gradle)

Затем нам нужно указать Gradle использовать Fabric Loader JUnit для тестирования. Это можно сделать, добавив следующий код в `build.gradle`:

@[code transcludeWith=:::automatic-testing:2](@/reference/latest/build.gradle)

### Написание тестов {#writing-tests}

После перезагрузки Gradle вы будете готовы писать тесты.

Эти тесты пишутся так же, как и обычные тесты JUnit, с небольшой дополнительной настройкой, если вам нужно получить доступ к любому классу, зависящему от реестра, например `ItemStack`. Если вы знакомы с JUnit, можете перейти к разделу [Настройка реестров](#setting-up-registries).

#### Настройка вашего первого тестового класса {#setting-up-your-first-test-class}

Тесты пишутся в каталоге `src/test/java`.

Одним из правил именования является отражение структуры пакета тестируемого класса. Например, чтобы протестировать `src/main/java/com/example/docs/codec/BeanType.java`, вам нужно создать класс в `src/test/java/com/example/docs/codec/BeanTypeTest.java`. Обратите внимание, как мы добавили `Test` в конец имени класса. Это также позволяет легко получить доступ к методам и полям, закрытым для пакета.

Другое соглашение об именовании — наличие пакета `test`, например `src/test/java/com/example/docs/test/codec/BeanTypeTest.java`. Это предотвращает некоторые проблемы, которые могут возникнуть при использовании того же пакета, если вы используете модули Java.

After creating the test class, use <kbd>⌘/CTRL</kbd>+<kbd>N</kbd> to bring up the Generate menu. Выберите Test и начните вводить имя метода, обычно начинающееся с `test`. Нажмите <kbd>ENTER</kbd>, когда закончите. Дополнительные советы и рекомендации по использованию IDE см. в разделе [Советы и рекомендации по IDE](./getting-started/tips-and-tricks#code-generation).

![Создание метода тестирования](/assets/develop/misc/automatic-testing/unit_testing_01.png)

Конечно, вы можете написать сигнатуру метода вручную, и любой метод экземпляра без параметров и с типом возвращаемого значения void будет идентифицирован как тестовый метод. В итоге у вас должно получиться следующее:

![Пустой метод тестирования с тестовыми индикаторами](/assets/develop/misc/automatic-testing/unit_testing_02.png)

Обратите внимание на зеленые стрелки индикаторы в желобе: вы можете легко запустить тест, щелкнув по ним. В качестве альтернативы ваши тесты будут запускаться автоматически для каждой сборки, включая сборки CI такие, как GitHub Actions. Если вы используете GitHub Actions, не забудьте прочитать [Настройка GitHub Actions](#setting-up-github-actions).

Теперь пришло время написать настоящий тестовый код. Вы можете утверждать условия, используя `org.junit.jupiter.api.Assertions`. Проверьте следующий тест:

@[code lang=java transcludeWith=:::automatic-testing:4](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

Объяснение того, что на самом деле делает этот код, см. в разделе [Кодеки](./codecs#registry-dispatch).

#### Настройка реестров {#setting-up-registries}

Отлично, первый тест сработал! Но подождите, второй тест не прошёл? В журналах мы получаем одну из следующих ошибок.

<<< @/public/assets/develop/automatic-testing/crash-report.log

Это происходит потому, что мы пытаемся получить доступ к реестру или классу, который зависит от реестра (или, в редких случаях, зависит от других классов Minecraft таких, как `SharedConstants`), но Minecraft не инициализирован. Нам нужно просто немного инициализировать его, чтобы реестры заработали. Просто добавьте следующий код в начало метода `beforeAll`.

@[code lang=java transcludeWith=:::automatic-testing:7](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

### Настройка действий GitHub {#setting-up-github-actions}

::: info

This section assumes that you are using the standard GitHub Action workflow included with the example mod and with the mod template.

:::

Your tests will now run on every build, including those by CI providers such as GitHub Actions. But what if a build fails? We need to upload the logs as an artifact so we can view the test reports.

Add this to your `.github/workflows/build.yaml` file, below the `./gradlew build` step.

```yaml
- name: Store reports
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: reports
    path: |
      **/build/reports/
      **/build/test-results/
```

## Game Tests {#game-tests}

Minecraft provides the game test framework for testing server-side features. Fabric additionally provides client game tests for testing client-side features, similar to an end-to-end test.

### Setting up Game Tests with Fabric Loom {#setting-up-game-tests-with-fabric-loom}

Both server and client game tests can be set up manually or with Fabric Loom. This guide will use Loom.

To add game tests to your mod, add the following to your `build.gradle`:

@[code transcludeWith=:::automatic-testing:game-test:1](@/reference/latest/build.gradle)

To see all available options, see [the Loom documentation on tests](./loom/fabric-api#tests).

#### Setting up Game Test Directory {#setting-up-game-test-directory}

::: info

You only need this section if you enabled `createSourceSet`, which is recommended. You can, of course, do your own Gradle magic, but you'll be on your own.

:::

If you enabled `createSourceSet` like the example above, your gametest will be in a separate source set with a separate `fabric.mod.json`. The module name defaults to `gametest`. Create a new `fabric.mod.json` in `src/gametest/resources/` as shown:

<<< @/reference/latest/src/gametest/resources/fabric.mod.json

Note that this `fabric.mod.json` expects a server game test at `src/gametest/java/com/example/docs/ExampleModGameTest`, and a client game test at `src/gametest/java/com/example/docs/ExampleModClientGameTest`.

### Writing Game Tests {#writing-game-tests}

You can now create server and client game tests in the `src/gametest/java` directory. Here is a basic example for each:

::: code-group

<<< @/reference/latest/src/gametest/java/com/example/docs/ExampleModGameTest.java [Server]

<<< @/reference/latest/src/gametest/java/com/example/docs/ExampleModClientGameTest.java [Client]

:::

See the respective Javadocs in Fabric API for more info.

### Running Game Tests {#running-game-tests}

Server game tests will be run automatically with the `build` Gradle task. You can run client game tests with the `runClientGameTest` Gradle task.

### Run Game Tests on GitHub Actions {#run-game-tests-on-github-actions}

Existing GitHub Action workflows using `build` will run server game tests automatically. To run client game tests with GitHub Actions, add the following snippet to your `build.gradle` and the following job to your workflow. The Gradle snippet will run client game tests using [Loom's production run tasks](./loom/production-run-tasks), and the job will execute the production run task in the CI.

::: warning

Currently, game test may fail on GitHub Actions due to an error in the network synchronizer. If you encounter this error, you can add `-Dfabric.client.gametest.disableNetworkSynchronizer=true` to the JVM args in your production run task declaration.

:::

@[code transcludeWith=:::automatic-testing:game-test:2](@/reference/latest/build.gradle)

@[code transcludeWith=:::automatic-testing:game-test:3](@/.github/workflows/build.yaml)
