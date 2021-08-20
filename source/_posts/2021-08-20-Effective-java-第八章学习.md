---
title: Effective-java-第八章学习
date: 2021-08-20 03:23:55
tags:
---
第八章：方法

<!--more-->

# 检查方法参数

failure atomicity(原子性失败): 失败的方法调用，应该使对象保持在被调用之前的状态。

如果一个方法在执行之前未检查参数，那么可能出现以下几种情况：

1. 方法执行失败，抛出令人困惑的异常。
2. 方法执行成功，但是返回的值不正确。
3. 方法执行成功，返回值也是正确的，但是有些对象的状态可能处于未知的状态，可能在未知的时刻，未知的调用情况下出现错误。

以上几点都违反了原子性失败，基于以上几点，所以在方法执行前一定要进行方法参数的检查。检查遵循如下约定：

1. 对于`public`和`protected`方法，可以在方法上使用`@throw`注释说明方法参数可能会引发何种异常，比如`IllegalArgumentException, IndexOutOfBoundsException, NullPointerException`，并且在参数未通过检查时，抛出这些异常。

2. 使用`@Nullable`和其他相似的注解，注解在方法参数上，指示方法参数是否可以为空等等。

3. 使用`Objects`工具类中提供的静态方法，进行方法参数的校验。

4. 对于不被暴露对外的方法(`private`)，可以控制在什么情况下被调用，可以确认什么时候被调用，什么样参数会被传入，可以使用断言`assert`，通常在测试时候可以使用`-ea`开启断言，在正式环境默认是不开启。

5. 对于有些参数在方法本身没有用到，只是保存起来以便以后使用，对于这种参数检查特别重要，因为如果不检查在之后使用中出现了错误，你无法知道是参数错误还是其他错误，举例：静态工厂方法`Lists.asList()`，构造函数等。

6. 参数检查也有例外，比如检查参数非常的消耗资源，或者有效性检查，隐式的在后续计算中完成，`Collections.sort(list)`(不用检查`list`中每个元素是否可以比较，`sort`方法会做出检查)，当然有可能隐式的检查抛出的异常和我们预期的异常是不同的，所以此时我们需要进行异常转换。

7. 并非参数检查越多越好，如果一个方法能接受所有的参数而且都能工作，非常的通用，那么对参数的限制当然是越少越好。

8. 参数的检查应该写在方法的注释中。

# 防御性的复制(拷贝)

## 不可变类

```java
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;
    /**
    * @param start the beginning of the period
    * @param end the end of the period; must not precede start
    * @throws IllegalArgumentException if start is after end
    * @throws NullPointerException if start or end is null
    */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(start + " after " + end);
        this.start = start;
        this.end = end;
    }
    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
    ... // Remainder omitted
}
```
上述代码看起来是不可变，实际上有两个问题:

1. `start()`,`end()`方法返回了类的成员变量的引用，调用者可以使用该引用改变成员变量。

2. 构造函数中将传入参数的引用赋给成员变量，如果该引用发生了变化，那么该类的实例也发生了变化。

修改：

1. 在构造函数中使用形参内容创建一个*copy*的中间对象，然后在将中间对象赋值给成员变量，这样即使传入的参数内容发生了改变也不在会影响该类的实例。
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
    throw new IllegalArgumentException(
    this.start + " after " + this.end);
}
```

2. `start()`,`end()` 方法返回该成员变量的*copy*对象。

```java
public Date start() {
    return new Date(start.getTime());
}
public Date end() {
    return new Date(end.getTime());
}
```

> ps: 在`jdk 8`以后的版本可以直接使用`java.time`下的`LocalDate`，因为`LocalDate`被设计成不变的。但是本例讨论的是不可变类在内部有可变成员变量时候如何保证不可变性。


`defensive copy`: 以上设计就是一种防御性复制(拷贝)。  

一个不可变的类中含有可变的成员变量，在构造函数中进行防御性复制是不可缺少的。  

__防御性拷贝应该是在参数检查之前进行，并且参数检查应该检查防御性拷贝之后对象，如果先进行参数检查，在进行拷贝，那么在参数检查之后拷贝进行之前这段*脆弱的窗口期*其他线程就可以改变参数的内容(虽然时间很短，但是多线程可能出现这种情况)，那么在进行拷贝时，可能就是一个不正确的值。在计算机安全社区，这种是一种攻击手段，叫做：__

> *TOCTOU* : tiem of check/time of use    

上述构造方法中并没有使用`date.clone()`方法获得一个拷贝，因为`Date`类可以被继承，所以实际的参数就可以是`Date`的子类的实例，在这种情况调用`clone`方法，并不能保证正确性和安全性。所以对于 __不被信任__ 可以被子类化的参数(参数的类是可以被继承，实现(implements))不要使用其`clone（）`方法。但是在`start()`,`end()`成员方法时可以使用`clone()`方法，因为，此时我们已经知道了`start`,`end`成员变量就是`Date`类型，是可以被信任的。


## 可变类

在编写可变类时候也要仔细的考虑是否可以接受客户端传入一个可变的参数，如果不能，那么就要进项防御性拷贝，避免客户端改变参数，影响该类的实例。  

其次在返回类的成员变量之前也要考虑是否可以接受客户端改变该类的成员变量的引用，或者改变其内容，如果不能，那么也要进行防御性拷贝。

如果使用不可变类的对象作为成员变量，那么就不必考虑防御性拷贝。  

简而言之，如果类具有从客户端得到或者返回到客户端的可变组件，类就必须考虑是否要进行防御性拷贝。



# 仔细的设计方法签名

1. 仔细的设计方法名称，应该是见名之意，容易理解，并且保持和类的内部或者包内部其他方法名称格式保持一致，而且方法名也不能过长。

2. 不要过于追求便利：每个方法都应该做到它的责任，太多的方法，使类或者接口难以维护，阅读，实现。

3. 避免过长的参数列表： 最多4个参数，特别使相同类型的长参数列表，调用的时候记不住顺序，类型又相同，非常容易传错参数，但是类型相同，编译又不会出错。如果参数太多，考虑查拆分为多个方法; 或者写一个辅助类，保存这些参数; 或者使用`builder`构造模式。

4. 使用接口类型作为参数类型：增加通用性。

5. 使用枚举代替boolean参数。


# 谨慎的进行方法重载


```java
// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
        }
}
```

灵魂拷问：输出什么? 结果是3次"Unknow Collection",因为在编译期间所有参数都是c.要调用哪个方法是在编译期间决定的，所以尽管实际参数类型是`set`,`list`也还是调用`classify(Collection<?>)`调用。


重载方法是静态选择的或者说是编译期间选择的，而重写方法的选择是动态的或者说是在运行期间的，选择依据是重写方法在运行时候被调用传入的参数类型。比如如下代码：

```java
class Wine {
    String name() { return "wine"; }
}
class SparklingWine extends Wine {
    @Override String name() { return "sparkling wine"; }
}
class Champagne extends SparklingWine {
    @Override String name() { return "champagne"; }
}
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne());
            for (Wine wine : wineList)
                System.out.println(wine.name());
    }
}
```
如上代码就会输出我们想要的结果：`wine, sparkling wine, champagne`  

__结论：对于具有相同数目参数的方法来说，应该尽量避免重载__


# 谨慎的使用可变参数


# 返回零长度的数组或者空集合而不是返回null

返回`null`值，调用的客户端必须要做`null`检查，实际上0长度的数组或者集合和`null`代表了一个意思，都是没有的意思，显然返回0长度的集合或者数组更加合适。

# 慎用`optionals`
`Optional`是`java 8`提供的一种容器，用来解决`npe`问题。实际上类似另一种`checked exception`操作，只是该操作不像抛出异常时需要爬栈(实际自定义异常也可以关闭爬栈)，比较优雅。

容器类型，比如`collections, maps, streams, arrays`和`optioal`自身(也可看做一个容器，只能存放至多一个元素)不应该使用`optional`进行包装。

如果一个方法可能返回`null`可能返回具体的值，且客户端要对没返回值进行特殊处理，那么因该定义返回`Optional<T>`  


不要使用`optional`包装一个原始类型的装箱类型，`java`提供了`OptionalInt, OptionalLong,OptionalDouble.`3种类来处理这种情况，`Boolean, Byte, Character, Short, and Float.`这些没有对应的类，所以最好是不用`optional`进行包装，因为会进行2曾包装，所以最好是直接返回原始类型的值。  


不要集合中使用`optional`,也不要将`optinal`作为`key`存入`Map`.如果这么做了，在检查元素是否在集合中就会很麻烦。得不偿失。

不要将`optional`作为成员变量使用。 

# 为所有导出的api元素编写注释

