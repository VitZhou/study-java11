

# 模块化系统 quick start

本文档提供了一些简单的示例，以帮助开发人员开始使用模块。

示例中的文件路径使用正斜杠，路径分隔符是冒号。 Microsoft Windows上的开发人员应使用带有反斜杠的文件路径和分号作为路径分隔符.

## Greetings

第一个例子是一个名为com.greetings的模块，它只是打印“Greetings！”。 该模块由两个源文件组成：模块声明（module-info.java）和main class。

按照惯例，模块的源代码位于模块名称的目录中.

```shell
src/com.greetings/com/greetings/Main.java
src/com.greetings/module-info.java
```

```shel
$ cat src/com.greetings/module-info.java
    module com.greetings { }
$ cat src/com.greetings/com/greetings/Main.java
    package com.greetings;
    public class Main {
        public static void main(String[] args) {
            System.out.println("Greetings!");
        }
    }
```

使用以下命令将源代码编译到mods/com.greetings目录：

```shell
$ mkdir -p mods/com.greetings

$ javac -d mods/com.greetings \
    src/com.greetings/module-info.java \
    src/com.greetings/com/greetings/Main.java
```

现在我们使用以下命令运行示例：

```shell
 $ java --module-path mods -m com.greetings/com.greetings.Main
```

--module-path是模块路径，其值是包含模块的一个或多个目录。 -m选项指定主模块，斜杠后面的值是模块中主类的类名

##  Greetings Word

第二个示例更新模块声明以声明对模块org.astro的依赖性。 模块org.astro导出API包org.astro。

```shell
	src/org.astro/module-info.java
	src/org.astro/org/astro/World.java
	src/com.greetings/com/greetings/Main.java
	src/com.greetings/module-info.java

 	$ cat src/org.astro/module-info.java
    module org.astro {
        exports org.astro;
    }
    $ cat src/org.astro/org/astro/World.java
    package org.astro;
    public class World {
        public static String name() {
            return "world";
        }
    }

    $ cat src/com.greetings/module-info.java
    module com.greetings {
        requires org.astro;
    }

    $ cat src/com.greetings/com/greetings/Main.java
    package com.greetings;
    import org.astro.World;
    public class Main {
        public static void main(String[] args) {
            System.out.format("Greetings %s!%n", World.name());
        }
    }
```

这些模块一次编译一个。 编译模块com.greetings的javac命令指定模块路径，以便可以解析对模块org.astro的引用及其导出包中的类型。

```shell
    $ mkdir -p mods/org.astro mods/com.greetings

    $ javac -d mods/org.astro \
        src/org.astro/module-info.java src/org.astro/org/astro/World.java

    $ javac --module-path mods -d mods/com.greetings \
        src/com.greetings/module-info.java src/com.greetings/com/greetings/Main.java
```

该示例的运行方式与第一个示例完全相同：

```shell
   $ java --module-path mods -m com.greetings/com.greetings.Main
   Greetings world!
```

## 多模块编译

在上一个示例中，模块com.greetings和模块org.astro分别编译。 也可以使用一个javac命令编译多个模块：

```shell
    $ mkdir mods

    $ javac -d mods --module-source-path src $(find src -name "*.java")

    $ find mods -type f
    mods/com.greetings/com/greetings/Main.class
    mods/com.greetings/module-info.class
    mods/org.astro/module-info.class
    mods/org.astro/org/astro/World.class
```

## 打包

在到目前为止的示例中，编译模块的内容在文件系统上展开。 出于运输和部署的目的，将模块打包为模块化JAR通常更方便。 模块化JAR是一个常规JAR文件，在其顶级目录中有一个module-info.class。 以下示例在目录mlib中创建org.astro@1.0.jar和com.greetings.jar。

```shell
    $ mkdir mlib

    $ jar --create --file=mlib/org.astro@1.0.jar \
        --module-version=1.0 -C mods/org.astro .

    $ jar --create --file=mlib/com.greetings.jar \
        --main-class=com.greetings.Main -C mods/com.greetings .

    $ ls mlib
    com.greetings.jar   org.astro@1.0.jar
```

在此示例中，打包模块org.astro以指示其版本为1.0。 模块com.greetings已打包，表明其Main class是com.greetings.Main。 我们现在可以执行模块com.greetings而无需指定其Main class：

```shell
$ java -p mlib -m com.greetings
Greetings world!
```

使用-p作为--module-path的替代方法也可以缩短命令行。

jar工具有许多新选项（参见jar -help），其中之一是打印打包为模块化JAR的模块的模块声明。

```shell
$ jar --describe-module --file=mlib/org.astro@1.0.jar
    org.astro@1.0 jar:file:///d/mlib/org.astro@1.0.jar/!module-info.class
    exports org.astro
    requires java.base mandated
```

## 缺少requires或exports

现在让我们看看当我们错误地省略com.greetings模块声明中的requires时，上一个示例会发生什么：

```shell
$ cat src/com.greetings/module-info.java
module com.greetings {
	// requires org.astro;
}

 $ javac --module-path mods -d mods/com.greetings \
 src/com.greetings/module-info.java src/com.greetings/com/greetings/Main.java
    src/com.greetings/com/greetings/Main.java:2: error: package org.astro is not visible
    import org.astro.World;
     (package org.astro is declared in module org.astro, but module com.greetings does not read it) 1 error
```

我们现在修复了这个模块声明，但引入了一个不同的错误，这次我们省略了org.astro模块声明的导出：

```shell
	$ cat src/com.greetings/module-info.java
    module com.greetings {
        requires org.astro;
    }
    $ cat src/org.astro/module-info.java
    module org.astro {
        // exports org.astro;
    }
```

```shell
	$ javac --module-path mods -d mods/com.greetings \
        src/com.greetings/module-info.java src/com.greetings/com/greetings/Main.java
    $ javac --module-path mods -d mods/com.greetings \
       src/com.greetings/module-info.java src/com.greetings/com/greetings/Main.java
    src/com.greetings/com/greetings/Main.java:2: error: package org.astro is not visible
        import org.astro.World;
                  ^
      (package org.astro is declared in module org.astro, which does not export it)
    1 error
```

## Services

Services允许服务使用者模块和服务提供者模块之间的松耦合。

此示例具有服务使用者模块和服务提供者模块：

- com.socket模块导出网络套接字的API。 API在com.socket包中，因此export此包。 API是可插拔的，以允许替代实现。 service type是同一模块中的com.socket.spi.NetworkSocketProvider，因此也会导出包com.socket.spi。
- org.fastsocket模块是一个服务提供者模块。 它提供了com.socket.spi.NetworkSocketProvider的实现。 它不会导出任何包。

以下是com.socket模块的源代码。

```shell
    $ cat src/com.socket/module-info.java
    module com.socket {
        exports com.socket;
        exports com.socket.spi;
        uses com.socket.spi.NetworkSocketProvider;
    }

    $ cat src/com.socket/com/socket/NetworkSocket.java
    package com.socket;

    import java.io.Closeable;
    import java.util.Iterator;
    import java.util.ServiceLoader;

    import com.socket.spi.NetworkSocketProvider;

    public abstract class NetworkSocket implements Closeable {
        protected NetworkSocket() { }

        public static NetworkSocket open() {
            ServiceLoader<NetworkSocketProvider> sl
                = ServiceLoader.load(NetworkSocketProvider.class);
            Iterator<NetworkSocketProvider> iter = sl.iterator();
            if (!iter.hasNext())
                throw new RuntimeException("No service providers found!");
            NetworkSocketProvider provider = iter.next();
            return provider.openNetworkSocket();
        }
    }


    $ cat src/com.socket/com/socket/spi/NetworkSocketProvider.java
    package com.socket.spi;

    import com.socket.NetworkSocket;

    public abstract class NetworkSocketProvider {
        protected NetworkSocketProvider() { }

        public abstract NetworkSocket openNetworkSocket();
    }
```

以下是模块org.fastsocket的源代码。

```shell
    $ cat src/org.fastsocket/module-info.java
    module org.fastsocket {
        requires com.socket;
        provides com.socket.spi.NetworkSocketProvider
            with org.fastsocket.FastNetworkSocketProvider;
    }

    $ cat src/org.fastsocket/org/fastsocket/FastNetworkSocketProvider.java
    package org.fastsocket;

    import com.socket.NetworkSocket;
    import com.socket.spi.NetworkSocketProvider;

    public class FastNetworkSocketProvider extends NetworkSocketProvider {
        public FastNetworkSocketProvider() { }

        @Override
        public NetworkSocket openNetworkSocket() {
            return new FastNetworkSocket();
        }
    }

    $ cat src/org.fastsocket/org/fastsocket/FastNetworkSocket.java
    package org.fastsocket;

    import com.socket.NetworkSocket;

    class FastNetworkSocket extends NetworkSocket {
        FastNetworkSocket() { }
        public void close() { }
    }
```

为简单起见，我们将两个模块一起编译。 在实践中，服务消费者模块和服务提供者模块几乎总是分开编译。

```shell
$ mkdir mods
$ javac -d mods --module-source-path src $(find src -name "*.java")
```

最后，我们修改我们的模块com.greetings以使用API。

```shell
    $ cat src/com.greetings/module-info.java
    module com.greetings {
        requires com.socket;
    }

    $ cat src/com.greetings/com/greetings/Main.java
    package com.greetings;

    import com.socket.NetworkSocket;

    public class Main {
        public static void main(String[] args) {
            NetworkSocket s = NetworkSocket.open();
            System.out.println(s.getClass());
        }
    }
    
    $ javac -d mods/com.greetings/ -p mods $(find src/com.greetings/ -name "*.java")
```

最后我们运行它：

```shell
$ java -p mods -m com.greetings/com.greetings.Main
class org.fastsocket.FastNetworkSocket
```

输出确认已找到服务提供者，并且它已用作NetworkSocket的工厂。

## 连接

jlink是链接器工具，可用于链接一组模块及其传递依赖性，以创建自定义模块化运行时镜像（请参阅[JEP 220](http://openjdk.java.net/jeps/220)）。
该工具目前要求模块路径上的模块以模块化JAR或JMOD格式打包。 JDK构建以JMOD格式打包标准和JDK特定模块。

以下示例创建一个运行时镜像，其中包含模块com.greetings及其传递依赖项：

```shell
jlink --module-path $JAVA_HOME/jmods:mlib --add-modules com.greetings --output greetingsapp
```

-module-path的值是包含打包模块的目录的PATH。 在Microsoft Windows上将路径分隔符'：'替换为';' 。

$JAVA_HOME/jmods是包含java.base.jmod和其他标准和JDK模块的目录。

module path上的mlib目录包含模块com.greetings的工件。

jlink工具支持许多高级选项来自定义生成的镜像，有关更多选项，请参阅jlink --help

## `--patch-module`

从Doug Lea的CVS中检出java.util.concurrent类的开发人员将用于编译源文件并使用-Xbootclasspath/p部署这些类。

-Xbootclasspath/p已被删除，其模块替换是选项--patch-module来覆盖模块中的类。 它还可以用于增加模块的内容。 javac也支持--patch-module选项来编译代码“as if”模块的一部分。

这是一个编译新版本的java.util.concurrent.ConcurrentHashMap并在运行时使用它的示例：

```shell
javac --patch-module java.base=src -d mypatches/java.base \
        src/java.base/java/util/concurrent/ConcurrentHashMap.java

java --patch-module java.base=mypatches/java.base ...
```

