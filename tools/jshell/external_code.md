# 外部代码

JShell会话从classpath访问外部类。 可通过模块路径，附加模块设置和模块导出设置访问外部模块。

## 设置class path

您可以使用可通过JShell会话中的classpath访问外部代码。

在命令行上设置class path，如以下示例所示：

```shell
 % jshell --class-path myOwnClassPath
```

将类路径指向包含要访问的包的目录或JAR文件。 代码必须编译成.class文件。 无法从JShell访问默认包中的代码（也称为未命名的包）。 设置类路径后，可以将这些包导入到会话中：

```shell
jshell> import my.cool.code.*
```

您还可以使用/env命令设置class path，如以下示例所示：

```shell
jshell> /env --class-path myOwnClassPath
|  Setting new options and restoring state.
```

/env命令重置执行状态，使用新的class path设置或使用该命令输入的其他env设置重新加载任何当前片段。

## 设置模块选项

JShell支持模块。 可以设置模块路径，指定要解析的其他模块以及给定的模块导出。

可以在/env命令的选项或命令行中提供模块选项，如以下示例所示：

```shell
 % jshell --module-path myOwnModulePath  --add-modules my.module
```

要查看当前环境设置，请使用/env不带选项。 以下示例包括在设置class path中设置的class path信息：

```shell
jshell> /env
 |     --add-modules my.module
 |     --module-path myOwnModulePath
 |     --class-path myOwnClassPath
```

有关选项的详细信息，请输入以下命令：

```shell
jshell> /help context
```

