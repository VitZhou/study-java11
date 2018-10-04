# 序列化过滤

现在可以使用java序列化过滤机制来帮助防止反序列化漏洞.你可以定义基于pattern的过滤器.也可以创建自定义过滤器.

- 解决反序列化漏洞
- java序列化过滤器
- 黑白名单
- 创建基于pattern的过滤器
- 创建自定义过滤器
- 内置过滤器
- 记录过滤器操作

## 解决反序列化漏洞

接受不受信任的数据并对其反序列化的应用很容易受到攻击. 你可以创建过滤器在序列化对象的反序列化之前对齐进行筛选.

对象的序列化就是将其状态转换为字节流时。该流可以发送到文件,数据库或者经过网络传输.如果java对象的类或其他任何超类实现了java.io.Serializable接口或java.io.Externalizable子接口，则该对象是可序列化的。在JDK中,序列化用于许多领域,包括远程方法调用(RMI:remote method invoke)，用于进程间通信(IPC)协议和自定义RMI（例如spring http调用）,Java Management Extensions(JMX)和java消息传递服务(JMS)

将一个对象从字节流转换为对象称之为反序列化.确保次转换的安全性非常重要.反序列化是执行代码,因为反序列的类readObject方法可以包干自定义代码.可序列化类(也称为"小工具类")可以执行任意反射操作,例如将创建类和调用它们的方法.如果你的应用反序列化这些类,则可能导致拒绝服务或远程执行代码。

创建过滤器时,可以指定应用程序可接收的类.以及应该拒绝的类.你可以在反序列化期控制对象的图的大小和复杂性.以便对象图不会超出合理的限制.过滤器可以配置为属性,也可以通过编程方式实现.

除了创建过滤器之外,你还可以采用以下操作来防止反序列化漏洞:

- 不反序列化任何不受信任的数据.
- 使用SSL加密和颜泽应用程序之间的连接.
- 在复制之前验证字段值.包括使用readObject方法检测对象的不变量

> jdk为RMI提供了内置过滤器.但是你应该只使用这些内置过滤器作为起点.配置黑名单或扩展白名单,为使用RMI的应用天机额外保护.

## java序列化过滤器

java序列化过滤机制筛选序列化对象的传入字节流,以帮助提高安全性和健壮性.过滤器可用在反序列化之前验证传入的类.

如JEP 290中所述，Java序列化过滤机制的目标是：
提供一种方法来将可以反序列化的类缩小到适合上下文的类集。

在反序列化期间为过滤器提供图表大小和复杂性的度量标准，以验证正常的图形行为。

允许RMI导出的对象验证调用中所需的类。

您可以通过以下方式实现序列化过滤器：

- 基于pattern的过滤器不需要你修改应用的代码.它们由属性,匹配文件或命令行中定义的一系列pattern组成·基于pattern的过滤器可用接收或拒绝特定的类,包和模块.它们可以限制数组大小,图行深度,总引用和流大小.一个典型的用例是将已被识别为可能危及java运行的类列入黑名单.基于pattern的过滤器是为一个应用程序或过程中的所有应用程序定义的。
- 自定义过滤器使用ObjectInputFilter api实现.它们允许应用程序集成比基于pattern的过滤器更精细的控制,因为它们可以使用特定一每个ObjectInputStream。自定义过滤器可以设置在单个输入流或进程中的所有流上。

为流中的每个新对象调用过滤器机制.如果存在多个活动过滤器(过程范围过滤器,应用程序过滤器或流特定过滤器),则仅调用最具体的过滤器.

在大多数情况下,自定义过滤器赢检查是否设置了过程范围的过滤器.如果存在,则自定义筛选器应调用并使用流程范围的筛选器结果,除非状态为UNDECIDED.

从JDK 9开始，以及从8u121,7u131和6u141开始的Java CPU版本中包括对序列化过滤器的支持。

## 黑白名单

黑明白名单可以通过基于pattern或者自定义的过滤器实现.这些列表允许您采取主动和防御性方法来保护您的应用程序。

主动方法使用白名单仅接受已识别和受信任的类.你可以在开发应用时在代码中实现白名单,也可以稍后通过定义基于pattern的过滤器来实现白名单.如果你的应用只处理以小组类,那么这种方法可以很好的工作.你可以通过制定允许的类,包或者模块来实现白名单.

防御方法使用黑名单来拒绝不信任的类.通常,黑名单是在发现一个类是一个问题的攻击之后实现的.通过定义基于pattern的过滤器,可以将类添加到黑名单中,而唔需要更改代码.

## 创建基于模糊匹配的过滤器

基于pattern的过滤器是你定义的过滤器,无需更改应用代码.你可以在属性文件中添加流程范围的过滤器,或在java命令行上添加特定于应用的过滤器。

基于pattern的过滤器是一系列pattern。每个pattern都与六种的类名称和资源限制相匹配.基于类和资源限制模式可以组合在一个过滤字符串中.每个pattern用分号;分隔.

### 基于pattern的过滤器语法

创建由pattern组成的过滤器时，请使用以下准则：

- 用分号分隔Pattern,例如

  ```java
  pattern1.*;pattern2.*
  ```

- 白名单很重要，被认为是模式的一部分。

- 将limit放在字符串中的第一位。 无论它们在字符串中什么位置，都会首先对它们进行评估，因此首先将它们加以排序。 否则，从左到右评估模式。

- 一个pattern前面有!则它匹配到的类被拒绝，一个没有被parrent前面没有!则,它匹配到的类被接受.例如下面例子,拒绝pattern1.MyClass但接受pattern2.MyClass：

  ```java
  !pattern1.*;pattern2.*
  ```

- 使用通配符（*）表示模式中未指定的类，如以下示例所示：

  - 使用*匹配所有类
  - 要匹配mypackage中的每个类，请使用mypackage.*
  - 要匹配mypackage及其子包中的每个类，请使用mypackage.**
  - 要匹配以text开头的每个类，请使用text*

  如果某个类与任何过滤器都不匹配，则会接受该类。 如果您只想接受某些类，那么您的过滤器必须拒绝所有不匹配的内容。 要拒绝除指定类之外的所有类，请将!*作为类过滤器中的最后一个pattern。

  有关模式语法的完整说明，请参阅conf/security/java.security文件，或参阅[JEP 290](http://openjdk.java.net/jeps/290)。

#### 基础pattern的过滤器的限制

基于pattern的过滤器用于简单接受或拒绝。 这些过滤器有一些限制。 例如：

- Patterns不基于类的数组的大小不一致.
- Patterns无法根据类的超类类型型或接口匹配类。
- Patterns没有状态，无法根据流中反序列化的早期类进行选择。

#### 为一个应用程序定义基于Patterns的过滤器

您可以将基于Pattern的过滤器定义为一个应用程序的系统属性。 系统属性取代安全属性值。

要创建仅应用于一个应用程序的过滤器，并且仅创建一次Java调用，请在命令行中定义jdk.serialFilter系统属性。

以下示例显示如何限制单个应用程序的资源使用情况：

```shell
java -Djdk.serialFilter=maxarray=100000;maxdepth=20;maxrefs=500 com.example.test.Application
```

#### 为Process中的所有应用程序定义基于Pattern的过滤器

您可以为进程中的所有应用程序将基于Pattern的过滤器定义为security属性。 系统属性取代security属性值。

- 编辑java.security属性文件。
  - JDK 9 和之后版本: `$JAVA_HOME/conf/security/java.security`
  - JDK 8,7,6: `$JAVA_HOME/lib/security/java.security` 
- 将模式添加到jdk.serialFilter安全属性。

#### 定义类过滤器

您可以创建全局应用的基于Pattern的类过滤器。 例如，模式可能是类名或带通配符的包。

在以下示例中，过滤器拒绝包中的一个类（!example.somepackage.SomeClass），并接受包中的所有其他类：

```java
jdk.serialFilter=!example.somepackage.SomeClass;example.somepackage.*;
```

前面的示例过滤器接受所有其他类，而不仅仅是example.somepackage.* 中的类。 要拒绝example.somepackage包意外的所有其他类，请添加!*：

```java
jdk.serialFilter=!example.somepackage.SomeClass;example.somepackage.*;!*
```

#### 定义一个资源限制过滤器

资源过滤器限制图形的复杂性和大小。 您可以为以下参数创建过滤器，以控制每个应用程序的资源使用情况：

- 数组最大大小, For example: `maxarray=100000;`
- 图的最深深度,例如:maxdepth=20;
- 对象之间的最大引用:maxrefs=500;
- 流中的最大字节数。 例如：maxbytes = 500000;

## 创建自定义过滤器

自定义过滤器是您在应用程序代码中指定的过滤器。 它们设置在单个流上或Process中的所有流上。 您可以将自定义过滤器实现为Pattern，方法，lambda表达式或类。

### 读取序列化对象流

您可以在一个ObjectInputStream上设置自定义过滤器，或者，要将相同的过滤器应用于每个流，请设置Process范围的过滤器。 如果ObjectInputStream没有为其定义过滤器，则调用Process范围的过滤器（如果有）。

在队列进行解码时,会进行以下操作:

- 对于流中的每个新对象，在对象序列化和反序列化之前调用过滤器。
- 对于流中的每个类，使用已解析的类调用过滤器。 它是为流中的每个超类型和接口单独调用的。
- 过滤器可以检查流中引用的每个类，包括要创建的对象类，这些类的超类型及其接口。
- 对于流中的每个数组，无论是基元数组，字符串数组还是对象数组，都会使用数组类和数组长度调用过滤器。
- 对于已从流中读取的对象的每个引用，都会调用过滤器，以便检查深度，引用数和流长度。 深度从1开始，每个嵌套对象增加，并在每个嵌套调用返回时减少。
- 不会为基元(原函数)或在流中具体编码的java.lang.String实例调用过滤器。
- 过滤器返回接受，拒绝或未定的状态。
- 如果启用了日志记录，则会记录筛选操作

除非过滤器拒绝对象，否则接受该对象。

#### 为单个流设置自定义过滤器

当流的输入不受信任且过滤器具有一组有限的类或约束来强制执行时，您可以在单个ObjectInputStream上设置过滤器。 例如，您可以确保流只包含数字，字符串和其他应用程序指定的类型。

使用setObjectInputFilter方法设置自定义过滤器。 必须在从流中读取对象之前设置自定义筛选器。

在以下示例中，使用dateTimeFilter方法调用setObjectInputFilter方法。 此过滤器仅接受来自java.time包的类。 dateTimeFilter方法在将自定义筛选器设置为方法的代码示例中定义。

```java
 LocalDateTime readDateTime(InputStream is) throws IOException {
        try (ObjectInputStream ois = new ObjectInputStream(is)) {
            ois.setObjectInputFilter(FilterClass::dateTimeFilter);
            return (LocalDateTime) ois.readObject();
        } catch (ClassNotFoundException ex) {
            IOException ioe = new StreamCorruptedException("class missing");
            ioe.initCause(ex);
            throw ioe;
        }
    }
```

#### 设置Process-wide的自定义筛选器

您可以设置适用于ObjectInputStream的每次使用的Process-wide过滤器，除非它在特定流上被覆盖。 如果您可以识别整个应用程序所需的每种类型和条件，则过滤器可以允许这些类型和条件，并拒绝其余的。 通常，过程范围的过滤器用于拒绝特定的类或包，或限制数组大小，图深度或总图大小。

使用ObjectInputFilter.Config类的方法设置一个Process-wide的过滤器。 过滤器可以是类的实例，lambda表达式，方法引用或模式。

```java
ObjectInputFilter filter = ...
    ObjectInputFilter.Config.setSerialFilter(filter);
```

在以下示例中，使用lambda表达式设置Process-wide的过滤器。

在以下示例中，使用lambda表达式设置过程范围的过滤器。

```java
ObjectInputFilter.Config.setSerialFilter(info -> info.depth() > 10 ? Status.REJECTED : Status.UNDECIDED);
```

在以下示例中，通过使用类的实例来设置过程范围的过滤器。

```java
ObjectInputFilter.Config.setSerialFilter(FilterClass::dateTimeFilter);
```

#### 使用Pattern设置自定义过滤器

可以使用ObjectInputFilter.Config.createFilter方法创建基于Pattern的自定义过滤器，该过滤器可以方便地处理简单情况。 您可以创建基于Pattern的过滤器作为系统属性或安全属性。 将基于Pattern的过滤器实现为方法或lambda表达式可以提供更大的灵活性。

过滤器模式可以接受或拒绝特定的类，包，模块，并可以对数组大小，图形深度，总引用和流大小设置限制。 模式无法匹配类的超类型或接口。

在以下示例中，过滤器允许example.File并拒绝example.Directory类。

```java
ObjectInputFilter filesOnlyFilter = ObjectInputFilter.Config.createFilter("example.File;!example.Directory");
```

此示例仅允许example.File。 所有其他课程都被拒绝。

```java
ObjectInputFilter filesOnlyFilter = ObjectInputFilter.Config.createFilter("example.File;!*");
```

#### 将自定义过滤器设置为类

自定义过滤器可以实现为实现java.io.ObjectInputFilter接口的类，lambda表达式或方法。

过滤器通常是无状态的，仅根据输入参数执行检查。 但是，您可以实现一个过滤器，例如，在对checkInput方法的调用之间维护状态以计算流中的组件。

在以下示例中，FilterNumber类允许任何作为Number类实例的对象并拒绝所有其他对象。

```java
class FilterNumber implements ObjectInputFilter {
        public Status checkInput(FilterInfo filterInfo) {
            Class<?> clazz = filterInfo.serialClass();
            if (clazz != null) {
                return (Number.class.isAssignableFrom(clazz)) ? Status.ALLOWED : Status.REJECTED;
            }
            return Status.UNDECIDED;
        }
    }
```

#### 示例：过滤java.base模块中的类

此自定义过滤器（也作为方法实现）仅允许在JDK的基本模块中找到的类。 此示例适用于JDK 9及更高版本。

```java
static ObjectInputFilter.Status baseFilter(ObjectInputFilter.FilterInfo info) {
            Class<?> serialClass = info.serialClass();
            if (serialClass != null) {
                return serialClass.getModule().getName().equals("java.base")
                        ? ObjectInputFilter.Status.ALLOWED
                        : ObjectInputFilter.Status.REJECTED;
            }
            return ObjectInputFilter.Status.UNDECIDED;
       }
```

## 内置过滤器

Java远程方法调用（RMI）注册表，RMI分布式垃圾收集器和Java管理扩展（JMX）都具有JDK中包含的过滤器。 您应该为RMI注册表和RMI分布式垃圾收集器指定自己的过滤器以添加其他保护。

#### RMI注册表的过滤器

> 仅使用这些内置过滤器作为起点。 编辑sun.rmi.registry.registryFilter系统属性以配置黑名单和/或扩展白名单以为RMI注册表添加其他保护。 要保护整个应用程序，请将模式添加到jdk.serialFilter全局系统属性，以增强对没有自己的自定义筛选器的其他序列化用户的保护。

RMI Registry具有内置的白名单过滤器，允许在注册表中绑定对象。 它包括java.rmi.Remote，java.lang.Number，java.lang.reflect.Proxy，java.rmi.server.UnicastRef，java.rmi.activation.ActivationId，java.rmi.server.UID，java的实例。 .rmi.server.RMIClientSocketFactory和java.rmi.server.RMIServerSocketFactory类。

内置过滤器包括大小限制：

```java
maxarray=1000000,maxdepth=20
```

通过使用带有模式的sun.rmi.registry.registryFilter系统属性定义过滤器来取代内置过滤器。 如果您定义的过滤器接受传递给过滤器的类，或者拒绝类或大小，则不会调用内置过滤器。 如果您的过滤器不接受或拒绝任何内容，则调用内置过滤器。

#### RMI分布式垃圾收集器的过滤器

仅使用这些内置过滤器作为起点。 编辑sun.rmi.transport.dgcFilter系统属性以配置黑名单和/或扩展白名单为分布式垃圾收集器添加其他保护。 要保护整个应用程序，请将模式添加到jdk.serialFilter全局系统属性，以增强对没有自己的自定义过滤器的其他序列化用户的保护。

RMI分布式垃圾收集器具有内置白名单过滤器，可接受一组有限的类。 它包括java.rmi.server.ObjID，java.rmi.server.UID，java.rmi.dgc.VMID和java.rmi.dgc.Lease类的实例。

内置过滤器包括大小限制：

```java
maxarray=1000000,maxdepth=20
```

通过使用带有Pattern的sun.rmi.transport.dgcFilter系统属性定义过滤器来取代内置过滤器。 如果过滤器接受传递给过滤器的类，或拒绝类或大小，则不会调用内置过滤器。 如果取代过滤器不接受或拒绝任何内容，则调用内置过滤器。

#### JMX的过滤器

您可以指定在进行RMIServer.newClient远程调用时以及通过RMI将反序列化参数发送到服务器时要使用的反序列化过滤器Pattern字符串。 您还可以使用management.properties文件向默认代理程序提供过滤器模式字符串。

### 记录过滤器操作

您可以打开日志记录以记录对序列化过滤器的调用的初始化，拒绝和接受。 使用日志输出作为诊断工具来查看要反序列化的内容，并在配置白名单和黑名单时确认您的设置。

启用日志记录后，过滤器操作将记录到java.io.serialization记录器中。

要启用序列化过滤器日志记录，请编辑$ JDK_HOME / conf / logging.properties文件。

要记录被拒绝的调用，请添加

```properties
java.io.serialization.level = FINER
```

要记录所有过滤结果，请添加

```properties
java.io.serialization.level = FINEST
```

