# 参考资料
+ [https://github.com/google/guava/wiki](https://github.com/google/guava/wiki)
+ [https://guava.dev/releases/snapshot/api/docs/overview-summary.html](https://guava.dev/releases/snapshot/api/docs/overview-summary.html)

# 用户指南
Guava项目包含基于Java的项目中依赖的Google的几个核心库：集合，缓存，原语支持，并发库，通用批注，字符串处理，IO等。

+ Basic utilities: 更优雅地使用java语言
	+ Using and avoiding null: null 是不明确的，是容易导致错误的。许多 Guava 项目拒绝 null 并快速返回失败，而不是盲目地接收它。
	+ Preconditions: 更容易地测试方法的先决条件。
	+ Common object methods: 简化 Object 方法的实现，如 hashCode() 和 toString() 。
	+ Ordering: 强大的快速的比较器 Comparator 。
	+ Throwables: 简化传播异常和检查异常。
+ Collections: Guava 对 JDK 中 collections 地扩展。（Guava 最成熟和最受欢迎的部分）
	+ Immutable collections: 不可变集合，用于防御性编程、常量集合和提高效率。
	+ New collection types: 新集合类型，提供一些 JDK 中没有的集合类型: multisets, multimaps, tables, bidirectional maps 等等。
	+ Powerful collection utilities: 强大的集合操作方法，提供 java.util.Collections 中未提供的常见操作。
	+ Extension utilities: 集合扩展实用程序：编写集合装饰器？实现迭代器？可以使这些更容易。
+ Graphs: 一个为图结构数据建模的库，即实体和它们之间的关系。主要功能包括：
	+ Graph: 边是匿名实体的图，没有自己的身份或信息。
	+ ValueGraph: 边是具有关联的非唯一值的图。
	+ Network: 边是唯一对象的图形。
	+ 支持可变和不可变、有向和无向的图以及其他一些属性。
+ Caches: 本地缓存，做得好，并支持多种到期行为。
+ Functional idioms: 函数式语法，可以大大简化代码。
+ Concurrency: 强大、简单的抽象，使编写正确的并发代码更容易。
	+ ListenableFuture: Futures, 完成后进行回调。
	+ Service: 启动和关闭的东西，为你处理困难的状态逻辑。
+ Strings: 一些非常有用的字符串实用程序：拆分、连接、填充等等。
+ Primitives: 对原语类型（如int和char）的操作，不由JDK提供，包括某些类型的无符号变量。
+ Ranges: 强大的API，用于处理连续和离散的可比较类型的范围。
+ I/O: 简化的I/O操作，特别是对整个I/O流和文件的I/O操作，适用于Java5和6。
+ Hashing: 提供比 Object.hashCode() 更复杂的散列，包括布隆（Bloom）过滤器。
+ EventBus: 在组件之间发布订阅式通信，而不要求组件显式地彼此注册。
+ Math: 优化的，经过彻底测试的数学工具，不是由JDK提供的。
+ Reflection: 用于Java反射功能的实用程序。

# Basic Utilities 基础公用程序
## Using and avoiding null 空值处理
不小心使用空值会导致各种各样的错误。
通过研究Google代码库，我们发现大约95%的集合中不应该包含任何空值，而让这些集合快速失败而不是默默接受空值对开发人员是有帮助的。

此外，null是令人不快的模棱两可的。空返回值的含义并不明显。
例如，Map.get（key）可以返回空值，原因可能是映射中的值为空，或者该值不在映射中。空可以表示失败，可以表示成功，可以表示几乎任何事情。
使用空值以外的其他值可以使您的意思更清楚。

也就是说，有时使用空值是正确的。就内存和速度而言，空值很便宜，而且在对象数组中不可避免。
但在应用程序代码中，与库相反，它是混淆、困难和奇怪的错误以及令人不快的歧义的主要来源。
例如，当Map.get返回null时，它可能表示值不存在，或者值存在且为null。
最关键的是，空值不表示空值的含义。

由于这些原因，许多 Guava 的实用程序被设计成在存在空值的情况下快速失效，而不是允许使用空值，只要有一个 null 友好的解决方案可用。
此外，Guava 还提供了许多便利，使您在必须使用null时更容易使用，并帮助您避免使用null。

### Specific Cases 特殊情况
如果在 Set 中使用空值或在 Map 中使用null作为 key，请不要这样做；如果在查找操作期间显式地使用"null"，则会更清楚。

如果在 Map 中使用null作为 value，请省略该项；保留一组单独的非null键（或null键）。
很容易混淆映射包含键项（值为空）和映射没有键项的情况。最好将这些键分开，并考虑当与键关联的值为空时对应用程序意味着什么。

如果在 List 中使用null，如果列表是稀疏的，是否可以使用Map<Integer，E>。这实际上可能更有效，并且可能更准确地匹配应用程序的需求。

考虑是否有可以使用的自然“空对象”，有时可以，并不总是这样。
例如，如果它是一个枚举，则添加一个常量来表示您在这里所期望的空值。
例如，java.math.RoundingMode有一个不必要的值来指示“不进行舍入，如果需要舍入，则抛出异常”

如果您确实需要空值，并且遇到了空的恶意集合实现的问题，请使用其他实现。
例如，使用Collections.unmodifiableList（Lists.newArrayList（））代替 ImmutableList。

### Optional Optional
程序员使用空值的许多情况都是为了表示某种缺失：可能在有值的地方，没有值，或者找不到值。
例如，当找不到键的值时，Map.get返回null。

Optional<T>是用非空值替换可为空的T引用的一种方法。
一个 Optional 可以包含一个非空的T引用（在这种情况下，我们说引用是“存在的”），或者 Optional 可以不包含任何内容（在这种情况下，我们说引用是“不存在的”）。
但从来没有人说它“包含空值”

```
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

Optional 并不是直接从任何其他编程环境中的任何现有的“选项”或“可能”构造的直接模拟，尽管它可能有一些相似之处。
我们在这里列出一些最常见的可选操作。

#### Making an Optional 实例化Optional
每一个都是 Optional 的静态方法。
+ Optional.of(T) 设置一个包含给定非空值的Optional，或者在为空值时快速失败。
+ Optional.absent() 返回某种类型的不存在的Optional。	
+ Optional.fromNullable(T) 将给定的可能为空的引用转换为Optional，将非空视为存在，将空视为不存在。

#### Query methods 查询方法
每种方法都是针对特定可选<T>值的非静态方法。
+ boolean isPresent()	如果此可选值包含非空实例，则返回true。
+ T get()	返回包含的T实例，该实例必须存在；否则，抛出IllegalStateException。
+ T or(T)	返回此可选值中的当前值，如果没有，则返回指定的默认值
+ T orNull() 返回此可选值中的当前值，如果没有，则返回空值。fromNullable的逆运算。
+ Set<T> asSet() 返回包含此可选实例（如果有实例）的不可变单例集，或者返回空的不可变集。

### What's the point? 有什么意义？
除了通过给null起一个名字来增加可读性之外，Optional的最大优点是它的防白痴性。
如果你想让你的程序编译的话，它会强迫你积极地考虑不存在的情况，因为你必须积极地打开可选的并解决这个问题。
Null使得简单地忘记事情变得令人不安，尽管FindBugs有帮助，但我们认为它并没有很好地解决这个问题。

当您返回的值可能是“present”也可能不是“present”时，这一点尤其重要。
您（和其他人）更可能忘记另一个.method（a，b）返回空值，而不是您在实现other.method时忘记a可以是空值。
返回Optional使调用者无法忘记这种情况，因为他们必须自己展开对象以便编译代码。

### Convenience methods 方便方法
每当您希望用默认值替换空值时，请使用 MoreObjects.firstNonNull(T, T)。
如方法名所示，如果两个输入都为空，则它将以NullPointerException快速失败。
如果您使用的是Optional，则有更好的替代方案：first.or(second)。

字符串中提供了一些处理可能为空字符串值的方法。具体来说，我们提供了恰当的名称：
+ emptyToNull(String)
+ isNullOrEmpty(String)
+ nullToEmpty(String)

我们要强调的是，这些方法主要用于与不愉快的api接口，这些api等同于 null 字符串和 empty  字符串。
每次你写的代码把 null 字符串和 empty 字符串合并在一起，Guava 团队都会哭泣。
如果 null 字符串和 empty 字符串意味着不同的活动，那就更好了，但是把它们当作同一个东西是一种令人不安的常见代码气味。

## Preconditions 条件检查
Guava 提供了许多先决条件检查实用程序。我们强烈建议静态导入这些文件。

每种方法都有三种变体：
1. 没有额外的参数。抛出异常时不会显示错误消息。
2. 一个额外的 Object 参数。抛出异常时会显示错误消息，错误消息为object.toString()。
3. 一个额外的 String 参数，以及任意数量的附加对象参数。它的行为类似于printf，但是为了GWT的兼容性和效率，它只允许使用%s指示符。
	+ 注意：checkNotNull, checkArgument and checkState 有大量的方法重载，采用了原始数据类型参数和对象参数的组合，而不是 可变参数 数组 
		-- 这允许上面的调用避免绝大多数情况下的原始数据装箱和 可变参数 数组分配。

第三种变体的示例：
```
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s >= %s", i, j);
```

我们倾向于使用 Guava 的先决条件检查，而不是Apache Commons中的类似实用程序。有以下几个原因：
静态导入后，Guava 方法清晰明了。checkNotNull清楚地表明正在做什么，以及将抛出什么异常。
checkNotNull 在验证后返回其参数，允许在构造函数中使用简单的一行程序：this.field=checkNotNull(field)。
简单的，使用可变参数输出异常消息。（这也是我们建议继续对对象使用checkNotNull的原因而不是 Objects.requireNonNull)

我们建议您将预处理条件拆分成不同的行，这可以帮助您找出调试时哪个预处理条件失败。
另外，您应该提供有用的错误消息，当每个检查都在自己的行上时，这会更容易。

## Ordering 排序
### Example 示列
```
assertTrue(byLengthOrdering.reverse().isOrdered(list));
```

### Overview 概述
Ordering 是Guava的方便快速的 Comparator 类，它可以用来构建复杂的比较器并将它们应用于对象集合。
在其核心，Ordering 实例只不过是一个特殊的 Comparator 实例。
Ordering 简单地采用依赖于 Comparator 的方法（例如Collections.max），并使它们可用作实例方法。
此外，Ordering 类提供链式方法调用来调整和增强现有的比较器。

### Creation 创建
常用 Ordering 由静态方法创建。
+ natural() 使用可比较类型的自然顺序。
+ usingToString() 通过ToString（）返回的对象，比较它们的字符串表示的字典顺序排序。

将一个已存在的 Comparator 变成一个 Ordering：
```
Ordering.from(Comparator).
```

但创建自定义 Ordering 的更常见方法是完全跳过 Comparator 比较器，而直接扩展排序抽象类：
```
Ordering<String> byLengthOrdering = new Ordering<String>() {
  public int compare(String left, String right) {
    return Ints.compare(left.length(), right.length());
  }
};
```

### Chaining 链式调用
可以包装给定的 Ordering 以获取派生的 Ordering。一些最常用的变体包括：
+ reverse() 返回相反的 Ordering。
+ nullsFirst() 返回在非空元素之前对空元素进行排序，否则其行为与原始 Ordering 相同的 Ordering。
+ compound(Comparator) 返回使用指定 Ordering “断开连接” 的 Ordering。
+ lexicographical() 返回按字母顺序排列迭代项的排序。
+ onResultOf(Function) 返回一个Ordering，该 Ordering 通过将函数应用于值，然后使用原始 Ordering 比较结果来对值进行排序。

例如，假设你想要一个类的比较器：
```
class Foo {
  @Nullable String sortedBy;
  int notSortedBy;
}
```

可以处理sortedBy的空值。以下是建立在链接方法之上的解决方案：
```
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
  public String apply(Foo foo) {
    return foo.sortedBy;
  }
});
```

当解读一系列的Ordering命令调用时，是从右到左“向后”工作。
上面的示例表示，通过查找字段值 sortedBy 对 Foo 实例进行排序，首先将所有空的sortedBy值移动到顶部，然后按自然字符串顺序对剩余值进行排序。
之所以出现这种倒序，是因为每个链式调用都将先前的顺序“包装”成一个新的顺序。
（“向后”规则的例外：对于 compound 调用链，从左到右读取。为了避免混淆，避免将 compound 调用与其他链式调用混合。）

比较长的调用链可能很难理解。我们建议将调用链限制为大约三个，如上面的示例所示。
即便这样，您仍然可以通过分离中间对象（如函数实例）来简化代码：
```
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction);
```

### Application 使用
Guava提供了许多方法来使用 Ordering 操作或检查一些值或集合，我们在这里列出一些。
+ greatestOf(Iterable iterable, int k) 根据此Ordering，按从大到小的顺序返回指定iterable的k个最大元素。不一定稳定。
+ isOrdered(Iterable) 根据此Ordering，测试指定的Iterable是否处于非递减顺序。
+ sortedCopy(Iterable) 将指定元素的排序副本作为列表返回。
+ min(E, E) 根据此顺序返回其两个参数中的最小值。如果值比较为相等，则返回第一个参数。
+ min(E, E, E, E...) 根据此顺序返回其参数的最小值。如果存在多个最小值，则返回第一个。
+ min(Iterable) 返回指定Iterable的最小元素。如果Iterable为空，则抛出NoSuchElementException。

## Object methods 对象常用方法
### equals
当对象字段可以为空时，实现object.equals可能会很麻烦，因为您必须分别检查是否为空。
使用Objects.equal可以以一种对null敏感的方式执行equals检查，而不必冒NullPointerException的风险。
```
Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```

### hashCode
散列对象的所有字段应该更简单。Guava 的 Objects.hashCode（Object…）为指定的字段序列创建合理的、顺序敏感的哈希。
使用Objects.hashCode（field1，field2，…，fieldn）而不是手工构建哈希。

### toString
一个好的toString方法在调试中是非常宝贵的，但是编写起来很麻烦。
使用MoreObjects.toStringHelper（）轻松创建一个有用的toString。一些简单的例子包括：
```
// Returns "ClassName{x=1}"
MoreObjects.toStringHelper(this)
   .add("x", 1)
   .toString();

// Returns "MyObject{x=1}"
MoreObjects.toStringHelper("MyObject")
   .add("x", 1)
   .toString();
```

### compare/compareTo ComparisonChain
实现一个比较器，或者直接实现一个可比较的接口，可能会很痛苦。考虑：
```
class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;

  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}
```

这段代码很容易出错，很难扫描错误，而且冗长得令人不快。我们应该能做得更好。
为此，Guava 提供 ComparisonChain。

ComparisonChain 执行“惰性”比较：它只执行比较，直到找到一个非零的结果，然后忽略进一步的输入。
```
public int compareTo(Foo that) {
 return ComparisonChain.start()
	 .compare(this.aString, that.aString)
	 .compare(this.anInt, that.anInt)
	 .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
	 .result();
}
```

## Throwables 异常抛出
Guava 的异常抛出实用程序可以简化处理异常。
（略）

# Collections
## Immutable collections 不可变集合
### Example 示列
```
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
  "red",
  "orange",
  "yellow",
  "green",
  "blue",
  "purple");

class Foo {
  final ImmutableSet<Bar> bars;
  Foo(Set<Bar> bars) {
    this.bars = ImmutableSet.copyOf(bars); // defensive copy!
  }
}
```
Why?
Immutable objects have many advantages, including:

Safe for use by untrusted libraries.
Thread-safe: can be used by many threads with no risk of race conditions.
Doesn't need to support mutation, and can make time and space savings with that assumption. All immutable collection implementations are more memory-efficient than their mutable siblings. (analysis)
Can be used as a constant, with the expectation that it will remain fixed.

New collection types
Multiset
Multimap
BiMap
Table
ClassToInstanceMap
RangeSet
RangeMap
Utility Classes
Iterables
Lists
Sets
Maps
Multisets
Multimaps
Tables
Extension Utilities
Forwarding Decorators
PeekingIterator
AbstractIterator
Graphs
Definitions
Capabilities
Graph types
Graph
ValueGraph
Network
Building graph instances
Builder constraints vs. optimization hints
Mutable and Immutable graphs
Mutable* types
Immutable* implementations
Graph elements (nodes and edges)
Library contracts and behaviors
Mutation
equals(), hashCode() and graph equivalence
Accessor methods
Synchronization
Element objects
Notes for implementors
Storage models
Accessor behavior
Abstract* classes
Code examples
FAQ
Caches
Applicability
Population
Eviction
Removal Listeners
Refresh
Timed Eviction
Size Caps
Garbage Collection
Explicit Removals
Features
Statistics
Interruption
Functional Idioms
Obtaining
Using Predicates
Using Functions
Concurrency
ListenableFuture
Service
Using
Implementations
Strings
Joiner
Splitter
CharMatcher
Charsets
CaseFormat
Networking
InternetDomainName
Primitives
Primitive arrays
General utilities
Byte conversion
Unsigned support
Ranges
Building
Operations
Discrete Domains
I/O
Closing Resources
Hashing
BloomFilter
EventBus
Math
Integral
Overflow Checking
Floating Point
Reflection
TypeToken
Invokable
Dynamic Proxies
ClassPath