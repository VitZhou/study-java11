# 脚本

JShell脚本是文件中的一系列片段和JShell命令，每行一个片段或命令。

脚本可以是本地文件，也可以是以下预定义脚本之一：

| 脚本名   | 脚本内容                                                     |
| -------- | ------------------------------------------------------------ |
| DEFAULT  | 包括常用的导入声明。 如果未提供其他启动脚本，则使用此脚本。  |
| PRINTING | 定义重定向到PrintStream中的print，println和printf方法的JShell方法。 |
| JAVASE   | 导入由java.se模块定义的核心Java SE API，由于包的数量，导致启动JShell的明显延迟。 |

## 启动脚本

启动脚本包含在启动JShell会话时加载的代码段和命令。 默认启动脚本包含常见的import语句。 您可以根据需要创建自定义脚本。

每次重置jshell工具时都会加载启动脚本。 在初始启动期间使用/reset，/reload和/env命令进行重置。 如果未设置脚本，则使用默认启动脚本DEFAULT。 此默认脚本定义了常用的导入声明。

> Java语言定义自动导入java.lang包，因此不需要显式导入此包。

要设置启动脚本，请使用/set start命令：

```shell
jshell> /set start mystartup.jsh

jshell> /reset
|  Resetting state.
```

与所有的/set命令一样，除非使用-retain选项，否则设置的持续时间是当前会话。 通常，在测试启动脚本设置时不使用-retain选项。 找到所需的设置后，使用-retain选项保留它：

```shell
jshell> /set start -retain
```

然后在下次启动jshell工具时加载启动脚本。

请记住，仅在重置状态时才将启动脚本加载到当前会话中。 存储脚本的内容，而不是对脚本的引用。 该脚本仅在运行/set start命令时读取。 但是，预定义的脚本是通过引用加载的，可以使用JDK的新版本进行更新。

也可以使用--startup命令行标志指定启动脚本：

```shell
% jshell --startup mystartup.jsh
```

对于实验，使用不需要System.out的打印方法很有用,可以使用预定义的PRINTING脚本访问print，println和printf方法。 您可以使用/set start指定多个启动脚本。 以下示例将启动设置为加载default import和printing定义：

```shell
jshell> /set start -retain DEFAULT PRINTING

jshell> /exit
|  Goodbye

% jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

jshell> println("Hello World!")
Hello World!
```

-retain标志用于将这些预定义脚本设置为jshell工具的未来会话的启动脚本。 使用/set start不带参数来查看这些启动脚本定义的内容的详细信息。

要在命令行上设置多个启动脚本，请对每个脚本使用--startup标志：

```shell
% jshell --startup DEFAULT --startup PRINTING
```

## 创建和加载脚本

### 创建脚本

脚本可以在编辑器中外部创建，也可以从JShell中输入的项目生成。 使用以下命令之一从JShell会话中的条目创建脚本：

```shell
jshell> /save mysnippets.jsh

jshell> /save -history myhistory.jsh

jshell> /save -start mystartup.jsh
```

示例中显示的第一个命令将当前活动片段保存到mysnippets.jsh。 显示的第二个命令将myhistory.jsh的所有片段和命令（包括有效和无效）的历史记录保存。 显示的最后一个命令将当前启动脚本设置的内容保存到mystartup.jsh。 提供的文件名可以是任何有效的文件路径和名称。

### 加载脚本

启动JShell会话时，可以从命令行加载脚本：

```shell
% jshell mysnippets.jsh
```

也可以使用/open命令在JShell会话中加载脚本：

```shell
jshell> /open PRINTING
```

