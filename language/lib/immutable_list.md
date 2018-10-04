# 创建不可变Lists,Sets和Maps

在JDK 9中添加的List，Set和Map接口上的便捷静态工厂方法使您可以轻松创建不可变List，Set和Map。

如果一个对象的状态在构造后不能改变，则该对象被认为是不可变的。 在创建集合的不可变实例后，只要对其存在引用，它就会保留相同的数据。

如果使用这些方法创建的集合包含不可变对象，那么它们在构造后自动是线程安全的。 因为构造函数不需要支持其他变化，所以它们可以更加节省空间。 不可变集合实例通常比它们的可变对应物消耗更少的内存。

正如关于不可变性中所讨论的，不可变集合可以包含可变对象，如果是，则集合既不是不可变的也不是线程安全的。

## 用例

不可变方法的常见用例是一个从已知值初始化的集合，它永远不会更改。如果您的数据不经常更改，也请考虑使用这些方法。

为获得最佳性能，不可变集合存储一个永不改变的数据集。但是，即使您的数据可能会发生变化，您也可以利用性能和节省空间的优势。即使您的数据偶尔会发生变化，这些集合也可能提供比可变集合更好的性能。

如果您有大量值，可以考虑将它们存储在HashMap中。如果您不断添加和删除条目，那么这是一个不错的选择。但是，如果你有一组永远不会改变或很少改变的值，并且你从那个集中读取了很多，那么不可变的Map是一个更有效的选择。如果频繁读取数据集，并且值很少发生变化，那么您可能会发现整体速度更快，即使您在值发生变化时包含了破坏和重建不可变映射的性能影响。

## 语法

这些新集合的API很简单，特别是对于少量元素。

### List不可变静态工厂方法

List.of静态工厂方法提供了一种创建不可变列表的便捷方法。

List是有序集合，通常允许重复元素。 不允许空值。

这些方法的语法是：

```java
List.of()
List.of(e1)
List.of(e1, e2)         // fixed-argument表单最多可重载10个元素
List.of(elements...)   //varargs表单支持任意数量的元素或数组
```

在jdk 1.8:

```java
List<String> stringList = Arrays.asList("a", "b", "c");
stringList = Collections.unmodifiableList(stringList);
```

同样的效果在jdk 1.9可以如下实现:

```java
List stringList = List.of("a", "b", "c");
```

更多细节请查看[这里](https://docs.oracle.com/javase/10/docs/api/java/util/List.html#immutable)

### Set不可变静态工厂方法

Set.of静态工厂方法提供了一种创建不可变集的便捷方法。

集合是不包含重复元素的集合。 如果检测到重复条目，则抛出IllegalArgumentException。 不允许空值。

语法如下:

```java
Set.of()
Set.of(e1)
Set.of(e1, e2)         // fixed-argument表单最多可重载10个元素
Set.of(elements...)   // varargs表单支持任意数量的元素或数组
```

在jdk 1.8中如下:

```java
Set<String> stringSet = new HashSet<>(Arrays.asList("a", "b", "c"));
stringSet = Collections.unmodifiableSet(stringSet);
```

可在jdk 1.9中实现如下:

```java
Set<String> stringSet = Set.of("a", "b", "c");
```

更多细节看[这里](https://docs.oracle.com/javase/10/docs/api/java/util/Set.html#immutable);

## map的不可变静态方法

Map.of和Map.ofEntries静态工厂方法提供了一种创建不可变映射的便捷方法。

Map不能包含重复的键; 每个key最多可以映射一个值。 如果检测到重复key，则抛出IllegalArgumentException。 空值不能用作Map key或value。

语法如下:

```java
Map.of()
Map.of(k1, v1)
Map.of(k1, v1, k2, v2)    // ffixed-argument表单最多可重载10个元素
Map.ofEntries(entry(k1, v1), entry(k2, v2),...) //  varargs表单支持任意数量的Entry对象或数组
```

jdk 1.8:

```java
Map<String, Integer> stringMap = new HashMap<String, Integer>(); 
stringMap.put("a", 1); 
stringMap.put("b", 2);
stringMap.put("c", 3);
stringMap = Collections.unmodifiableMap(stringMap);
```

Jdk 1.9:

```java
Map stringMap = Map.of("a", 1, "b", 2, "c", 3);
```

### 具有任意对数的Map

如果您有超过10个键值对，则使用Map.entry方法创建映射条目，并将这些对象传递给Map.ofEntries方法。 例如：

```java
import static java.util.Map.entry;
Map <Integer, String> friendMap = Map.ofEntries(
   entry(1, "Tom"),
   entry(2, "Dick"),
   entry(3, "Harry"),
   ...
   entry(99, "Mathilde"));
```

更多细节查看[这里](https://docs.oracle.com/javase/10/docs/api/java/util/Map.html#immutable)

## 创建集合的不可变副本

让我们考虑通过添加元素并对其进行修改来创建集合的情况，然后在某些时候，您需要该集合的不可变快照。 使用JDK 10中添加的copyOf方法系列创建副本。

例如，假设您有一些代码可以从多个位置收集元素：

```java
List<Item> list = new ArrayList<>();
list.addAll(getItemsFromSomewhere());
list.addAll(getItemsFromElsewhere());
list.addAll(getItemsFromYetAnotherPlace());
```

使用List.of方法创建不可变集合很不方便。 这样做需要创建一个正确大小的数组，将列表中的元素复制到数组中，然后调用List.of（array）来创建不可变的快照。 相反，使用copyOf静态工厂方法一步完成：

```java
 List<Item> snapshot = List.copyOf(list); 
```

Set和Map有相应的静态工厂方法，名为Set.copyOf和Map.copyOf。

如果原始集合是可变的，则copyOf方法创建一个不可变集合，该集合是原始集合的副本。也就是说，结果包含与原始元素相同的所有元素。如果在原始集合中添加或删除元素，则不会影响副本。

如果原始集合已经是不可变的，则copyOf方法只返回对原始集合的引用。制作副本的目的是将返回的集合与原始集合的更改隔离开来。但如果原始集合无法更改，则无需复制它。

在这两种情况下，如果元素是可变的，并且元素被修改，则该更改会导致原始集合和副本看起来已更改。

## 从Streams创建不可变集合

Streams库包括一组称为收集器的终端操作。 收集器通常用于创建包含流的元素的新集合。 从JDK 10开始，java.util.stream.Collectors类具有Collectors，它们从流的元素创建新的不可变集合。

如果要保证返回的集合是不可变的，则应使用toUnmodifiable-collector之一。 这些collectors是：

```java
  Collectors.toUnmodifiableList()
   Collectors.toUnmodifiableSet()
   Collectors.toUnmodifiableMap(keyMapper, valueMapper)     
   Collectors.toUnmodifiableMap(keyMapper, valueMapper, mergeFunction)
```

例如，要转换源集合的元素并将结果放入不可变集合，可以执行以下操作：

```java
   Set<Item> immutableSet =
      sourceCollection.stream()
                      .map(...) 
                      .collect(Collectors.toUnmodifiableSet());
```

这些收集器在概念上类似于它们对应的toList，toSet和相应的两个toMap方法，但它们具有不同的特征。 具体来说，toList，toSet和toMap方法不保证返回的集合是可变的还是不可变的。

## 随机迭代顺序

Set元素和Map Key的迭代顺序是随机的：它可能与一个JVM运行到下一个运行不同。 这是故意的 - 它使您更容易识别依赖于迭代顺序的代码。 有时，对迭代顺序的依赖会无意中蔓延到代码中，并导致难以调试的问题。

在重新启动jshell之前，您可以看到迭代顺序是如何相同的。

```shell
jshell> Map stringMap = Map.of("a", 1, "b", 2, "c", 3);
stringMap ==> {b=2, c=3, a=1}

jshell> Map stringMap = Map.of("a", 1, "b", 2, "c", 3);
stringMap ==> {b=2, c=3, a=1}

jshell> /exit
|  Goodbye

C:\Program Files\Java\jdk-9\bin>jshell
|  Welcome to JShell -- Version 9-ea
|  For an introduction type: /help intro

jshell> Map stringMap = Map.of("a", 1, "b", 2, "c", 3);
stringMap ==> {a=1, b=2, c=3}
```

由Set.of，Map.of和Map.ofEntries方法创建的集合实例是迭代顺序随机化的唯一集合实例。 集合实现的迭代排序（如HashMap和HashSet）保持不变。

## 关于不可变

JDK 9中添加的便利工厂方法返回的集合通常是不可变的。 任何从这些集合中添加，设置或删除元素的尝试都会导致抛出UnsupportedOperationException。

那些不是“不可变的持久的”或“functional”集合。 如果您正在使用其中一个集合，那么您可以对其进行修改，但是当您这样做时，将返回一个新的更新集合，该集合可能共享第一个集合的数据结构。

不可变集合的一个优点是它是自动线程安全的。 创建集合后，您可以将其交给多个线程，并且它们都将看到一致的视图。

但是，不可变对象的集合与不可变集合(元素是对象)不同。 如果包含的元素是可变的，那么这可能导致集合行为不一致或使其内容看起来改变。

让我们看一个不可变集合包含可变元素的示例。 使用jshell，使用ArrayList类创建两个String对象列表，其中第二个列表是第一个列表的副本。 删除了琐碎的jshell输出。

```shell
jshell> List<String> list1 = new ArrayList<>();
jshell> list1.add("a")
jshell> list1.add("b")
jshell> list1
list1 ==> [a, b]

jshell> List<String> list2 = new ArrayList<>(list1);
list2 ==> [a, b]
```

接下来，使用List.of方法创建指向第一个列表的ilist1和ilist2。 如果您尝试修改ilist1，则会看到异常错误，因为ilist1是不可变的。 任何修改尝试都会引发异常。

```shell
jshell> List<List<String>> ilist1 = List.of(list1, list1);
ilist1 ==> [[a, b], [a, b]]

jshell> List<List<String>> ilist2 = List.of(list2, list2);
ilist2 ==> [[a, b], [a, b]]

jshell> ilist1.add(new ArrayList<String>())
|  java.lang.UnsupportedOperationException thrown:
|        at ImmutableCollections.uoe (ImmutableCollections.java:70)
|        at ImmutableCollections$AbstractImmutableList.add (ImmutableCollections
.java:76)
|        at (#10:1)
```

但是如果修改原始列表1，则ilist1和ilist2不再相等。

```shell
jshell> list1.add("c")
jshell> list1
list1 ==> [a, b, c]
jshell> ilist1
ilist1 ==> [[a, b, c], [a, b, c]]

jshell> ilist2
ilist2 ==> [[a, b], [a, b]]

jshell> ilist1.equals(ilist2)
$14 ==> false
```

### 不可变和不可修改不一样

不可变集合的行为与Collections.unmodifiable ...包装器的行为相同。 但是，这些集合不是包装器 - 这些是由类实现的数据结构，其中任何修改数据的尝试都会导致抛出异常。

如果您创建一个List并将其传递给Collections.unmodifiableList方法，那么您将获得一个不可修改的视图。 基础list仍然是可修改的，并且通过返回的List可以看到对它的修改，因此它实际上不是不可变的。

要演示此行为，请创建一个List并将其传递给Collections.unmodifiableList。 如果您尝试直接添加到该List，则会引发异常。

```java
jshell> List<String> unmodlist1 = Collections.unmodifiableList(list1);
unmodlist1 ==> [a, b, c]

jshell> unmodlist1.add("d")
|  java.lang.UnsupportedOperationException thrown:
|        at Collections$UnmodifiableCollection.add (Collections.java:1056)
|        at (#17:1)
```

但是，如果更改原始list1，则不会生成错误，并且已修改unmodlist1列表。

```shell
jshell> list1.add("d")
$19 ==> true
jshell> list1
list1 ==> [a, b, c, d]

jshell> unmodlist1
unmodlist1 ==> [a, b, c, d]
```

## 空间效率

便利工厂方法返回的集合比它们的可变等价物更节省空间。

这些集合的所有实现都是隐藏在静态工厂方法后面的私有类。 调用它时，静态工厂方法根据大小选择实现类。 数据可以存储在紧凑的基于字段或基于数组的布局中。

让我们看看两个替代实现所消耗的堆空间。 首先，这是一个包含两个字符串的不可修改的HashSet：

```java
Set<String> set = new HashSet<>(3);   // 3 buckets
set.add("silly");
set.add("string");
set = Collections.unmodifiableSet(set);
```

该集包括六个对象：不可修改的包装器; HashSet，包含HashMap; hash表（数组）; 和两个Node实例（每个元素一个）。 在典型的VM上，每个对象有一个12字节的标头，总开销为96字节+ 28 * 2 = 152字节。 与存储的数据量相比，这是一个大量的开销。 另外，访问数据不可避免地需要多个方法调用和指针解引用。

相反，我们可以使用Set.of实现该集：

```java
Set<String> set = Set.of("silly", "string");
```

因为这是一个基于字段的实现，所以该集包含一个对象和两个字段。 开销是20个字节。 无论是在固定开销方面还是在每个元素方面，新集合都消耗更少的堆空间。

不需要支持可变也有助于节省空间。 此外，改进了引用的局部性，因为保存数据所需的对象较少。