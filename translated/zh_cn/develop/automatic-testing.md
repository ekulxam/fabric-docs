---
title: 自动化测试
description: 使用 Fabric Loader JUnit 写自动化测试的指南。
authors:
  - kevinthegreat1
---

此页面解释了如何在你的模组中编写自动化测试。 有两种对你的mod进行自动化测试的方法：使用Fabric Loader JUnit进行单元测试或使用Minecraft游戏测试框架进行游戏内测试。

单元测试用于测试你代码中的组件，比如方法和工具类；游戏内测试则启动Minecraft客户端与服务端来运行你的测试，它适用于测试功能和游玩过程。

## 单元测试 {#unit-testing}

由于Minecraft mod运行依赖于运行时字节码修改工具比如Mixin，仅仅使用JUnit一般不会生效。 这就是为什么Fabric提供了Fabric Loader JUnit，一个针对Minecraft mod单元测试的JUnit插件。

### 配置Fabric Loader JUnit {#setting-up-fabric-loader-junit}

首先，我们需要将Fabric Loader JUnit添加到开发环境。 将以下依赖添加到你的`build.gradle`：

@[code transcludeWith=:::automatic-testing:1](@/reference/build.gradle)

然后，我们需要告诉Gradle使用Fabric Loader JUnit来测试。 你可以通过将以下代码添加到`build.gradle`来做到这件事：

@[code transcludeWith=:::automatic-testing:2](@/reference/latest/build.gradle)

### 编写测试 {#writing-tests}

您已重新加载Gradle，您现在已经可以编写测试了。

这些测试的编写方式与常规 JUnit 测试相同，如果你想访问任何依赖于注册表的类（例如`ItemStack`），则需要进行一些额外的设置。 如果你对 JUnit 比较熟悉，那么你可以跳至 [设置注册表](#setting-up-registries)。

#### 设置您的第一个测试类 {#setting-up-your-first-test-class}

测试编写于`src/test/java`目录下。

一种命名约定是镜像你正在测试的类的包结构。 例如，为了测试 `src/main/java/com/example/docs/codec/BeanType.java`，您应在 `src/test/java/com/example/docs/codec/BeanTypeTest.java` 创建一个类。 注意我们是如何将`Test`加入到类名称最后的。 这还允许你轻松访问包私有的方法和字段。

另一个命名约定是有一个 `test` 包，例如 `src/test/java/com/example/docs/test/codec/BeanTypeTest.java`。 如果你使用 Java 模块，这可以避免使用相同包时可能出现的一些问题。

After creating the test class, use <kbd>⌘/CTRL</kbd>+<kbd>N</kbd> to bring up the Generate menu. 选择测试并开始输入方法名称，通常以 `test` 开头。 完成时请按下 <kbd>ENTER</kbd> 键。 有关使用 IDE 的更多提示和技巧，请参阅 [IDE 提示和技巧](./getting-started/tips-and-tricks#code-generation)。

![正在生成一个测试方法](/assets/develop/misc/automatic-testing/unit_testing_01.png)

当然，你可以手写方法签名，任何没有参数且返回类型为 void 的实例方法都将被标识为测试方法。 你应该以这样结尾：

![一个带有测试指示的空测试方法](/assets/develop/misc/automatic-testing/unit_testing_02.png)

注意侧边栏中的绿色箭头指引——您可以简单地通过点击它们来运行一个测试。 或者，你的测试将在每次构建时自动运行，包括 GitHub Actions 等 CI 构建。 如果你正在使用 GitHub Actions，请不要忘记阅读[设置 GitHub Actions](#setting-up-github-actions)。

现在，该编写您的实测代码了。 你可以使用 `org.junit.jupiter.api.Assertions` 断言条件。 检查以下测试：

@[code lang=java transcludeWith=:::automatic-testing:4](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

有关此代码实际作用的解释，请参阅 [Codecs](./codecs#registry-dispatch)。

#### 设置注册表 {#setting-up-registries}

很好，首次测试运行了！ 但是稍等，第二次测试失败了？ 在日志文件中，我们得到了以下错误之一。

<<< @/public/assets/develop/automatic-testing/crash-report.log

这是因为我们正在尝试访问注册表或依赖于注册表的类（或者，在极少数情况下，依赖于其他 Minecraft 类，如 `SharedConstants`），但 Minecraft 尚未初始化。 我们只需要初始化以下就能使注册表工作。 在您的`beforeAll`函数前简单地加入以下代码。

@[code lang=java transcludeWith=:::automatic-testing:7](@/reference/latest/src/test/java/com/example/docs/codec/BeanTypeTest.java)

### 配置 GitHub Actions {#setting-up-github-actions}

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
