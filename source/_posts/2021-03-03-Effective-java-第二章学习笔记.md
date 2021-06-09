---
title: Effective_java_第二章学习笔记
date: 2021-03-03 14:27:01
tags: [java]
categories: java,读书笔记
---
第二章: 创建和销毁对像
<!--more-->
# 引言
第一章主要讲除了用构造函数之外,如何创建一个对象,以及他们之间的利弊

# 正文

1. 使用静态工厂方法创建对象.  
    在对象方法内部,或者使用一个单独的工具类来维护一些静态的创建该类的对象的方法.  
    工具类(**书中术语叫做伴生类**)命名方式一般在该类的后面加`s`, 比如`Collection` 和 `Collections` , `Collector` 和 `Collectors`  
    工厂方法的命名方式一般有:  
    `from` 类型转换方法, 从一个对象中获取我们想要的类的对象,通常只有一个参数. 如:
    ```java
    NewsBody newsbody = NewsBody.from(news);  
    ```

    `of` 聚合方法,将多个参数聚合在一起, 如:  
    ```java
    NewsBody newsbody = NewsBody.of(news.getTitle(),news.getAuthor(),news.getReleaseDate());  
    ```
    `valueOf` 功能和上面两个类似, 只是相对来说更加啰嗦, 如:
    ```java 
    NewsBody newsBody = NewsBody.valueOf(news);  
    ```

    `instance` or `getInstance` 根据给定的参数(可选)来创建对象,但是不能保证该对象一定和参数所描述的对象一致, 如:  
    ```java
    NewsBody newsBody = NewsBody.instance(news); // 可能newsbody中的author字段或者其他字段与参数news中的不一致
    ```
    `create` or `newInstance` 根据参数每次都返回一个新的对象, 如: 
    ```java 
    NewsBody newsBody = NewsBody.create(news);
    ```
    `get`*Type* 功能和`getInstance`相同,只是该方法处于工具类中, 如`java nio2`中:  
    ```java
    FileStore fs = Files.getFileStore(path);
    ```
    `news`*Type* 功能和`newInstance`相同, 只是该方法处于工具类中, 如: 
    ```java 
    BufferedReader br = Files.newBufferedReader(path);
    ```

    静态工厂的优点:  
    - 除了上述通用的命名方式之外, 可以起一个见名知意的方法, 书中举例为`BigInteger`中获取素数的方法.
    - 第二个优点是可以控制返回的实例,可以在第一次创建时候缓存起来,以便之后使用,经典案例就是`单例模式`和`享元模式(String采用的模式)`. 伪代码: 
    ```java
    public class Elvis{
        private static final Elvis INSTANCE = new Elvis();
        private Elvis(){}
        public static Elvis getInstance(){return INSTANCE;}
        public String doSth(){}
    }
    ```
    由于`INSTANCE`是静态的, 所以在类加载时就会创建类的实例, 天然避免了多线程并发问题, 使用静态工厂方法获取该实例, 则每次都获取相同的实例.
    - 第三个优点就是多态. 这个是面向对象三大特性中的重要特性. 使用静态工厂方法, 你可以返回任意一个子类的对象.书中讲述了`Collections`的由来, 但是在`java 8`之后, 接口是可以包含静态方法的,所以`伴生类`存在的理由就很薄弱.  
    - 第四个优点是可以根据参数的不同, 静态工厂方法返不同类型的对象.(这一点理解比较模糊,感觉和上一条重复).
    - 第五个优点在编写静态工厂方法时候, 方法返回对象所属的类, 不一定存在. 还是利用多态的特性. 书中举例`SPI(Service Provider Interface)`机制, 在编写`Driver.getConnection()` 具体的`Connection`实现类不一定存在, 由各大数据库厂商自己提供实现.  

    静态工厂的缺点: 
    - 必须提供一个`public`或者`protected`的构造函数, 否则无法子类化. 
    - 第二个缺点是不好找到, 如果是在该类内部还好, 如果是工具类, 那么就不容易被发现. 

    扩展:  
    - `SPI`主要是使用`ServiceLoader`加载位于`META-INF/services`下面配置的具体的实现类来完成服务. 具体角色如下:  

        - `Service Provider Interface` 服务提供者接口, 通常一种约定, 约定了实现了该接口的类会提供哪种服务.

        - `Service Providers` 服务的具体提供者, 实现了`Service Provider Interface`.  并将该类全限定名称写在`META-INF/services`目录下以服务提供者接口命名的文件中. 

        - `ServiceLoader` 用来加载`META-INF/services`下所有配置的服务具体提供者的类. 
        ```java
        ServiceLoader<T> serviceLoader = ServiceLoader.load(服务提供者.class);
        ```


2. 遇见多参数构造函数时,考虑使用`Builder`模式.  
    如果参数过多可以使用`重叠构造器`, 即构造器套构造器这种, 或者使用`javaBean`, 即`setXxx()`, 这两种方式来创建对象.  
    第一种方法, 代码臃肿不好维护, 而且可读性也差, 如果参数多了就会不知道构造器里面参数是干什么的, 一般有思想的程序员都不会写出来这种代码.  
    第二种方法我们在编码过程中经常使用,如: `news.setAuthor("dengbojing")`, `news.setTitle("xxxx")`, `news.setReleaseDate(new LocalDate())`, `news.setContent("xxxx")` 等等, 该方法弊端就是会出现在构造过程中出现对象状态不一致, 因为构造过程分为几个步骤(分别设置所有属性).   
    此时使用`Builder`模式就很容易避免上述错误, 在`jdk`中其实有很多地方都是使用这种方法, 比如: 
    ```java
    var httpClient = HttpClient.newBuilder()
                    .authenticator(Authenticator.getDefault())
                    .connectTimeout(Duration.ofSeconds(10))
                    .cookieHandler(CookieHandler.getDefault())
                    .executor(Executors.newFixedThreadPool(2))
                    .followRedirects(HttpClient.Redirect.NEVER)
                    .priority(1)
                    .proxy(ProxySelector.getDefault())
                    .sslContext(SSLContext.getDefault())
                    .sslParameters(new SSLParameters())
                    .version(HttpClient.Version.HTTP_2)
                    .build();
    ```
    有现成`lombok`插件可以通过`@Builder`注解实现`Builder`模式, 方便快捷.  
    简而言之, 就是如果类里面有很多参数时候, 使用`Builder` 就是一个很不错的额选择. 

3. 用私有构造器或者枚举类型来强化单例模式
    单例模式,老生常谈的话题, 具体衍生有`饱汉模式`, `饿汉模式`(翻译过来), `双重检测`等等专业名词,  
    总结一句话: 单元素的枚举类型经常成为单例模式的最佳实践.  

4. 通过`private`构造器来增强不可实例化的类
    该条主要针对工具类,包含一些列静态参数或者方法,实例化这些类无意义,所以应该采用私有的构造函数.

5. 使用依赖注入代替硬编码
    依赖注入指的是在构造函数或者静态工厂方法中,传入参数来注入所需要的资源(如: `this.resource = recource`).其中需要注入的资源具有不可变特性.  
    需要引用底层资源的类不适合使用静态工具类和单例类来实现. 因为这两种方式都不能主动实例化对象, 每次获取的都是同一个底层资源, 所以不适合.    
    也不适合直接在这种类(需要依赖底层资源)中实例化需要的资源, 应该将这些资源或者资源工厂方法传递到构造函数或者静态工厂方法中, 通过这些来创建这种类


6. 避免创建不需要的对象

    `String` 对象, 这种频繁使用的对象, 则应该避免创建而是直接使用字符串池中的对象.  简而言之就是不要显示创建. 
    优先使用基本类型, 要避免自动拆箱装箱. 


# 后记
剩下几条不是很重要,消除过期引用, 这个在源码里面可以看到, 就是将对象引用等于`null`; 不使用`finalizer`方法,这个方法从来都没用过,只在面试题见过; 使用`try-with-resource` 一般都会使用这种方式. 