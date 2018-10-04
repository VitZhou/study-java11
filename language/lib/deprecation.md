# Deprecation增强

Deprecation澄清了弃用意味着什么语义，包括是否可以在不久的将来删除API。

如果您是库维护者，则可以利用更新的弃用语法向库的用户通知库提供的API的状态。

如果您是库或应用程序开发人员，则可以使用jdeprscan工具查找应用程序或库中已弃用的JDK API元素的使用。

## jdk中的deprecation

deprecation是通知库的使用者应该从已弃用的API迁移代码。

在JDK中，API因各种原因而被弃用，例如：

- API很危险（例如，Thread.stop方法）。
- 有更简单的重命名(例如,AWT Component.show/hide被setVisible取代)
- 可以使用更新，更好的API。
- 已弃用的API将被删除。

在以前的版本中，API虽然被标记为已弃用，但实际上从未删除过。 从JDK 9开始，可能会将API标记为已弃用以进行删除。 这表明API有资格在JDK平台的下一版本中删除。 如果您的应用程序或库使用任何这些API，那么您应该制定一个计划很快从中迁移。

有关当前版本的JDK中已弃用的API的列表，请参阅API规范中的[弃用API](https://docs.oracle.com/javase/10/docs/api/deprecated-list.html)页面。

## 如何弃用API

弃用API需要使用两种不同的机制：@Deprecated注解和@deprecated Javadoc标记。

@Deprecated注解以一种记录在class文件中的方式标记API，并且在运行时可用。 这允许各种工具（例如javac和jdeprscan）检测和标记已弃用的API的使用。 @deprecated Javadoc标记用于不推荐使用的API的文档，例如，描述弃用的原因，以及建议替代API。

请注意大小写：注解以大写D开头，Javadoc标记以小写d开头。

### 使用@Deprecated注解

要表示弃用，请在模块，类，方法或成员声明之前加上@Deprecated。 注解包含以下元素：

- @Deprecated(since="`<version>`")
  - <version>标记API为弃用时的版本。 这是出于提供信息的目的。 默认值为空字符串（“”）。
- @Deprecated(forRemoval=`<boolean>`)
  - forRemoval = true表示API将在以后的版本中删除。
  - forRemoval = false建议代码不再使用此API; 但是，目前没有意图删除API。 这是默认值。

例如: @Deprecated(since="9", forRemoval=true)

@Deprecated注解导致Javadoc生成的文档在以下任何一个程序元素出现时标记为(只选择其中一个)：

- Deprecated
- Deprecated, for removal: This API element is subject to removal in a future version(不推荐使用，要删除：此API元素将在以后的版本中删除)。

javadoc工具生成一个名为deprecated-list.html的页面，其中包含已弃用的API列表，并在导航栏中向该页面添加一个链接。

以下是使用java.lang.Thread类中的@Deprecated注释的简单示例：

```java
public class Thread implements Runnable {
	... 
	@Deprecated(since="1.2")
	public final void stop() {
		...
	}
	...
```

### Deprcation语义

@Deprecated注解的两个元素使开发人员有机会阐明弃用对其导出的API意味着什么。

对于JDK平台：
@Deprecated（forRemoval = true）表示API将在JDK平台的未来版本中删除。

@Deprecated（since =“<version>”）包含JDK版本字符串，该字符串指示不推荐使用API元素的时间，以及JDK 9及更高版本中不推荐使用的那些元素。

如果您维护库并生成自己的API，那么您可能使用@Deprecated注释。 您应该围绕API删除确定并传达您的策略。 例如，如果您每6周发布一个新库，那么您可以选择弃用API以进行删除，但不能立刻删除,需要留出几个月时间以便为您的客户提供迁移时间。

### 使用@deprecated Javadoc标记

在任何已弃用的程序元素的javadoc注释中使用@deprecated标记，以指示不应再使用它（即使它可能继续工作）。 此标记在所有类，方法或字段文档注释中有效。 @deprecated标记后面必须跟一个空格或换行符。 在@deprecated标记后面的段落中，解释该项目被弃用的原因，并建议使用什么。 使用@link标记标记引用相同功能的新版本的文本。

当遇到@deprecated标记时，javadoc工具会将@deprecated标记后面的文本移动到描述的前面，并在其前面加上警告。 例如，这个来源：

```java
/**
   * ...
   * @deprecated  This method does not properly convert bytes into
   * characters.  As of JDK 1.1, the preferred way to do this is via the
   * {@code String} constructors that take a {@link
   * java.nio.charset.Charset}, charset name, or that use the platform's
   * default charset.
   * ...
   */
   @Deprecated(since="1.1")
   public String(byte ascii[], int hibyte) {
      ...
```

生成以下输出：

```java
@Deprecated(since="1.1")
public String(byte[] ascii,  int hibyte)
Deprecated. This method does not properly convert bytes into characters. As of 
JDK 1.1, the preferred way to do this is via the String constructors that take a 
Charset, charset name, or that use the platform's default charset.
```

如果使用@deprecated Javadoc标记而没有相应的@Deprecated注释，则会生成警告.

## 通知和警告

不推荐使用API时，必须通知开发人员。 不推荐使用的API可能会导致代码出现问题，或者，如果最终将其删除，则会导致运行时出现故障。

Java编译器生成有关已弃用API的警告。 有一些选项可以生成有关警告的更多信息，您还可以禁止弃用警告。

### 编译器弃用警告

如果弃用是forRemoval = false，则Java编译器会生成“普通弃用警告”。 如果弃用是forRemoval = true，则编译器会生成“删除警告”。

这两种警告由单独的-Xlint标志控制：-Xlint：deprecation和-Xlint：removal。 默认情况下启用javac -Xlint:removal，因此会显示删除警告。

警告也可以独立关闭（注意“ - ”）： - Xlint：-deprecation和-Xlint：-removal。

这是普通弃用警告的示例。

```shell
$ javac src/example/DeprecationExample.java	
Note: src/example/DeprecationExample.java uses or overrides a deprecated API.	
Note: Recompile with -Xlint:deprecation for details.	
```

使用javac -Xlint：deprecation选项查看不推荐使用的API。

```shell
$ javac -Xlint:deprecation src/example/DeprecationExample.java	
src/example/DeprecationExample.java:12: warning: [deprecation] getSelectedValues() in JList has been deprecated	
   Object[] values = jlist.getSelectedValues();	
                     ^	
1 warning
```

以下是删除警告的示例。

```java
public class RemovalExample {
    public static void main(String[] args) {
        System.runFinalizersOnExit(true);
    }
}
$ javac RemovalExample.java
RemovalExample.java:3: warning: [removal] runFinalizersOnExit(boolean) in System 
has been deprecated and marked for removal
        System.runFinalizersOnExit(true);
              ^
1 warning
==========
```

### 抑制弃用警告

javac -Xlint选项控制在特定javac运行中编译的所有文件的警告。 您可能已在源代码中确定了生成警告的特定位置，您不再希望看到这些位置。 您可以使用@SuppressWarnings注解在编译代码时禁止显示警告。 将@SuppressWarnings注解放在使用不推荐使用的API的类，方法，字段或局部变量的声明中。

`@SuppressWarnings` 选项有:

- @SuppressWarnings("deprecation") — 仅抑制普通弃用警告
- @SuppressWarnings("removal") - 仅一致删除警告
- @SuppressWarnings({"deprecation","removal"}) - 禁止这两种类型的警告。

这是一个抑制警告的例子。

```java
 @SuppressWarnings("deprecation")
 Object[] values = jlist.getSelectedValues();
```

使用@SuppressWarnings注解，即使在命令行上启用了警告，也不会为此行发出警告。

## 运行jdeprscan

jdeprscan是一个静态分析工具，它报告应用程序对已弃用的JDK API元素的使用。运行jdeprscan以帮助识别已编译的类文件或jar文件中的可能问题。

您可以从编译器通知中找到已弃用的JDK API。但是，如果您不对每个JDK版本重新编译，或者警告被抑制，或者您依赖于作为二进制工件分发的第三方库，那么您应该运行jdeprscan。

在从JDK中删除API之前，发现已弃用API的依赖性非常重要。如果二进制文件使用在当前JDK版本中不推荐且未来会删除的API，并且您不重新编译，那么您将不会收到任何通知。在将来的JDK版本中删除API时，二进制文件将在运行时失败。 jdeprscan允许您在删除API之前立即检测此类使用情况。

有关如何运行该工具以及如何解释输出的完整语法，请参考Java平台标准版工具参考中的[jdeprscan](https://www.oracle.com/pls/topic/lookup?ctx=en/java/javase/11/core&id=JSWOR-GUID-2B7588B0-92DB-4A88-88D4-24D183660A62)。