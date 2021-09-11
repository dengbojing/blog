---
title: Effective-java-第九章学习笔记
date: 2021-08-25 15:19:25
tags: java
---

第九章：通用编程意见

<!--more-->

# 最小化局部变量得作用范围

前面说过将类得成员变量访问最小化，增强代码得可读性和可维护性，同样将局部变量得访问最小化，也利于维护和可读性。

## 在变量使用前声明

如果过早得声明变量，会造成读代码的人不得一直记住这个变量的值，会给阅读带来混乱，等到读到使用变量的地方，可能读者已经忘记了变量的值，不得不重新在去寻找一遍该变量的声明。如果一变量在使用他的代码块(`{}`之间)之外声明，那么在代码块执行结束以后，该变量仍然可见，如果在在代码块执行之前被失误的使用，那么可能造成程序错误。


## 声明变量应该初始化(赋值)

如果你要声明一个变量那么最好是有足够的信息进行初始化该变量，否则应该推迟到直至有足够的信息初始化该变量在声明。在`try-catch`声明变量例外，因为try里面的变量外部无法访问，那么就无法释放资源。

## 循环变量

一般在循环中的变量都有一个特殊的地方声明变量来使变量作用域最小化，就是在循环的小括号内，因此应该优先使用`for`循环，而不是`while`循环，因为`while`循环没有一个特殊的地方(小括号内)声明变量，比如：

```java

for (Element e : c) {
... // Do Something with e
}


Iterator<Element> i = c.iterator();
while (i.hasNext()) {
doSomething(i.next());
} ...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
doSomethingElse(i2.next());
}
```

上述`while`循环就会出现问题，第二个`while`循环使用的条件判断是`i`而不是`i2`,有可能第二个`while`会执行，也可能不会执行，也可能执行几个，那得看`c`比`c2`大多少倍了。 第一个增强的`for`循环就不会有问题。

另一种是循环变量最小化的做法：

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
... // Do something with i;
}
```
`n`用来表示`i`的上限，此处`n`只是用来限制`i`的上限，只会被用到一次，所以应该声明为循环变量。  



## 是方法体代码少和功能单一

如果把多个功能的操作合并到一个方法里面，不仅造成方法臃肿，而且两个功能的操作变量还可能重复，造成误操作。、


# 优先使用`for-each`循环代替`for`循环

传统的迭代器循环
```java
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
Element e = i.next();
... // Do something with e
}
```
传统的数组循环
```java
Click here to view code image
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
... // Do something with a[i]
}
```

以上两种方法，迭代器中的变量和索引变量都要使用3次，可能会出现错误的使用索引(多重循环最容易出现此问题)，编译器可能发现不了问题。使用`for-each`循环(增强的`for`)

```java 
for (Element e : elements) {
... // Do something with e
}
```

总之，`for-each`循环的简洁性和预防bug性都要强于传统的`for`循环，但是如果有一下情况无法使用`for-each`循环。

1. 过滤-如果要遍历集合，并删除指定元素，就需要显示的的迭代器，以便可以调用它的`remove`方法。

2. 转换-如果需要遍历数组，并且替换其中某些位置的元素，那么就需要用到下标替换。

3. 平行迭代-如果需要并行的遍历多个集合，就需要显示的控制迭代器或者索引变量，以便于所有的迭代器或者索引都同步前向移动。


# 了解和使用`java`常用类库

书中开头举例了一个随机算法的例子，说明了线性同余`x(n+1) = (a*x(n)+c) % n`伪随机方法,`a,c,n`为常数，`x(n)`随机种子，产生随机数的缺陷： 
1. 存在生成周期，如果n比较小,那么生成周期就小，随机数过一个周期就会重复。
2. 如果n不是一个2的乘方，随机分布不均匀，如果n比较大更会体现出这个缺点。
3. 第三个问题和线性随机无关，是`java`中最小值取绝对值，还是最小值(负值)。

但是书中又说了,直接使用`Random.nextInt(int)`就可以产生一个想要的随机数，没看太明白，待实践到底表达什么意思

同时`jdk7`以后生成随机数时候使用`threadlocalrandom`，在`stream`中使用`splittablerandom`  


__总之， 每个`java`程序员都应该熟悉的包：`java.lang,  java.util.concurrent, java.util.function, java.time, java.io, java.nio, java.util.stream`__ 



# 精确的数值计算避免使用 float double

__浮点数公式: V = (-1)^S * M * R^E__

其中各个变量的含义如下：

- S：符号位，取值 0 或 1，决定一个数字的符号，0 表示正，1 表示负
- M：尾数，用小数表示，例如前面所看到的 8.345 * 10^0，8.345 就是尾数
- R：基数，表示十进制数 R 就是 10，表示二进制数 R 就是 2
- E：指数，用整数表示，例如前面看到的 10^-1，-1 即是指数

参见文章[浮点数的二进制表述](https://www.ruanyifeng.com/blog/2010/06/ieee_floating-point_representation.html), 显然由于浮点数的表示精度由尾数决定, 浮点数的范围有指数决定,而浮点数是固定宽度的(32和64位), 按照约定来说就可能出现丢失精度或者说出现很大无法表示的小数, 在对数字高敏感度的环境并不适用, 比如银行交易(在项目里使用浮点数计算金钱基本就等着被开除吧), 出现了错误可能是致命的, 所以应该是`BigDecimal`来代替。



# 基本类型优先于包装类型

1. 基本类型值包含值，包装类型不仅包含值，还包含一些对象信息，所以包装类型是不能使用==进行比较的.  
2. 在使用包装类型进行操作时候，会出现空指针异常，而且进行算术运算会有自动拆箱操作，性能较差.  


**在编写`PO`和`VO`时候应该使用包装类型，数据对应的任何字段都有可能是`null`值，使用基本类型在获取数据时候会出现错误，前端页面传值也会有这种问题**

# 有更合适的类型，尽量避免使用`String`


1. `String` 不适合代替其他类型得值
2. `String` 不适合代替枚举值
3. `String` 不适合代替聚合类型
4. `String` 不适合作为能力表示

# 字符串连接`+`性能低下


如果在`for`循环中拼接字符串，使用`+`会使得性能非常低下，建议使用`StringBuilder`,如果只是定义简单得字符串可以使用`+`


# 通过接口引用对象

如果有合适得接口存在，那么参数，返回值，成员变量，这些都应该被定义为接口类型。使用接口定义，可以使程序更加零多（多态性），有一种例外，就是如果需要类得具体功能，而接口又不具备，那么此时应该使用具体得类作为对象的引用。


# 接口优先于反射


反射性能非常差，而且代码多，编译期间无法检查你反射出的对象是否可以执行对应的代码。  
但是有些时候不得不使用反射，在编写一些底层框架代码,如`Spring`等这些框架就利用了反射，而反射只需要在启动时候执行一次，也不会影响运行过程中的性能;或者编写一些类似对象监视作用的代码也可能会需要反射机制。


# 谨慎的使用`native`代码


非常难以调试，出错了也不知道报错信息，之前朋友代码中使用`JNI`非常不友好，`native`代码出错没法调试，内存泄漏没法控制，灾难性的东西。


# 不要过早的优化

1. More computing sins are committed in the name of efficiency (without
necessarily achieving it) than for any other single reason—including blind
stupidity.  
                                                   —William A. Wulf [Wulf72]
2. We should forget about small efficiencies, say about 97% of the time:
premature optimization is the root of all evil.  
                                                  —Donald E. Knuth [Knuth74]
3. We follow two rules in the matter of optimization:
Rule 1. Don’t do it.
Rule 2 (for experts only). Don’t do it yet—that is, not until you have a
perfectly clear and unoptimized solution.  
                                                  —M. A. Jackson [Jackson75]



4. 努力编写好的程序，而不是快的程序

5. 努力避免那些限制性能的设计决策

6. 考虑`API`设计决策对性能的影响

7. 未获得更好的性能，对`API`进行包装，是一个非常愚蠢的想法

8. 每次优化前后都要进行性能测试

#  遵循通用命名规范

参见[Naming Convention--命名规范](https://dengbojing.com/2020/09/21/Naming-Convention-%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83/)
 