---
title: Effective-java-第六章学习笔记
date: 2021-07-15 10:29:18
tags: java
---

第六章: 枚举和注解

<!--more-->

# 使用枚举替代int常量


在没有枚举之前一致使用int常量或者string常量

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
int常量并不好维护, 而且即使你将常量类型弄混了, 也不会有什么问题, 因为都是int类型, 而且java没有命名空间这个概念, 只能使用前缀作为区分,这种情况下将apple常量和orange常量混合起来计算也不会出现任何编译或者运行时错误, 只会得到错误的结果,同样在进行调试的时候，显示的是一些数字，很难搞清楚数字背后的真正表达的含义。

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

如果将这些常量分开使用枚举类型来表示,  那么此时你将Apple和Orange混合使用计算,那么会报错, 因为两者不兼容.  


__如果一个枚举类型和一个类又紧密的联系,而且不会被外部使用, 那么应该将枚举作为该类的成员内部类使用, 而且尽量降低枚举的可见性, 如果只在包内使用那么可以设为private 或者 package-private__  


__constant-specific:__ 在枚举中定义一个抽象方法，然后在具体的枚举实现常量里面实现该方法。这样做可以使每个枚举常量都可以拥有不同的行为。比如：加，减，乘，除，4个枚举常量对应4中不同的处理数据的行为。

## 如果重写了toString方法,那么可以写一个fromString方法反向获取枚举

```java
public enum GenderType {
    /**
     * 0-男, 1-女
     */
    MALE("男"), FEMALE("女");
    /**
     * 描述性字段
     */
    private final String value;

    GenderType(String value) {
        this.value = value;
    }

    public static GenderType instance(int order) {
        if (order == MALE.ordinal()) {
            return MALE;
        }
        return FEMALE;
    }

    private static final Map<String, GenderType> STRING_TO_ENUM = Stream.of(values()).collect(Collectors.toMap(Object::toString, e -> e));


    public static Optional<GenderType> fromString(String value){
        return Optional.ofNullable(STRING_TO_ENUM.get(value));
    }

    @Override
    public String toString(){
        return this.value;
    }
}
```

# 使用实例字段代替ordinals


在枚举里面ordinals代表了每个枚举的顺序, 如果增加了或减少了枚举, 这个ordinal就发生了变化, 而这种变化是我们无法控制的,  所以应该用一个字段来代替他.  


# 使用EnumSet和EnumMap

太高深了,完全不懂,重读第三遍在重新理解.


# 用接口扩展枚举

枚举是不可继承的,因为,枚举实际上就是一个`class`编译过后的枚举是集成于`Enum`类的,而且是`final`的,`final`类是无法集成的, 为什么要设置成`final`呢,  很好理解, 所有枚举都是`extends Enum` 如果你还要在继承自己的`BasEnum` 就会出现`childEnum extends BaseEnum, Enum`这种多继承,  而`java`是禁止多继承的.

不过枚举是可以实现接口的,所以如果又扩展需要,可以实现接口

```java
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
    PLUS("+") {
    public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
    public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
    public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
    };
    private final String symbol;
    BasicOperation(String symbol) {
    this.symbol = symbol;
    }
    @Override public String toString() {
    return symbol;
    }
}
```

# 使用注解代替命名模式

命名模式: 就是约定(方法名,字段名,类名等)命名的格式或者规范, 这种方式有很大的弊端，拿`junit3`来举例: 
1. 容易拼写错误`test`拼写为`tset`
2. 你无法确定使用的是否合适或者说使用的是否准确，比如一个类名叫做`TestSafeMechanisms`, 本意是想测试该类下面所有的方法，但是`junit3`并不会这么做。
3. 没法很好的把参数和程序元素关联起来，比如想写一个测试类，在抛出特定的异常时候才算成功，这时候异常的本质是测试的参数。这时候可以利用具体的命名模式，把异常名字写在方法名中，首先这很不优雅，其次就算你写错了，编译器也检查不出来。

使用注解可以解决上述的弊端，而且从`junit4`开始也使用@Test来代替命名模式。
具体的使用可以搜索元注解，annotationsprocessor等关键信息。
总之，使用注解代替原来的命名模式；应该使用`java`预定义的注解`@Override`等注解。

# 标记接口 vs 标记注解

1. 标记接口定义了一个由标记类的实例实现的类型，这句话有点绕，实际上就是标记接口定义了一种类型。在实际运用过程中，如果一个方法要求的参数是一种标记接口，那么编译器可以检查传入的实际参数是否符合要求，而标记注解(上面所说的@Test)则不能让编译器进行类型检查。


2. 标记接口能更精确的标记，比如只想标记某些特定的接口，就可以创建一个标记接口继承这个特定的接口来进行标记。比如·`Set`之于`Collection`，可以把`Set`看成一个标记接口，`Collection`是一个特定的接口，`Set`并没有增加任何方法，可以看作只是一个标记接口。

3. 标记注解更好的地方在于可以应用于大型的框架之内，可以作为框架的元注解构成更高级的注解比如`@SpringBootApplication`就是基于·@EnableAutoConfiguration`等注解，这种方式可以实现更加丰富的框架。

4. 如果是标记方法参数，成员变量等应当使用标记注解；因为标记接口只能又类和接口来扩展；如果需要标记一个类或者接口，那么分情况，如果该接口是一个或多个方法的参数，则应该使用标记接口。如果这个类不会被用做参数，那么应该使用标记注解。或者如果被标记的类是重度使用注解的框架，那么也应该使用注解。

5. 如果标记注解是`Element.TYPE`类型（可以用在任何地方），那么应该花时间思考是否真的应该是一个标记注解，或者可以使用标记接口会更好。