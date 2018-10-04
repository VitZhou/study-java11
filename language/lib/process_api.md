# Process(进程) API

Process API允许您启动，检索有关本机操作系统进程的信息并对其进行管理。

使用此API，您可以按如下方式使用操作系统进程：

- 运行任意命令
  - 过滤正则运行的进程。
  - 重定向输出
  - 通过调度进程来连接异构命令和shell，以便在另一个结束时启动。
- 测试命令的执行
  - 运行一系列测试。
  - 记录输出。
  - 清理剩余的进程。
- 监控命令
  - 监视长时间运行的进程并在它们终止时重新启动它们
  - 收集使用统计信息

## Process Api的类和接口

Process API由类和接口ProcessBuilder，Process，ProcessHandle和ProcessHandle.Info组成。

### ProcessBuilder类

ProcessBuilder类允许您创建和启动操作系统进程。

有关如何创建和启动流程的示例，请参考[Create Process](https://docs.oracle.com/en/java/javase/11/core/process-api1.html#GUID-81804DFA-D47A-4874-AF73-528365FEC711)。 [ProcessBuilder](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html)类管理各种进程属性，下表总结了这些属性：

| 进程属性                                        | 描述                                                         | 相关方法                                                     |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Command                                         | 指定要调用的外部程序文件的字符串及其参数（如果有）。         | - [ProcessBuilder](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#ProcessBuilder-java.lang.String...-)构造函数；                                                  - [command(String... command)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#command-java.lang.String...-) |
| Environment                                     | 环境变量（及其值）。 这最初是当前进程的系统环境的副本。      | [environment()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#environment--) |
| Working directory                               | 默认情况下，当前进程的当前工作目录。                         | - [directory()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#directory--)                                                                                 - [directory(File directory)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#directory-java.io.File-) |
| Standard input source                           | 默认情况下，进程从channel读取标准输入; 通过[Process.getOutputStream](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getOutputStream--)方法返回的输出流访问它。 | [redirectInput(ProcessBuilder.Redirect source)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#redirectInput-java.lang.ProcessBuilder.Redirect-) |
| Standard output and standard error destinations | 默认情况下，进程将标准输出和标准错误写入chanel; 通过[Process.getInputStream](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getInputStream--)和[Process.getErrorStream](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getErrorStream--)方法返回的输入流访问它们。 有关示例，请参考从进程重定向输出。 | - [redirectOutput(ProcessBuilder.Redirect destination)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#redirectOutput-java.lang.ProcessBuilder.Redirect-)                                             - [redirectError(ProcessBuilder.Redirect destination)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#redirectError-java.lang.ProcessBuilder.Redirect-) |
| redirectErrorStream property                    | 指定是将标准输出和错误输出作为两个单独的流（值为false）发送，还是将任何错误输出与标准输出合并（值为true） | - [redirectErrorStream()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#redirectErrorStream--)                          - [redirectErrorStream(boolean redirectErrorStream)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessBuilder.html#redirectErrorStream-boolean-) |

### Process 类

Process类中的方法允许您控制ProcessBuilder.start和Runtime.exec方法启动的进程。 下表总结了这些方法：

下表总结了Process类的方法。

| 方法类型                                                     | 相关方法                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 等待该进程完成。                                             | - [waitfor()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#waitFor--)  - [waitFor(long timeout, TimeUnit unit)](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#waitFor-long-java.util.concurrent.TimeUnit-) |
| 检索有关该进程的信息。                                       | - [isAlive()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#isAlive--) - [pid()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#pid--) - [info()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#info--) - [exitValue()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#exitValue--) |
| 检索输入，输出和错误流。 有关示例，请参考使用onExit方法终止时处理进程。 | - [getInputStream()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getInputStream--) - [getOutputStream()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getOutputStream--) - [getErrorStream()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#getErrorStream--) |
| 检索直接和间接子进程。                                       | - [children()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#children--) [descendants()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#descendants--) |
| 销毁或终止进程。                                             | [destroy()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#destroy--) - [destroy()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#destroy--) - [supportsNormalTermination()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#supportsNormalTermination--) |
| 返回一个CompletableFuture实例，该实例将在进程退出时完成。 有关示例，请参考使用onExit方法终止时处理进程。 | [onExit()](https://docs.oracle.com/javase/10/docs/api/java/lang/Process.html#onExit--) |

### ProcessHandle接口

ProcessHandle接口使您可以识别和控制本机进程。 Process类与ProcessHandle不同，因为它允许您控制仅由ProcessBuilder.start和Runtime.exec方法启动的进程; 但是，Process类允许您访问进程的输入，输出和错误流。

有关ProcessHandle接口的示例，请参考使用Streams过滤进程。 下表总结了此接口的方法：

| 方法类型                                                     | 相关方法                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 检索所有操作系统进程。                                       | [allProcesses()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#allProcesses--) |
| 检索进程句柄。                                               | - [current()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#current--) - [of(long pid)](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#of-long-) - [parent()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#parent--) |
| 检索进程的信息                                               | - [isAlive()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#isAlive--) - [pid()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#pid--) - [info()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#info--) |
| 检索直接和间接子进程的stream。                               | - [children()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#children--) - [descendants()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#descendants--) |
| 销毁进程                                                     | - [destroy()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#destroy--) - [destroyForcibly()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#destroyForcibly--) |
| 返回一个CompletableFuture实例，该实例将在进程退出时完成。 有关示例，请参考使用onExit方法终止时处理进程。 | [onExit()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.html#onExit--) |

### ProcessHandle.Info接口

ProcessHandle.Info接口允许您检索有关进程的信息，包括ProcessBuilder.start方法和本机进程创建的进程。

有关ProcessHandle.Info接口的示例，请参考获取有关进程的信息。 下表总结了此接口中的方法：

| 方法                                                         | 描述                               |
| ------------------------------------------------------------ | ---------------------------------- |
| [arguments()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#arguments--) | 以String数组的形式返回进程的参数。 |
| [command()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#command--) | 返回进程的可执行路径名。           |
| [commandLine()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#commandLine--) | 返回进程的命令行。                 |
| [startInstant()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#startInstant--) | 返回进程的开始时间。               |
| [totalCpuDuration()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#totalCpuDuration--) | 返回进程累计的CPU总时间。          |
| [user()](https://docs.oracle.com/javase/10/docs/api/java/lang/ProcessHandle.Info.html#user--) | 返回进程的用户。                   |

## 创建一个进程

要创建进程，首先使用ProcessBuilder类指定进程的属性，例如命令名称及其参数。 然后，使用ProcessBuilder.start方法启动该进程，该方法返回一个Process实例。

以下行创建并启动一个进程：

```java
  ProcessBuilder pb = new ProcessBuilder("echo", "Hello World!");
  Process p = pb.start();
```

在下面的摘录中，setEnvTest方法设置两个环境变量，`horse`和 `oats`，然后使用echo命令打印这些环境变量（以及系统环境变量HOME）的值：

```java
 public static void setEnvTest() throws IOException, InterruptedException {
    ProcessBuilder pb =
      new ProcessBuilder("/bin/sh", "-c", "echo $horse $dog $HOME").inheritIO();
    pb.environment().put("horse", "oats");
    pb.environment().put("dog", "treats");
    pb.start().waitFor();
  }
```

此方法打印以下内容（假设您的home目录是/home/admin）：

```shell
oats treats /home/admin
```

## 获取进程相关信息

Process.pid方法返回进程的本机进程ID。 Process.info方法返回ProcessHandle.Info实例，该实例包含有关该进程的其他信息，例如其可执行路径名，开始时间和用户。

在下面的摘录中，方法getInfoTest启动一个进程，然后打印有关它的信息：

```java
 public static void getInfoTest() throws IOException {
    ProcessBuilder pb = new ProcessBuilder("echo", "Hello World!");
    String na = "<not available>";
    Process p = pb.start();
    ProcessHandle.Info info = p.info();
    System.out.printf("Process ID: %s%n", p.pid());
    System.out.printf("Command name: %s%n", info.command().orElse(na));
    System.out.printf("Command line: %s%n", info.commandLine().orElse(na));
    
    System.out.printf("Start time: %s%n",
      info.startInstant().map(i -> i.atZone(ZoneId.systemDefault())
                                    .toLocalDateTime().toString())
                         .orElse(na));
    
    System.out.printf("Arguments: %s%n",
      info.arguments().map(a -> Stream.of(a)
                                      .collect(Collectors.joining(" ")))
                      .orElse(na));
  
    System.out.printf("User: %s%n", info.user().orElse(na));
  }
```

此方法打印类似于以下内容的输出：

```shell
Process ID: 18761
Command name: /usr/bin/echo
Command line: echo Hello World!
Start time: 2017-05-30T18:52:15.577
Arguments: <not available>
User: administrator
```

> 进程的属性因操作系统而异，并非在所有实现中都可用。 此外，有关进程的信息受制于请求的进程的操作系统特权限制。
>
> ProcessHandle.Info接口中的所有方法都返回Optional <T>的实例; 始终检查返回的值是否为空。

## 重定向进程的输出

默认情况下，进程将标准输出和标准错误写入channel。 在您的应用程序中，可以通过Process.getOutputStream和Process.getErrorStream方法返回的输入流来访问这些channel。 但是，在开始此进程之前，您可以使用方法redirectOutput和redirectError将标准输出和标准错误重定向到其他目标（例如文件）。

在下面的摘录中，方法redirectToFileTest将标准输入重定向到文件out.tmp，然后打印此文件：

```java
 public static void redirectToFileTest() throws IOException, InterruptedException {
    File outFile = new File("out.tmp");
    Process p = new ProcessBuilder("ls", "-la")
      .redirectOutput(outFile)
      .redirectError(Redirect.INHERIT)
      .start();
    int status = p.waitFor();
    if (status == 0) {
      p = new ProcessBuilder("cat" , outFile.toString())
        .inheritIO()
        .start();
      p.waitFor();
    }
  }
```

摘录将标准输出重定向到文件out.tmp。 它将标准错误重定向到调用进程的标准错误; 值Redirect.INHERIT指定子进程I / O源或目标与当前进程的相同。 对inheritIO（）方法的调用等效于redirectInput（Redirect.INHERIT）.redirectOuput（Redirect.INHERIT）.redirectError（Redirect.INHERIT）。

## 使用Stream过滤进程

ProcessHandle.allProcesses方法返回当前进程可见的所有进程的流。 您可以像过滤集合中的元素一样过滤此流的ProcessHandle实例。

在下面的摘录中，filterProcessesTest方法打印有关当前用户拥有的所有进程的信息，并按其父进程的进程ID进行排序：

```java
public class ProcessTest {

  // ...

  static void filterProcessesTest() {
    Optional<String> currUser = ProcessHandle.current().info().user();
    ProcessHandle.allProcesses()
        .filter(p1 -> p1.info().user().equals(currUser))
        .sorted(ProcessTest::parentComparator)
        .forEach(ProcessTest::showProcess);
  }

  static int parentComparator(ProcessHandle p1, ProcessHandle p2) {
    long pid1 = p1.parent().map(ph -> ph.pid()).orElse(-1L);
    long pid2 = p2.parent().map(ph -> ph.pid()).orElse(-1L);
    return Long.compare(pid1, pid2);
  }

  static void showProcess(ProcessHandle ph) {
    ProcessHandle.Info info = ph.info();
    System.out.printf("pid: %d, user: %s, cmd: %s%n",
      ph.pid(), info.user().orElse("none"), info.command().orElse("none"));
  } 

  // ...
}
```

请注意，allProcesses方法受本机操作系统访问控制的限制。 此外，由于所有进程都是异步创建和终止的，因此无法保证流中的进程处于活动状态，或者自调用allProcesses方法以来未创建其他进程。

## 使用onExit方法终止时处理进程

Process.onExit和ProcessHandle.onExit方法返回一个CompletableFuture实例，您可以使用该实例在进程终止时调度任务。 或者，如果您希望应用程序等待进程终止，则可以调用onExit().get()。

在下面的摘录中，方法startProcessesTest创建三个进程，然后启动它们。 然后，它在每个进程上调用onExit().thenAccept(onExitMethod); onExitMethod打印进程ID（PID），退出状态和进程输出。

```java
public class ProcessTest {

  // ...

  static public void startProcessesTest() throws IOException, InterruptedException {
    List<ProcessBuilder> greps = new ArrayList<>();
    greps.add(new ProcessBuilder("/bin/sh", "-c", "grep -c \"java\" *"));
    greps.add(new ProcessBuilder("/bin/sh", "-c", "grep -c \"Process\" *"));
    greps.add(new ProcessBuilder("/bin/sh", "-c", "grep -c \"onExit\" *"));
    ProcessTest.startSeveralProcesses (greps, ProcessTest::printGrepResults);      
    System.out.println("\nPress enter to continue ...\n");
    System.in.read();  
  }

  static void startSeveralProcesses (
    List<ProcessBuilder> pBList,
    Consumer<Process> onExitMethod)
    throws InterruptedException {
    System.out.println("Number of processes: " + pBList.size());
    pBList.stream().forEach(
      pb -> {
        try {
          Process p = pb.start();
          System.out.printf("Start %d, %s%n",
            p.pid(), p.info().commandLine().orElse("<na>"));
          p.onExit().thenAccept(onExitMethod);
        } catch (IOException e) {
          System.err.println("Exception caught");
          e.printStackTrace();
        }
      }
    );
  }
  
  static void printGrepResults(Process p) {
    System.out.printf("Exit %d, status %d%n%s%n%n",
      p.pid(), p.exitValue(), output(p.getInputStream()));
  }

  private static String output(InputStream inputStream) {
    String s = "";
    try (BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
      s = br.lines().collect(Collectors.joining(System.getProperty("line.separator")));
    } catch (IOException e) {
      System.err.println("Caught IOException");
      e.printStackTrace();
    }
    return s;
  }

  // ...
}
```

startProcessesTest方法的输出类似于以下内容。 请注意，进程可能以与启动顺序不同的顺序退出。

```shell
Number of processes: 3
Start 12401, /bin/sh -c grep -c "java" *
Start 12403, /bin/sh -c grep -c "Process" *
Start 12404, /bin/sh -c grep -c "onExit" *

Press enter to continue ...

Exit 12401, status 0
ProcessTest.class:0
ProcessTest.java:16

Exit 12404, status 0
ProcessTest.class:0
ProcessTest.java:8

Exit 12403, status 0
ProcessTest.class:0
ProcessTest.java:38
```

此方法调用System.in.read()方法以防止程序在所有进程退出之前终止（并运行thenAccept方法指定的方法）。

如果要在继续执行程序的其余部分之前等待进程终止，则调用onExit().get()：

```java
 static void startSeveralProcesses (
    List<ProcessBuilder> pBList, Consumer<Process> onExitMethod)
    throws InterruptedException {
    System.out.println("Number of processes: " + pBList.size());
    pBList.stream().forEach(
      pb -> {
        try {
          Process p = pb.start();
          System.out.printf("Start %d, %s%n",
            p.pid(), p.info().commandLine().orElse("<na>"));
          p.onExit().get();//会被阻塞在这里
          printGrepResults(p);          
        } catch (IOException|InterruptedException|ExecutionException e ) {
          System.err.println("Exception caught");
          e.printStackTrace();
        }
      }
    );
  }
```

ComputableFuture类包含各种方法，您可以调用这些方法来在进程退出时调度任务，包括以下内容：

- thenApply：类似于thenAccept，除了它采用Function类型的lambda表达式（一个返回值的lambda表达式）。
- thenRun：采用Runnable类型的lambda表达式（没有形式参数或返回值）。
- thenApplyAsyc：使用ForkJoinPool.commonPool()中的一个线程运行指定的Function。

因为ComputableFuture实现了Future接口，所以该类还包含同步方法：

- get(long timeout，TimeUnit unit)：如果需要，最多等待其参数指定的时间以完成进程。
- isDone：如果进程完成，则返回true。

## 控制对敏感过程信息的访问

进程信息可能包含敏感信息，例如用户ID，路径和命令参数。 使用安全管理器控制对进程信息的访问。

作为普通应用程序运行时，ProcessHandle具有与作为本机应用程序的其他进程相关的信息的相同操作系统权限。 但是，有关系统进程的信息可能不可用。

如果您的应用程序使用SecurityManager类来实现安全策略，那么为了使其能够访问进程信息，安全策略必须授予RuntimePermission（“manageProcess”）。 此权限允许本机进程终止并访问进程ProcessHandle信息。 请注意，此权限允许代码识别和终止它未创建的进程。

