# 介绍

Java Shell工具（JShell）是一个用于学习Java编程语言和Java代码原型的交互式工具。 JShell是一个Read-Evaluate(评估)-Print循环（REPL），它在输入时评估声明，语句和表达式，并立即显示结果。 该工具从命令行运行.

## 为什么要使用JShell？

使用JShell，您可以一次输入一个程序元素，立即查看结果，并根据需要进行调整.

Java程序开发通常涉及以下过程:

- 写一个完整的程序。
- 编译它并修复错误。
- 运行程序
- 弄清楚它有什么问题
- 然后再编辑它
- 重复以上过程

JShell可帮助您在开发程序时尝试代码并轻松探索选项。 您可以测试单个语句，尝试不同的方法变体，并在JShell会话中试验不熟悉的API。 JShell并不能替换IDE。 在开发程序时，将代码粘贴到JShell中进行试用，然后将JShell中的工作代码粘贴到程序编辑器或IDE中.

## 启动和停止JShell

JShell是在JDK 9中引入的。要启动JShell，请在命令行中输入jshell命令。

必须在系统上安装JDK 9或更高版本。 如果您的路径不包含bin目录，例如java-home/jdk-9/bin，则从该目录中启动该工具。

以下示例显示了JShell的命令和响应:

```shell
% jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

jshell>
```

本教程中的示例使用详细模式。 在学习本教程时，建议使用详细模式，以便您看到的内容与示例相匹配。 当您更熟悉该工具时，您可能更喜欢以正常模式或更简洁模式运行。

要以详细模式启动JShell，请使用-v选项：

```shell
% jshell -v
```

要退出JShell，请输入/ exit：

```shell
jshell> /exit
|  Goodbye
```

