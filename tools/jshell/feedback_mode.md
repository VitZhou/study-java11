# 反馈模式

反馈模式确定JShell中的提示，反馈和其他交互。 提供具有不同反馈级别的预定义模式。 可以根据需要创建自定义模式。

## 设置反馈模式

反馈模式定义了与JShell交互时使用的提示和反馈。 为方便起见，提供了预定义模式。 您可以根据需要创建自定义模式.

无法修改预定义模式，但可以将它们用作自定义模式的基础。 预定义模式按详细程度的降序排列是verbose(详细), normal(一般), concise(简洁), 和 silent(静默).

下表显示了预定义模式的差异。

| 模式    | 片段值                          | 声明 | 更新 | 命令 | 提示      |
| ------- | ------------------------------- | ---- | ---- | ---- | --------- |
| verbose | name ==> value (和描述)         | Yes  | Yes  | Yes  | \njshell> |
| normal  | name ==> value                  | Yes  | No   | Yes  | \njshell> |
| concise | name ==> value (仅在异常时显示) | No   | No   | No   | jshell>   |
| silent  | No                              | No   | No   | No   | ->        |

- Mode列指示正在描述的模式
- “值片段”列指示显示具有值的片段的内容，例如表达式，赋值和变量声明。
- 声明列指示是否为声明或方法，类，枚举，接口和注解接口提供反馈。
- “更新”列指示是否显示除当前代码段之外的更改。
- “命令”列指示命令是否提供指示成功的反馈。
- “提示”列指示使用的提示。

默认反馈模式是normal的。 通过设置命令行选项或使用/set feedback命令更改反馈模式，如以下示例所示：

```shell
jshell> /set feedback verbose 
|  Feedback mode: verbose

jshell> 2 + 2
$1 ==> 4
|  created scratch variable $1 : int

jshell> /set feedback silent 
-> 2 + 2
-> /set feedback normal 
|  Feedback mode: normal

jshell> 2 + 2
$3 ==> 4

jshell> /set feedback concise 
jshell> 2 + 2
$4 ==> 4
jshell> 
```

请注意，当设置normal或verbose时，命令反馈set显示，但concise和silent模式则不会。 另请注意，当模式设置为silent时，表达式2 + 2的反馈会跟其他三种反馈模式的形式不同。

要查看当前和可用的反馈模式，请使用/set feedback命令，不带选项。 请注意，当前模式显示为设置它的命令：

```shell
jshell> /set feedback 
|  /set feedback verbose
|  
|  Available feedback modes:
|     concise
|     normal
|     silent
|     verbose
```

## 定义一个反馈模式

通过自定义反馈模式，您可以定义要查看的提示以及要为JShell中输入的不同元素接收的反馈。

反馈模式具有以下设置：

- Prompts(提示)：Regular和continue
- Truncation(截断): 显示的最大值
- Format: 提供的反馈格式

无法更改预定义模式，但您可以轻松创建现有模式的副本，如以下示例所示：

```
jshell> /set mode mine normal -command
|  Created new feedback mode: mine
```

 新模式mine是正常模式的副本。 -command选项表示您需要命令反馈。 如果您不希望命令描述发生的操作，请使用-quiet而不是-command。

### 设置Prompts

与所有/set命令一样，使用/set prompt命令来显示当前设置：

```shell
jshell> /set prompt normal
|  /set prompt normal "\njshell> " "   ...> "
```

在为前一个示例提供的反馈中，第一个字符串是常规提示，第二个字符串是在代码段扩展到多行时使用的延续提示。 以下示例显示如何切换到新模式以尝试它：

```shell
jshell> /set prompt mine "\nmy mode: "  ".......: "

jshell> /set feedback mine
|  Feedback mode: mine

my mode: class C {
.......:    int x;
.......: }
|  created class C

my mode:
```

提示字符串可以包含％s，它将替换为下一个代码段ID。 但是，如果输入命令或代码段导致错误，则可能不会为用户在提示符处输入的值分配该ID。

所有设置都有当前会话的持续时间; 它们不会被/reset命令重置。 如果您希望将设置作为将来会话的默认设置，请使用-retain选项保留它们。 以下示例显示如何跨会话保持自定义模式：

```shell
my mode: /set mode mine -retain

my mode: /set feedback mine -retain
|  Feedback mode: mine

my mode: /exit 
|  Goodbye
% jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

my mode: 
```

### 设置Truncation

如果值太长，则在显示时会截断它们。 使用/ et truncation命令设置为值显示的最大长度。 如果未使用该命令输入任何设置，则显示当前设置。 以下示例显示从normal模式继承的设置：

```shell
my mode: /set truncation mine 
|  /set truncation mine 80 
|  /set truncation mine 1000 expression,varvalue

my mode: String big = IntStream.range(0,1200).mapToObj(n -> "" + (char) ('a' + n % 26)).collect(Collectors.joining())
big ==> "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuv ... fghijklmnopqrstuvwxyzabcd"
```

截断设置生效的条件由截断长度后输入的可选选择器确定。 定义了两种类型的选择器（在线帮助中称为选择器类型）：

- 案例选择器指示其值显示的片段类型。
- 操作选择器描述了代码段发生的情况。

输入/help/set truncation以获取有关选择器的详细信息。

上一个示例中显示的设置意味着值被截断为80个字符，除非该值是表达式（表达式大小写选择器）的值或变量的值，如通过仅输入变量名称（varvalue）明确请求的设置。

以下示例将默认截断设置为100，并且仅在显式请求时显示长值：

```shell
my mode: /set truncation mine 100

my mode: /set truncation mine 300 varvalue

my mode: big + big
$2 ==> "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghi ... yzabcdefghijklmnopqrstuvwxyzabcd"

my mode: big
big ==> "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstu
vwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijkl ... jklmnopq
rstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcd"

my mode: /set mode mine -retain
```

要保留新设置，请保留更改，如示例末尾所示。

## 设置Formats

代码段输出是您可以自定义的另一个设置。 在从normal模式继承的格式中，import不提供任何反馈，并且不显示值的类型：

```shell
my mode: import java.beans.*

my mode: Locale.CANADA.getUnicodeLocaleAttributes()
$5 ==> []
```

使用/set format命令设置片段输出的格式。 使用模式名称输入此项，无需设置即可查看当前格式设置：

```shell
my mode: /set format mine
```

/help/set format命令提供了有关此命令的帮助。 您可能希望将其用作本节其余部分的参考，其中提到了在定义format时使用的字段。

显示的主要反馈由display字段确定。 可以定义其他字段以帮助定义display字段。 silent之外的预定义模式定义了其中几个字段，如/help/set format命令的输出中所示。 这些字段在示例模式中继承。 导入的display定义如以下示例所示：

```shell
my mode: /set format mine display "{pre}added import {name}{post}" import-added

my mode: /set format mine display "{pre}re-added import {name}{post}" import-modified,replaced
```

name字段预定义为代码段的名称。 以下示例显示现在为import提供反馈：

```
my mode: import java.beans.*
|  re-added import java.beans.*
```

显示定义中使用的pre和post字段是每行反馈输出的前缀和后缀字符。 以下示例将pre更改为空字符串：

```shell
my mode: /set format mine pre ""

my mode: void m() {}
created method m()

my mode: import java.beans.*
re-added import java.beans.*

my mode: /set truncation mine 
/set truncation mine 100
/set truncation mine 300 varvalue
```

> 对前缀字符的更改会影响所有反馈，包括命令反馈。

要在display值时显示类型，请更改由预定义模式定义的result字段：

```shell
my mode: /set format mine result "{type} {name} = {value}{post}" added,modified,replaced-primary-ok

my mode: Locale.CANADA.getUnicodeLocaleAttributes()
Set<String> $11 = []

my mode: 2 + 2
int $12 = 4
```

在输入的片段（主要）上只有当它是new的或updated（added，modified，replaced），并且没有错误（ok）时，此更改才会使result非空。

要永久删除保留模式，请将-retain选项与-delete选项一起使用：

```shell
my mode: /set feedback verbose -retain
|  Feedback mode: verbose

jshell> /set mode mine -delete -retain
```

