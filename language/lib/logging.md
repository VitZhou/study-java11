# Java日志概述

包含在[java.util.logging](https://docs.oracle.com/javase/10/docs/api/java/util/logging/package-summary.html)包中的Java Logging API通过生成适合最终用户，系统管理员，现场服务工程师和软件开发团队分析的日志报告，促进客户站点的软件服务和维护。 Logging API捕获应用程序或平台中的安全性故障，配置错误，性能瓶颈和/或错误等信息。

核心软件包包括支持将纯文本或XML格式的日志记录传送到内存，输出流，控制台，文件和套接字。 此外，日志记录API能够与主机操作系统上已存在的日志记录服务进行交互。

## 控制流概述

应用程序在Logger对象上进行日志调用。 Logger对象在分层命名空间中组织，子Logger对象可以从命名空间中的父级继承一些日志属性。

应用程序在Logger对象上进行日志调用。 这些Logger对象分配LogRecord对象，这些对象传递给Handler对象以供发布。 Logger和Handler对象都可以使用日志级别对象和（可选）Filter对象来确定它们是否对特定的LogRecord对象感兴趣。 当需要在外部发布LogRecord对象时，Handler对象可以（可选）使用Formatter对象在将消息发布到I / O流之前对其进行本地化和格式化。

![](../img/java_pb_001a.png)

每个Logger对象都跟踪一组输出Handler对象。 默认情况下，所有Logger对象也将其输出发送到其父Logger。 但Logger对象也可以配置为忽略树上方的Handler对象。

一些Handler对象可以将输出定向到其他Handler对象。 例如，MemoryHandler维护LogRecord对象的内部环形缓冲区，并且在触发事件上，它通过目标处理程序发布其LogRecord对象。 在这种情况下，任何格式化都由链中的最后一个处理程序完成。

![](../img/java_pb_002a.png)

API的结构使得在禁用日志记录时对Logger API的调用可能很简单。 如果对给定的日志级别禁用日志记录，则Logger可以进行简单的比较测试并返回。 如果为给定的日志级别启用了日志记录，则在将LogRecord传递给Handler之前，Logger仍会小心地将成本降至最低。 特别是，本地化和格式化（相对昂贵）推迟到处理程序请求它们之前。 例如，MemoryHandler可以维护LogRecord对象的循环缓冲区，而无需支付格式化成本。

### 日志级别

每条日志消息都有一个关联的日志级别对象。 该级别粗略指导了日志消息的重要性和紧迫性。 日志级别对象封装整数值，值越高表示优先级越高。

Level类定义了七个标准日志级别，范围从FINEST（最低优先级，最低值）到SEVERE（最高优先级，具有最高值）。

### Loggers

如前所述，客户端代码将日志请求发送到Logger对象。 每个记录器都会跟踪它感兴趣的日志级别，并丢弃低于此级别的日志请求。

Logger对象通常是命名实体，使用点分隔名称，如java.awt。 命名空间是分层的，由LogManager管理。 命名空间通常应与Java打包命名空间对齐，但不要求完全遵循它。 例如，一个名为java.awt的Logger可能会处理java.awt包中类的日志记录请求，但它也可能处理sun.awt中的类的日志记录，这些类支持java.awt包中定义的客户端可见抽象。

除了命名的Logger对象之外，还可以创建未出现在共享命名空间中的匿名Logger对象。 请参考[安全性](https://docs.oracle.com/en/java/javase/11/core/java-logging-overview.html#GUID-B83B652C-17EA-48D9-93D2-563AE1FF8EDA__SECURITY-4C4493F9)部分。

记录器会在日志记录命名空间中跟踪其父记录器。 记录器的父级是其在日志记录命名空间中最近的现存祖先。 根记录器（名为“”）没有父级。 匿名记录器都被赋予根记录器作为其父记录器。 记录器可以在记录器名称空间中从父项继承各种属性。记录器可以继承：

- 日志级别: 如果记录器的级别设置为null，则记录器将使用通过树向上查找并使用第一个非null级别获得的有效级别。
- Handlers： 默认情况下，Logger会将任何输出消息记录到其父级的处理程序，在树上依此类推。
- 资源包名称: 如果记录器没有设置资源包名称，那么它将继承为其父级定义的任何资源包名称，在树上依此类推.

### 记录方法

Logger类提供了一组用于生成日志消息的便捷方法。 为方便起见，每个日志记录级别都有方法，以日志记录级别名称命名。 因此，开发人员可以简单地调用方便方法logger.warning（...），而不是调用logger.log（Level.WARNING，...）

有两种不同风格的日志记录方法，可以满足不同社区用户的需求。

首先，有一些方法采用显式的源类名和源方法名。 这些方法适用于希望能够快速找到任何给定日志消息源的开发人员。 这种风格的一个例子是：

```java
void warning(String sourceClass, String sourceMethod, String msg);
```

其次，有一组方法不采用显式的源类或源方法名称。 这些适用于需要易于使用的日志记录且不需要详细源信息的开发人员。

```java
void warning(String msg);
```

对于第二组方法，Logging框架将“尽最大努力”确定调用日志框架的类和方法，并将此信息添加到LogRecord中。 但是，重要的是要意识到这种自动推断的信息可能只是近似的。 虚拟机在即时编译时执行广泛的优化，并且可能完全删除堆栈帧，从而无法可靠地定位调用类和方法。

### Handlers

Java SE提供以下Handler类：

- StreamHandler：用于将格式化记录写入OutputStream的简单处理程序。
- ConsoleHandler：用于将格式化记录写入System.err的简单处理程序
- FileHandler：将格式化日志记录写入单个文件或一组旋转日志文件的处理程序。
- SocketHandler：将格式化日志记录写入远程TCP端口的处理程序。
- MemoryHandler：缓冲内存中日志记录的处理程序。

开发新的Handler类非常简单。 需要特定功能的开发人员可以从头开发处理程序，也可以子类化其中一个提供的处理程序。

### 格式化

Java SE还包括两个标准的Formatter类：

- SimpleFormatter：简短的“人类可读”日志记录摘要。
- XMLFormatter：写入详细的XML结构信息。

与处理程序一样，开发新的格式化程序也相当简单。

### 日志管理

有一个全局LogManager对象可以跟踪全局日志记录信息。 这包括：

- 命名Loggers的分层命名空间。
- 从配置文件中读取的一组日志记录控制属性。

可以使用静态LogManager.getLogManager方法检索单个LogManager对象。 这是在LogManager初始化期间根据系统属性创建的。 此属性允许容器应用程序（例如EJB容器）替换它们自己的LogManager子类来代替默认类。

## 配置文件

可以使用将在启动时读取的日志记录配置文件初始化日志记录配置。此日志记录配置文件采用标准java.util.Properties格式。

或者，可以通过指定可用于读取初始化属性的类来初始化日志记录配置。此机制允许从任意源（例如LDAP和JDBC）读取配置数据。

有一小组全局配置信息。这在LogManager类的描述中指定，并包括要在启动期间安装的根级处理程序的列表。

初始配置可以指定特定记录器的级别。这些级别将应用于命名层次结构中的命名记录器及其下方的任何记录器。级别按照在配置文件中定义的顺序应用。

初始配置可能包含供处理程序或执行日志记录的子系统使用的任意属性。按照惯例，这些属性应使用以处理程序类的名称开头的名称或子系统的主Logger的名称。

例如，MemoryHandler使用属性java.util.logging.MemoryHandler.size来确定其环形缓冲区的默认大小。

### 默认配置

JRE附带的默认日志记录配置仅为默认配置，可由ISV，系统管理员和最终用户覆盖。

默认配置仅限制使用磁盘空间。 它不会向用户提供信息，但确保始终捕获关键故障信息。

默认配置在根记录器上建立单个处理程序，用于将输出发送到控制台。

#### 动态配置更新

程序员可以通过多种方式在运行时更新日志记录配置：

- FileHandler，MemoryHandler和ConsoleHandler对象都可以使用各种属性创建
- 可以添加新的Handler对象，删除旧的对象。
- 可以创建新的Logger对象，并且可以使用特定的处理程序提供。
- 可以在目标Handler对象上设置级别对象。

#### Native方法

没有用于日志记录的Native API。

希望使用Java Logging机制的Native代码应该对Java Logging API进行正常的JNI调用。

### XML DTD

XMLFormatter使用的XML DTD在附录A：XMLFormatter输出的DTD中指定。

DTD的设计使用<log>元素作为顶级文档。 然后将各个日志记录写为<record>元素。

请注意，在JVM崩溃的情况下，可能无法使用适当</ log>干净地终止XMLFormatter流。 因此，应准备分析日志记录的工具以应对未终止的流。

### 唯一消息ID

Java Logging API不提供对唯一消息ID的任何直接支持。 那些需要唯一消息ID的应用程序或子系统应该定义它们自己的约定，并在适当的时候在消息字符串中包含唯一的ID。

### 安全性

主要安全性要求是不受信任的代码不能更改日志记录配置。 具体来说，如果已将日志记录配置设置为将特定类别的信息记录到特定处理程序，则不受信任的代码应该无法阻止或中断该日志记录。

安全权限LoggingPermission控制对日志记录配置的更新。

为受信任的应用程序提供适当的LoggingPermission，以便它们可以调用任何日志记录配置API。不受信任的applet是不同存储。不受信任的applet可以以正常方式创建和使用命名记录器，但不允许它们更改日志记录控制设置，例如添加或删除处理程序或更改日志级别。但是，不受信任的applet能够使用Logger.getAnonymousLogger创建和使用自己的“匿名”记录器。这些匿名记录器未在全局命名空间中注册，并且它们的方法未经过访问检查，但是允许不受信任的代码更改其日志记录控制设置。

日志框架不会尝试防止欺骗。无法可靠地确定日志记录调用的来源，因此当发布声称来自特定源类和源方法的LogRecord时，它可能是一种fabrication。类似地，格式化程序（如XMLFormatter）不会尝试保护自己免受消息字符串中的嵌套日志消息的影响。因此，欺骗性LogRecord可能在其消息字符串中包含一组欺骗性XML，使其看起来好像在输出中有另外的XML记录。

此外，日志记录框架不会尝试保护自己免受拒绝服务攻击。任何给定的日志记录客户端都可以使用无意义的消息充斥日志记录框架，以试图隐藏一些重要的日志消息。

### LogManager

API的结构使得初始配置信息集从配置文件中读取为属性。 然后可以通过对各种日志记录类和对象的调用以编程方式改变配置信息。

此外，LogManager上还有一些方法可以重新读取配置文件。 发生这种情况时，配置文件值将覆盖以编码方式进行的任何更改。

### Packaging

所有日志记录类都在java.util.logging包中的java.*部分命名空间中。

### 本地化

可能需要本地化日志消息。

每个记录器可能具有与之关联的ResourceBundle名。 相应的ResourceBundle可用于在原始消息字符串和本地化消息字符串之间进行映射。

通常，格式化程序执行本地化。 为方便起见，Formatter类提供了formatMessage方法，该方法提供了一些基本的本地化和格式化支持。

#### 远程访问和序列化

与大多数Java平台API一样，日志记录API设计用于单个地址空间。所有调用都是本地调用。但是，预计一些处理程序会将其输出转发到其他系统。有多种方法可以做到这一点：

某些处理程序（例如SocketHandler）可能使用XMLFormatter将数据写入其他系统。这提供了一种简单，标准，互换的格式，可以在各种系统上进行解析和处理。

一些处理程序可能希望通过RMI传递LogRecord对象。因此，LogRecord类是可序列化的。但是，如何处理LogRecord参数存在问题。某些参数可能无法序列化，并且其他参数可能已设计为序列化比记录所需的更多状态。为了避免这些问题，LogRecord类有一个自定义的writeObject方法，该方法在将参数写出之前将参数转换为字符串（使用Object.toString（））。

大多数日志记录类都不是可序列化的。记录器和处理程序都是绑定到特定虚拟机的有状态类。在这方面，它们类似于java.io类，它们也不可序列化。

## Java Logging 例子

### 简单使用

以下是使用默认配置执行日志记录的小程序。

此程序依赖于LogManager根据配置文件建立的根处理程序。 它创建自己的Logger对象，然后调用该Logger对象来报告各种事件。

```java
package com.wombat;
import java.util.logging.*;

public class Nose {
    // Obtain a suitable logger.
    private static Logger logger = Logger.getLogger("com.wombat.nose");
    public static void main(String argv[]) {
        // Log a FINE tracing message
        logger.fine("doing stuff");
        try {
            Wombat.sneeze();
        } catch (Exception ex) {
            // Log the exception
            logger.log(Level.WARNING, "trouble sneezing", ex);
        }
        logger.fine("done");
    }
}
```

### 更改配置

这一段小代码，可以动态调整日志记录配置，将输出发送到特定文件，并获取有关wombats的大量信息。 模式％t表示系统临时目录。

```java
public static void main(String[] args) {
    Handler fh = new FileHandler("%t/wombat.log");
    Logger.getLogger("").addHandler(fh);
    Logger.getLogger("com.wombat").setLevel(Level.FINEST);
    ...
}
```

### 简单使用，忽略全局配置

这一段小代码，它设置自己的日志记录处理程序并忽略全局配置。

```java
package com.wombat;

import java.util.logging.*;

public class Nose {
    private static Logger logger = Logger.getLogger("com.wombat.nose");
    private static FileHandler fh = new FileHandler("mylog.txt");
    public static void main(String argv[]) {
        // Send logger output to our FileHandler.
        logger.addHandler(fh);
        // Request that every detail gets logged.
        logger.setLevel(Level.ALL);
        // Log a simple INFO message.
        logger.info("doing stuff");
        try {
            Wombat.sneeze();
        } catch (Exception ex) {
            logger.log(Level.WARNING, "trouble sneezing", ex);
        }
        logger.fine("done");
    }
}
```

### XML输出示例

以下是一些XMLFormatter XML输出的小样本：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
  <record>
    <date>2015-02-27T09:35:44.885562Z</date>
    <millis>1425029744885</millis>
    <nanos>562000</nanos>
    <sequence>1256</sequence>
    <logger>kgh.test.fred</logger>
    <level>INFO</level>
    <class>kgh.test.XMLTest</class>
    <method>writeLog</method>
    <thread>10</thread>
    <message>Hello world!</message>
  </record>
</log>
```

附录A：XMLFormatter输出的DTD

```xml
<!-- DTD used by the java.util.logging.XMLFormatter -->
<!-- This provides an XML formatted log message. -->

<!-- The document type is "log" which consists of a sequence
of record elements -->
<!ELEMENT log (record*)>

<!-- Each logging call is described by a record element. -->
<!ELEMENT record (date, millis, nanos?, sequence, logger?, level,
class?, method?, thread?, message, key?, catalog?, param*, exception?)>

<!-- Date and time when LogRecord was created in ISO 8601 format -->
<!ELEMENT date (#PCDATA)>

<!-- Time when LogRecord was created in milliseconds since
midnight January 1st, 1970, UTC. -->
<!ELEMENT millis (#PCDATA)>

<!-- Nano second adjustement to add to the time in milliseconds. 
This is an optional element, added since JDK 9, which adds further
precision to the time when LogRecord was created.
 -->
<!ELEMENT nanos (#PCDATA)>

<!-- Unique sequence number within source VM. -->
<!ELEMENT sequence (#PCDATA)>

<!-- Name of source Logger object. -->
<!ELEMENT logger (#PCDATA)>

<!-- Logging level, may be either one of the constant
names from java.util.logging.Level (such as "SEVERE"
or "WARNING") or an integer value such as "20". -->
<!ELEMENT level (#PCDATA)>

<!-- Fully qualified name of class that issued
logging call, e.g. "javax.marsupial.Wombat". -->
<!ELEMENT class (#PCDATA)>

<!-- Name of method that issued logging call.
It may be either an unqualified method name such as
"fred" or it may include argument type information
in parenthesis, for example "fred(int,String)". -->
<!ELEMENT method (#PCDATA)>

<!-- Integer thread ID. -->
<!ELEMENT thread (#PCDATA)>

<!-- The message element contains the text string of a log message. -->
<!ELEMENT message (#PCDATA)>

<!-- If the message string was localized, the key element provides
the original localization message key. -->
<!ELEMENT key (#PCDATA)>

<!-- If the message string was localized, the catalog element provides
the logger's localization resource bundle name. -->
<!ELEMENT catalog (#PCDATA)>

<!-- If the message string was localized, each of the param elements
provides the String value (obtained using Object.toString())
of the corresponding LogRecord parameter. -->
<!ELEMENT param (#PCDATA)>

<!-- An exception consists of an optional message string followed
by a series of StackFrames. Exception elements are used
for Java exceptions and other java Throwables. -->
<!ELEMENT exception (message?, frame+)>

<!-- A frame describes one line in a Throwable backtrace. -->
<!ELEMENT frame (class, method, line?)>

<!-- an integer line number within a class's source file. -->
<!ELEMENT line (#PCDATA)>
```

