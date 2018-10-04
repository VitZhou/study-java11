# 片段

JShell接受Java语句; 变量，方法和类定义;imports; 和表达试。 这些Java代码称为片段。

## 试用片段

将Java代码片段输入JShell并立即进行评估。 显示有关结果，执行的操作以及发生的任何错误的反馈。 使用本节中的示例尝试JShell。

使用verbose(详细)选项启动JShell以获得最大可用反馈量：

```shell
% jshell -v
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro
```

在提示符处输入以下示例语句，并查看显示的输出：

```shell
jshell> int x = 45
x ==> 45
|  created variable x : int
```

首先，显示结果。 将其读作：变量x的值为45.因为您处于详细模式，所以还会显示所发生情况的描述。 信息性消息以垂直条|开始。 请注意，显示了创建的变量的名称和类型。

> 如果未输入分号，则会自动将终止分号添加到完整代码段的末尾。

当输入的表达式没有命名变量时，会创建一个临时变量，以便稍后可以引用该值。 以下示例显示表达式和方法结果的临时值。 该示例还显示了在代码段需要多行输入完成时使用的延续提示（...>）：

```java
jshell> 2 + 2
$3 ==> 4
|  created scratch variable $3 : int

jshell> String twice(String s) {
   ...>    return s + s;
   ...> }
|  created method twice(String)

jshell> twice("Ocean")
$5 ==> "OceanOcean"
|  created scratch variable $5 : String

```

## 改变定义

在试验代码时，您可能会发现变量，方法或类的定义没有按照您希望的方式执行。 通过输入新的定义可以轻松更改定义，该定义将覆盖先前的定义。

要更改变量，方法或类的定义，只需输入新定义即可。 例如，Trying Out Snippets中定义的两次方法在以下示例中获取新定义：

```shell
jshell> String twice(String s) {
   ...>    return "Twice:" + s;
   ...> }
|  modified method twice(String)

jshell> twice("thing")
$7 ==> "Twice:thing"
|  created scratch variable $7 : String
```

请注意，不是像以前一样显示创建的方法，反馈显示修改后的方法。 此消息表示定义已更改，但该方法具有相同的签名，因此所有现有用法仍然有效。

您还可以以不兼容的方式更改定义。 以下示例显示x正在从int更改为String：

```shell
jshell> int x = 45
x ==> 45
|  created variable x : int

jshell> String x
x ==> null
|  replaced variable x : String
|    update overwrote variable x : int
```

变量x的类型已更改，现在反馈显示已replaced.

### 改变反馈级别

JShell是在详细的反馈模式下启动的，它提供了很多评估。 您可以使用/ set feedback命令设置输出的数量和格式，例如/set feedback concise。 如果您主要通过从其他窗口粘贴来使用JShell，那么您可能更喜欢没有提示且只有错误反馈的反馈模式。 如果是，则输入/set feedback silent命令。

### Forward References

JShell接受引用尚未定义的方法，变量或类的方法定义。 这样做是为了支持探索性编程，因为某些形式的编程需要它。

例如，如果要为球体的体积定义方法，则可以输入以下公式作为方法体积：

```shell
jshell> double volume(double radius) {
   ...>    return 4.0 / 3.0 * PI * cube(radius);
   ...> }
|  created method volume(double), however, it cannot be invoked until variable PI, and method cube(double) are declared
```

JShell允许定义，但警告尚未定义的内容。 可以引用该定义，但是如果尝试执行，则在所有必需元素定义之前失败：

```shell
jshell> double PI = 3.1415926535
PI ==> 3.1415926535
|  created variable PI : double

jshell> volume(2)
|  attempted to call method volume(double) which cannot be invoked until method cube(double) is declared

jshell> double cube(double x) { return x * x * x; }
|  created method cube(double)
|    update modified method volume(double)

jshell> volume(2)
$5 ==> 33.510321637333334
|  created scratch variable $5 : double
```

有了所有定义，现在可以使用valume方法。

此方法现在用于说明有关不兼容替换的更多信息。 例如，要更改PI的精度，请输入新值，如以下示例所示：

```shell
jshell> BigDecimal PI = new BigDecimal("3.141592653589793238462643383")
PI ==> 3.141592653589793238462643383
|  replaced variable PI : BigDecimal
|    update modified method volume(double) which cannot be invoked until this error is corrected: 
|      bad operand types for binary operator '*'
|        first type:  double
|        second type: java.math.BigDecimal
|          return 4.0 / 3.0 * PI * cube(radius);
|                 ^------------^
|    update overwrote variable PI : double
```

PI的新定义与volume()的定义类型不兼容。 由于您处于详细模式，因此会显示受更改影响的其他定义的更新信息，在本例中描述了不兼容性。 请注意，详细模式是唯一显示更新信息的预定义反馈模式。 在其他反馈模式中，在执行代码之前不会显示警告。 这样做的目的是防止更新过载。 在所有预定义模式下，执行volume()方法会显示问题：

```shell
jshell> volume(2)
|  attempted to call method volume(double) which cannot be invoked until this error is corrected: 
|      bad operand types for binary operator '*'
|        first type:  double
|        second type: java.math.BigDecimal
|          return 4.0 / 3.0 * PI * cube(radius);
|                 ^------------^
```

### 异常

在异常回溯中，反馈会识别代码段以及发生异常的代码段内的位置。

输入JShell的代码中的位置显示为#ID：line-number，其中代码段ID是/list命令显示的数字，line-number是代码段中的行号。 在下面的示例中，异常发生在方法的第二行的代码段1中，它是divide()方法：

```shell
jshell> int divide(int x, int y) {
   ...> return x / y;
   ...> }
|  created method divide(int,int)

jshell> divide(5, 0)
|  java.lang.ArithmeticException thrown: / by zero
|        at divide (#1:2)
|        at (#2:1)
                             
jshell> /list
                                                            
   1 : int divide(int x, int y) {
           return x / y;
       }
   2 : divide(5, 0)
```

## tap键自动补全片段

输入片段时，使用Tab键自动完成项目。 如果无法根据输入的内容确定项目，则提供可能的选项。

例如，如果您从Forward References输入了volume方法，则可以输入方法名称的前几个字母，然后按Tab键完成输入：

```shell
jshell> vol<Tab>
```

输入更改为以下内容：

```shell
jshell> volume(
```

如果有多种可能性，则会显示一组可能性：

```
jshell> System.c<Tab>
class                 clearProperty(        console()             currentTimeMillis()

jshell> System.c
```

当您在方法调用的左括号中时，按Tab键会显示参数类型的完成可能性：

```shell
jshell> "hello".startsWith(<Tab>
Signatures:
boolean String.startsWith(String prefix, int toffset)
boolean String.startsWith(String prefix)

<press tab again to see documentation>

jshell> "hello".startsWith(
```

再次按Tab键会显示第一个方法的纯文本版本文档。

```shell
jshell> "hello".startsWith(<Tab>
boolean String.startsWith(String prefix, int toffset)
Tests if the substring of this string beginning at the specified index starts with the
specified prefix.

Parameters:
prefix - the prefix.
toffset - where to begin looking in this string.

Returns:
true if the character sequence represented by the argument is a prefix of the substring of this
          object starting at index toffset ; false otherwise. The result is false if toffset is
          negative or greater than the length of this String object; otherwise the result is
          the same as the result of the expression
                    this.substring(toffset).startsWith(prefix)
                    

<press tab to see next documentation>

jshell> "hello".startsWith(
```

## 片段转换

JShell可以在首次引用时轻松导入所需的类，并使用键盘快捷键将表达式转换为变量声明。

当您输入尚未导入的标识符时，请在标识符后面立即按Shift + Tab i以查看允许您将导入添加到会话的选项：

```shell
jshell> new JFrame<Shift+Tab i>
0: Do nothing
1: import: javax.swing.JFrame
Choice: 1
Imported: javax.swing.JFrame

jshell> new JFrame
```

输入所需选项的编号。 可以提供多个导入选项。

输入表达式后，按Shift + Tab v可以将表达式转换为变量声明。 表达式成为变量声明的初始值，表达式的类型成为变量的类型。 按Shift + Tab v后，光标（由示例中的竖线（|）表示）将放置在您需要输入变量名称的行中：

```shell
jshell> new JFrame("Demo") <Shift+Tab v>
jshell> JFrame | = new JFrame("Demo")
```

> 表达式必须有效或忽略转换请求。 在该示例中，在变量转换之前需要导入JFrame。

要完成上一个示例，请在光标处输入变量名称frame，然后按Enter键：

```java
jshell> JFrame frame = new JFrame("Demo")
frame ==> javax.swing.JFrame[frame0,0,0,0x0,invalid,hidden, ... tPaneCheckingEnabled=true]
|  created variable frame : JFrame

jshell>
```

有时表达式的结果类型尚未导入。 在这种情况下，Shift + Tab v提供导入和创建变量，如以下示例所示：

```java
jshell> frame.getGraphics() <Shift+Tab v>
0: Do nothing
1: Create variable
2: import: java.awt.Graphics. Create variable
Choice: 2
Imported: java.awt.Graphics

jshell> Graphics | = frame.getGraphics()
```

要完成上一个示例，请在光标处输入变量名称gc，然后按Enter键：

```java
jshell> Graphics gc = frame.getGraphics()
|  created variable gc : JFrame

jshell>
```

