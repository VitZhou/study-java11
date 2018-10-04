# 命令

JShell命令在JShell会话中输入，用于控制环境和显示信息。

## 命令简介

JShell命令控制环境并在session中显示信息。

命令与前导斜杠（/）的片段不同。 有关当前变量，方法和类型的信息，请使用/vars，/methods和/types命令。 有关输入的代码段列表，请使用/ ist命令。 以下示例基于Trying Out Snippets中的示例显示这些命令：

```java
jshell> /vars
|    int x = 45
|    int $3 = 4
|    String $5 = "OceanOcean"

jshell> /methods
|    twice (String)String

jshell> /list

   1 : System.out.println("Hi");
   2 : int x = 45;
   3 : 2 + 2
   4 : String twice(String s) {
         return s + s;
       }
   5 : twice("Ocean")

```

JShell有一个默认的启动脚本，它在JShell启动之前以静默方式自动执行，这样你就可以快速开始工作。 除非您使用/list -start或/list -all命令请求启动脚本中的条目，否则不会列出这些条目：

```shell
jshell> /list -all

  s1 : import java.util.*;
  s2 : import java.io.*;
  s3 : import java.math.*;
  s4 : import java.net.*;
  s5 : import java.util.concurrent.*;
  s6 : import java.util.prefs.*;
  s7 : import java.util.regex.*;
   1 : System.out.println("Hi");
   2 : int x = 45;
   3 : 2 + 2
   4 : String twice(String s) {
         return s + s;
       }
   5 : twice("Ocean")
```

默认启动脚本包含几个常见的导入。 您可以使用/set start命令个性化您的启动条目。 有关此命令的信息，请输入/help/set start。 /save -start命令将当前启动脚本保存为您自己的启动脚本的起点。

其他重要命令包括/exit以退出JShell，/save以保存您的代码段，以及/open以从文件中输入代码段。

输入/help以获取JShell命令列表。

## tap键补全命令

与片段补全类似，当您输入命令和命令选项时，使用Tab键自动补全命令或选项。 如果无法根据输入的内容确定完成，则提供可能的选择。

以下示例显示在命令的前导斜杠（/）后按Tab键时的反馈：

```shell
jshell> /<Tab>
/!          /?          /drop       /edit       /env        /exit       /help
/history    /imports    /list       /methods    /open       /reload     /reset      
/save       /set        /types      /vars       

<press tab again to see synopsis>

jshell> /
```

在输入/ l并按Tab键后，该行将替换为/ list：

```shell
jshell> /l<Tab>

jshell> /list
```

tap补全也适用于命令选项。 以下示例显示使用Tab键显示/list命令的选项：

```shell
jshell> /list -<Tab>
-all       -history   -start     

<press tab again to see synopsis>

jshell> /list -
```

请注意有关再次按Tab键以显示命令概要的消息，该命令是命令的简短描述。 再次按Tab键以显示帮助文档。 以下示例显示了第二次和第三次按Tab键的结果：

```shell
jshell> /list -<Tab>
list the source you have typed

<press tab again to see full documentation>

jshell> /list -<Tab>
Show the source of snippets, prefaced with the snippet id.

/list
    List the currently active snippets of code that you typed or read with /open

/list -start
    List the automatically evaluated start-up snippets

/list -all
    List all snippets including failed, overwritten, dropped, and start-up

/list <name>
    List snippets with the specified name (preference for active snippets)

/list <id>
    List the snippet with the specified snippet id

jshell> /list -
```

已完成唯一参数的补全。 例如，输入/ list -a <Tab>后，将自动显示-all选项：

```shell
jshell> /list -a<Tab>

jshell> /list -all
```

也可以使用Tab补全代码段名称。 例如，如果您在JShell会话中先前定义了volume方法，则在开始输入方法名称后按Tab键会导致显示完整的方法名称：

```shell
jshell> /ed v<Tab>

jshell> /ed volume
```

在命令的文件参数位置使用Tab可显示可用文件：

```shell
jshell> /open <Tab>
myfile1      myfile2    definitions.jsh

<press tab again to see synopsis>

jshell> /open 
```

补全唯一文件名：

```shell
jshell> /open d<Tab> 

jshell> /open definitions.jsh
```

## 命令缩写

使用缩写减少您必须执行的键入操作。 只要缩写是唯一的，命令，/set子命令，命令参数和命令选项都可以缩写。

以/l开头的唯一命令是/list，而以-a开头的/list的唯一选项是-all。 因此，您可以使用以下缩写输入/list -all命令：

```shell
jshell> /l -a
```

此外，以/se开头的唯一命令是/set，以fe开头的唯一/set子命令是反馈，并且以v开头的唯一反馈模式是verbose(详细模式)，假设不存在以v开头的自定义反馈模式。 因此，您可以使用以下缩写将反馈模式设置为verbose：

```shell
jshell> /se fe v
```

请注意，/s不是一个完整的缩写，因为/save和/set都以相同的字母开头。 如有疑问，您可以使用Tab完成查看选项。