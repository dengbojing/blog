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
int常量并不好维护, 而且即使你将常量类型弄混了, 也不会有什么问题, 因为都是int类型, 而且java没有命名空间这个概念, 只能使用前缀作为区分,这种情况下将apple常量和orange常量混合起来计算也不会出现任何编译或者运行时错误, 只会得到错误的结果. 

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

如果将这些常量分开使用枚举类型来表示,  那么此时你将Apple和Orange混合使用计算,那么会报错, 因为两者不兼容.  


__如果一个枚举类型和一个类又紧密的联系,而且不会被外部使用, 那么应该将枚举作为该类的成员内部类使用, 而且尽量降低枚举的可见性, 如果只在包内使用那么可以设为private 或者 package-private__  



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

命名模式: 就是约定(方法名,字段名,类名等)命名的格式或者规范, 这种方式有很大的弊端: 
1. 容易拼写错误`test`拼写为`tset`
2. 