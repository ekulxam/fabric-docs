---
title: Tham Số Câu Lệnh
description: Học cách tạo ra câu lệnh với tham số phức tạp.
---

Hầu hết các câu lệnh đều có tham số. Nhiều khi tham số đó không bắt buộc, bạn không cần phải đưa vào câu lệnh nhưng nó vẫn chạy. Một node có thể có nhiều loại tham số, nhưng bạn nên tránh xảy ra trường hợp kiểu dữ liệu của tham số không rõ.

@[code lang=java highlight={3} transcludeWith=:::command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

In this case, after the command text `/command_with_arg`, you should type an integer. For example, if you
run `/command_with_arg 3`, you will get the feedback message:

> Called /command_with_arg with value = 3

@[code lang=java highlight={3,13} transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/command/FabricDocsReferenceCommands.java)

@[code lang=java highlight={3,13} transcludeWith=:::5](@/reference/latest/src/main/java/com/example/docs/command/FabricDocsReferenceCommands.java)

@[code lang=java highlight={3,5} transcludeWith=:::command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_command_with_two_args](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Giờ đây, bạn có thể nhập một hoặc hai số nguyên làm tham số. Nếu bạn đưa vào một số nguyên, bạn sẽ nhận được thông báo với một giá trị. Nếu là hai số nguyên, thông báo sẽ đưa ra hai giá trị.

Có thể bạn thấy việc viết hai quy trình xử lý dữ liệu giống nhau là không cần thiết. Ta có thể tạo một method sử dụng trong cả hai tham số.

@[code lang=java highlight={4,6} transcludeWith=:::command_with_common_exec](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java transcludeWith=:::execute_common](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

## Custom Argument Types {#custom-argument-types}

Nếu bạn không tìm thấy kiểu tham số bạn cần trong vanilla, bạn có thể tự tạo kiểu của riêng mình. Bạn cần tạo một class mà kế thừa interface `ArgumentType<T>`, với `T` là kiểu tham số.

Giả sử bạn cần một kiểu tham số cho ra `BlockPos` khi người dùng nhập: `{x, y, z}`

Giả sử bạn cần một kiểu tham số cho ra `BlockPos` khi người dùng nhập: `{x, y, z}`

@[code lang=java transcludeWith=:::1](@/reference/latest/src/main/java/com/example/docs/command/BlockPosArgumentType.java)

### Registering Custom Argument Types {#registering-custom-argument-types}

::: warning

You need to register the custom argument type on both the server and the client or else the command will not work!

:::

You can register your custom argument type in the `onInitialize` method of your [mod's initializer](../getting-started/project-structure#entrypoints) using the `ArgumentTypeRegistry` class:

@[code lang=java transcludeWith=:::register_custom_arg](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

### Using Custom Argument Types {#using-custom-argument-types}

We can use our custom argument type in a command - by passing an instance of it into the `.argument` method on the command builder.

@[code lang=java highlight={3} transcludeWith=:::custom_arg_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

@[code lang=java highlight={2} transcludeWith=:::execute_custom_arg_command](@/reference/latest/src/main/java/com/example/docs/command/ExampleModCommands.java)

Running the command, we can test whether or not the argument type works:

![Invalid argument](/assets/develop/commands/custom-arguments_fail.png)

![Valid argument](/assets/develop/commands/custom-arguments_valid.png)

![Command result](/assets/develop/commands/custom-arguments_result.png)
