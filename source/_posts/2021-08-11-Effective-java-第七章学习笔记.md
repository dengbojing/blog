---
title: Effective-java-第七章学习笔记
date: 2021-08-11 03:02:51
tags: java
---

第七章：Lambdas 表达式 和 Streams流处理

<!--more-->


# 使用`Lambads`优先于匿名内部类

函数式接口： 只拥有一个抽象方法的接口(interface)被称作函数式接口。

函数对象：只具有一个方法的接口的实例，代表一种方法或者具体的执行动作。


`Lambad`表达式减少了匿名内部类的样板代码，比如: 
```java
Thread  thread = new Thread(new Runnable() {
            @Override
            public void run() {
                //do sth
            }
        });
```
使用`Lambads`代替：
```java
Thread  thread = new Thread(() -> {//do sth});
```
当明白了`Lambads`的语法，会觉得上述写法非常简洁，也非常直观的知道这个线程的作用.

1. 当使用`Lambads`时，通常省略参数类型，除非参数类型的指定能让程序看起来更加清晰，如果编译器发出警告或者编译错误无法推断参数类型，此时你应该明确的指定参数类型。编译器的类型推断是根据传入参数，或者返回值的泛型类型来确定，所以一定要使用泛型来代替原始类型。

2. `Lambads`最好不要超过3行，超过3行可读性就会非常差，超过3行就要进行重构

3. `Lambads`没有名称和很好的文档注释，如果表达式不能自我表达，自我描述，或者超过行数限制，都不应该写`Lambads`

4. `Lambads`中`this`指代的是包含表达式的对象，匿名内部类中`this`代表该匿名内部类的实例。

6. 使用泛型更好的实现枚举的constant-specific方法。
```java

public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);
    private final String symbol;
    private final DoubleBinaryOperator op;
    Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = op;
}
```


# 使用方法引用优先于`Lambads`

如果方法引用不能使代码更加清晰，降低了可读性，那么则应该使用`lambad`表达式。


# 优先使用标准的函数式接口


1. 如果有标准的函数式接口可以完成需求，那么就应该优先使用，而不是专门构建一个函数式接口。`java 8`内置了很多函数式接口在`java.util.function`包下，优先使用这些函数式接口。如果有以下情况除外：  
  1.1. 需要一个通用的，具有描述性的名字的函数式接口.
  1.2. 拥有很强的约束关系.
  1.3. 讲受益于自定义的默认方法.


2. 不要试图使用带有包装类型的基础函数式接口代替原始类型的函数式接口。

3. 函数式接口必须要使用`@FunctionalInterface`注解。



# 谨慎的使用`Streams·

`Stream`概念包含一组数据(`Steam`)和作用于数据上的一个或者多个操作(`Stream pipline`).

`Stream pipline`分为两种操作：中间操作(`intermediate operations`) 和 终止操作(`terminal operation`),中间操作可以有一个或者多噶，而终止操作只能有一个，但是只有在调用了有了终止操作，整个`Stream pipline`才会被触发，如果没有终止操作，那么处理流的代码永远不会被执行。

流的处理默认是串行的，只有当调用流的`parallel`方法时候流的处理才会被并行处理，但是很少这么做，并行处理需要注意很多地方，且并行处理并不一定都会使处理速度加快。

过度是使用流操作也会是代码难以阅读和维护，比如: 
```java
// Overuse of streams - don't do this!
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
            groupingBy(word -> word.chars().sorted()
            .collect(StringBuilder::new,
            (sb, c) -> sb.append((char) c),
            StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
        }
    }
}
```
这段代码里面的`gourpingBy`函数的代码读对于没怎么使用过流的人来说读起来非常吃力。
建议讲`groupingBy`单独封装一个方法，单独封装一个方法，可以使用有涵义的方法名和良好的说明注释，这样可以使整个代码可读性和维护性有很大的提升。


由于`lambdd`表达式没有显示的参数类型，所以参数的名称就非常重要，好的参数名称对于可读性来说是非常重要的，尽量不要使用单个字母来命名，使用单词或者合成单词来命名。  


避免使用流来处理`char`数据，书中举例是因为`char`字符在输出时会使用对应的`int`值。


除非有必要重构`for`循环代码为`Stream`，否则你应该保持克制，并不是所有的循环改为`Stream`都拥有很好的可读性和可维护性。复杂的任务进行重构时可能会带来过度使用`Stream`的问题。从而使代码的可读性和维护性降低。


`Stream`适用的场景：
- 对元素进行一些列的变换
- 过滤元素
- 合并元素(添加，合并)
- 元素聚集(合并为一个map,list等)
- 按条件搜索元素



# 使用`Streawms`中无副产物的函数

纯函数(pure function): 输出只依赖输入，不依赖其他任何阶段的状态，也不会改变其他任何东西的状态。

`forEach`函数应该只用作输出`Stream`最后计算出来的结果，不应该执行具体的计算，偶尔可以用作其他目的，比如添加`Stream`计算的结果到一个集合中。

应该静态导入`Collectors`类，提高`Stream`最终的可读性，永远不要使用`Collectors.collect(counting())`等其他方法(`suming,averaging,summarizing,filtering,reducing,mapping,flatmapping`等)，这些方法是为了下游的`Stream`设计，应该使用`Stream`自带的方法


# 优先使用集合作为返回值

问题很简单，因为所有集合都实现了`Iterable`接口，整个更加通用。

如果返回的数据集特别大，不建议使用现有的集合类，而是自己实现`AbstractList`.

如实现一个给定集合的全排列： 
```java
//The power set of {a, b, c} is {{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}.
public class PowerSet {

    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }
            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }
            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```


# `Stream`并行化的建议

如果使用`Stream.iterate` 和 中间操作`limit`,并行化流并不会得到性能提升。

在`ArrayList, HashMap, HashSet, and ConcurrentHashMap instances; arrays; int ranges; and long ranges`上使用并行流会有显著的性能提升。因为这些数据结构统一，可以被精确的分割，有利于并行化。另一重要的因素是这些数据结构有着很好的局部引用，数据引用在内存连续的，虽然这些也引用的数据对象在内存上是不连续的，不利于并行处理的；内存的连续性(局部的引用性)是并行处理的最关键因素，如果没有内存连续性的，线程大部分是空闲的，在等待cpu将内存数据获取到cpu缓存中。最好的内存连续性数据结构就是原始数据类型的数组，他们的分配都是在连续的内存上。

使用并行流不止有可能导致性能降低，还有可能导致结果不正确(使用`forEachOrder`)和活锁(`limit`).

只用在适当的情况下，使用并行流才会提升性能，所以建议是不要并行化，除非你有很好的理由非要这么做。