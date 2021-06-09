---
title: Effective-java-第三章学习笔记
date: 2021-03-08 20:01:55
tags: java, 读书笔记
categories: java
---
第三章: 重写Object类中的几个方法
<!--more-->
# 引言

这一章主要讲的是Object类中的几个方法该如何重写.实际上本章内容没有在开发实践中并不会出现,一般开发人员都用`lombok`或者其他工具类实现了Object中的方法,很少遇到自己重写的情况,而且在正常逻辑上也不会违背文中所说的注意点.

# 正文

## 重写equals方法

### 在一下3种情况不应该重写equals

1. 该类不是一个值类,是代表活动实体的类,比如`Thread`.
2. 该类没必要提供逻辑相等,比如单例类,只会产生一个实例,`Object`类提供的地址相等的`equals`方法已经足够;再比如一些工具类`xxxUtil`,`xxxxBuilder`等,实际上这些类有时候无法实例化,所以没必要重写`equals`方法
3. 该类是私有的或者包级私有的,**可以确保`equals`方法不会被调用.(这句才是重点)**

### 重写equals时遵循的规范 

1. `自反性(reflexive)`: 对于任何非`null`的引用值`x`, `x.equals(x)`必须返回`true`.
2. `对称性(symmetric)`: 对于任何非`null`的引用值`x`和`y`, 当且仅当`y.equals(x)`返回`true`, `x.equals(y)`必须返回`true`.
3. `传递性(transitive)`: 对于任何非`null`的引用值`x,y,z`, 如果`x.equals(y)`返回`true`,`y.equals(z)`返回`true`, 那么`x.equalis(z)`也返回`true`
4. `一致性(consistent)`: 对于任何非`null`的引用值`x,y`,只要对象中的信息没有被修改过,那么多次调用`x.equals(y)`的结果必然一致.
5. 任何对象`equals(null)`必然返回false.

以上规范,看起来挺复杂,实际上属于一种自然而然的做法,在重写`equals`的时候,很自然的就做到了.最好的方法就是使用第三方库来重写`equals`省事,还不会出错,除非你有非常特别的理由要自己手动重写.

### 重写equals时注意事项

1. 不要依赖不可靠资源,比如`java.net.URL`中主机`ip`地址的比较,可能会存在`host`不变但是`ip`变了.
2. 优先比较最有可能不一致的字段,或者开销比较低的字段,最理想是二者兼备,有这些字段组成关键字段.
3. 重写`equals`时总是重写`hashCode`方法.
4. 不要让`equals`过于智能.

### 重写equals方法的步骤

1. 使用==操作符检查对象引用是否相等. 如果是,那么是同一个对象,直接返回`true`.
2. 使用`instanceOf`检查参数类型是否正确.
3. 类型转换.(如果是`jdk 14`以上可以和上一步合并: `if(o instanceof X x){}`)
4. 对该类型中的关键字段进行比较.如果是除浮点数之外的基本类型,直接用==判断,如果是对象递归使用`equals`,如果是浮点数(`float,double`)使用`Float.compare(param1,param2), Double.compare(param1,param2)`,原因是`float`和`double`中存在`Float.NaN,-0.0f` 这样的常量.


**总结: 总之不要轻易的自己重写`equals`方法,在多数情况下并不需要,如果需要请使用第三方,如果还不满足在自己动手写.**


---

## 重写equals方法时重写hasCode方法

### 特点

1. 如果两个对象`equals`, 那么他们必然具有相同的`hashCode`.  
**为什么?**   
**因为在使用`hashMap`等集合时, 如果相等的对象具有不同的`hashCode`,可能会放在不同的`bucket`中,这样导致`get`逻辑上相等的对象时, 会出现获取不到对象.**  
2. 如果两个对象不`equals`, 但是他们可能具有相同的`hashCode`, 但是最好不要, 因为这样`HashMap`等依赖`hashCode`方法的集合类会变的性能非常低下,最好是不同的对象具有不同的`hashCode`

### 重写步骤

1. 定义一个`result`存储第一个关键字段的`hashCode`.
2. 关键字段的`hashCode`的计算:   
    a. 若果字段是基本类型,则调用对应的包装类型的`hashCode(value)`方法,如: `Integer.hashCode(code)`.
    b. 如果字段是对象引用,并且`equals`中使用到这个字段时, 则同样的递归的调用该字段的`hashCode`方法. 如果需要更复杂的比较, 则可以为这个字段计算一个`范式`,然后针对这个范式计算`hashCode`. 如果该字段是`null`则返回`0` 
    c. 如果字段是一个数组,则逐个计算数组中的元素的`hashCode`,如果数组不重要,返回一个常量,但最好不是`0`.
3. 根据前两步骤计算,合并除最后的`hashCode`
4. 完整示例: 
```java
public int hashCode() {
    int result = Short.hashCode(param);
    result = 31 * result + Integer.hashCode(param1);
    result = 31 * result + Double.hashCode(param2);
    return result;
}
```
> 使用乘法使得`hashCode`依赖字段顺序,设想一下如果不用乘法那么`abc`和`bac`将会拥有相同的`hashCode`,这显然是不对的.
> 使用31这个数字书中给出的原因是: 因为它是一个奇素数,习惯上使用,可以使用移位和减法来优化乘法`31 * i == (i << 5) - i`,而且虚拟机自动完成这一优化;如果想知道更多具体内容请参考[stackoverflow上的回答](https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier)


--- 

## 重写toString方法

该条主要作用是在日志或者输出对象时候,比较容易的读懂对象中的信息,书上说的有点啰嗦,最简单的方法是使用三方`json`库将对象直接按`json`输出.不建议自己手写,字段多了容易遗漏外加出错.

---
## 重写clone方法

### 通用约定(非必须)

1. x.clone() != x;
2. x.clone().getClass() == x.getClass();
3. x.clone().equals(x);

### 实现步骤

1. 调用`super.clone()`, 然后转换类型.
2. 如果该类包含数组引用类型的字段, 并且是非`final`的, 那么调用该字段的`clone()`方法.
3. 如果该类包含引用类型中的字段还包含其他引用类型,那么递归调用,进行深拷贝.
4. 使用`构造拷贝器`--一个接收自身类型为参数的构造函数. 或者使用构造器静态工厂方法.


> 总结: 实际生产过程中, 很少遇到需要调用`clone`方法来获取对象.

---
## 考虑实现comparable接口

如果是在编写一个值类(value class)并且可能排序敏感,那么建议你实现`comparable`接口,这样当这个类的实例添加到集合里面的时候,便于搜索,分类,排序.  

### 通用约定
1. `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`,其中`sgn`为`signum`函数,根据表达式的值,返回`-1,0或者1`.
2. 如果`x.compareTo(y) > 0 && y.compareTo(z) > 0` 则 `x.compareTo(z) > 0`.
3. 如果`x.compareTo(y) == 9` 那么有` x.compareTo(z) == y.compareTo(z)`. 
4. 如果`x.compareTo(y) == 0` 那么他们最好是相等, 如果不等请注明.

### 实现

实现该方法时,最好不要使用`<, >`符号.

```java
public int compareTo(PhoneNumber pn){
    int result = Short.compare(this.areaCode, pn.areaCode);
    if(result == 0){
        result = Short.compare(this.prefix, pn.prefix);
        if(result == 0){
            result = Short.compare(this.lineNum, pn.lineNum);
        }
        ... 
    }
    return result;
}
```
或者使用`java 8`中`Comparator`中的函数式接口实现
```java
public static final Comparator<PhoneNumber> COMPARATOR = comparingInt(pn -> pn.araeCode)
                                                                     .thenComparingInt(pn -> pn.prefix)
                                                                     .thenCOmparingInt(pn -> pn.lineNum);
public int compareTo(PhoneNumber pn){
    return this.COMPARATOR.compare(this, pn);
}
```



# 后记

实际开发过程中很少用到该章节知识, `equals, hasCode, toString`等方法都是用第三方类库实现, `comparable`接口, 在流式处理集合的时候可以手动指定比较器. 总之, 该章节内容,理论大于实践, 只有在很少的情况用到, 自己有特殊的需求时才会用到. 