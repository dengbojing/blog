---
title: Effective-java-第五章学习笔记
date: 2021-07-08 14:52:12
tags: java,读书笔记
---

第五章: 泛型使用注意事项

<!--more-->

# 泛型术语

| 名称                    | 写法                               | 翻译           |
| ----------------------- | ---------------------------------- | -------------- |
| Parameterized type      | List<String>                       | 参数化类型     |
| Actual type parameter   | String                             | 实际类型参数   |
| Generic type            | List<E>                            | 泛型           |
| Formal type parameter   | E                                  | 形式类型参数   |
| Unbounded wildcard type | List<?>                            | 无界通配符类型 |
| Raw type                | List                               | 原始类型       |
| Bounded type parameter  | \<E extends Number>                | 有界类型参数   |
| Recursive type bound    | \<T extends Comparable\<T>>         | 递归类型限制   |
| Bounded wildcard type   | List<? extends Number>             | 有界通配符类型 |
| Generic method static   | static \<E> List\<E> asList(E[] a) | 泛型方法       |
| Type token              | String.class                       | 类型标记       |

# 不要使用原始类型

## 为什么不该使用

每一个泛型类型定义都对应着原始类型,通俗来说就是不带泛型. 如:

```java

Collections stamps = ...;
```

如果使用这种集合,你可以添加任何类型的对象,看名字其实只是想添加`Stamp`类的实例,他是不报错的.ide 工具会给出警告.

```java
stamps.add(new Stamp()); //unchecked call add(e)
stamps.add(new String());  // unchecked call add(E)
stamps.add(new Integer(1)); // unchecked call add(E)
```

在遍历取出的时候,编译期间还是不会报错,只有运行时候才会抛出异常.

```java
for (Iterator i = stamps.iterator(); i.hasNext(); )
  Stamp stamp = (Stamp) i.next(); // Throws ClassCastException
  stamp.cancel();
```

编译期间发现不了问题就很可怕,如果这段代码一直没有执行,那么系统一直没有问题,知道有一天它执行了,boom!  
这段代码就是个定时炸弹,所以千万不要用泛型的原始类型.  
而且就修复来说,这种情况就很麻烦, 修复,重新编译, 测试, 发布.

## 替代原始类型的方式

如果你想一个容器添加任意类型的参数,那么可以使用`List<Object>`这种形式,这种形式处于泛型系统之内,只是显示的告诉了编译器,我何以接受任何类型的参数.  
如果你使用原始类型的那么你将失类型安全,但是如果使用`List<Object>`并不会有这个问题.

```java
List<String> strings = new ArrayList<>();
unsafeAdd(Strings, Integer.valueOf(3));

public void unsafeAdd(List list, Object obj){
	list.add(obj); // warning  unchecked call to  add()
}
```

如上代码,编译器并不会报错,只会给出警告,如果你不在意这些警告,那么,运行时就会报错.但是如果我们换成以下代码:

```java
List<String> strings = new ArrayList<>();
unsafeAdd(strings, Integer.valueOf(2)): //compiler error
public void unsafeAdd(List<Object> list, Object obj){
	list.add(obj):
}
```

这段代码会在编译期间就报错,有助于你提前发现问题,修复问题.

## 无界通配符 <?>

如果你在写一个对外的`api`但是返回值不确定,可能是`String`,可能是自定义类型,这取决与需求是什么样的,那么这个时候该怎么写呢:

```java
public Response<?> api(String str){
	return Response.success(someService.doSth(str));
}
public Response<T>{
	public static <T> Response<T> success(@NotNull T t) {
	        return of(t, ResponseState.SUCCESS);
	}
}
```

无界通配符不能用来的容器不能用来添加东西,`null`值例外,如:

```java
public void toAdd(List<?> list){
	list.add(null);
	list.add("fda"); //complier error
}

```

这个是编译器为了阻止改变参数的类型, 假设调用此方法传进来的参数是`List<Integer>`, 那么添加`String` 类型就会抛出异常, 所以编译器提前阻止了这种类型的改变.

## 只能使用原始类型的特殊情况

1. 代表`class`字面量, 比如`List.class`
2. 使用`instanceof`

# 消除`unchecked`警告

## 手动消除警告

使用`@SuppressWarnings("unchecked")` 消除警告.注意一下几点:

- 尽可能的缩小注解的作用范围.
- 需要在注解上注释描述为什么这么做

# 优先使用泛型集合,而不使用数组

## 数组是协变的, 而泛型集合是不变的

如果一个`sub` 是 `sup`的子类, 那么`sub[]` 也是 `sup[]`的子类型. 是兼容的.  
但是,`List<sub>`却不是`List<sup>`的子类行,两者是不兼容的.

```java
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException

// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

以上代码可以看出第一段代码得等到运行时才会发现问题,如果代码得不到执行,那么程序就一直是正常运行,定时炸弹.  
而第二段代码在编译期间就指出了错误.

## 为什么不能显示的创建泛型数组

数组是具体化的,数组不论是在编译期间还是运行期间都会强制要求数组中的元素类型一致.  
而泛型在编译期间强制要求元素类型,但是在运行期间会进行泛型擦除.这么做的目的是为了兼容 `jdk5` 之前的代码.

基于以上两点就很好解释为什么不能创建泛型数组了.

```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

如上代码, 假设`(1)`是可以编译通过, `(3)`由于数组的协变是正确的语法,因为所有类都是`Object`的子类, `(4)` 由于泛型的擦除在运行期间`ListMString>[]` 被擦除为`List[]`, `List<Integer>` 被擦除为`List` 因此不管编译还是运行都不会出错, `(5)`在运行期间会出现`ClassCastException` ,因为存进去的显然不是`String`; 因此在编译期间就阻止`(1)`编译通过.  
如上所说`List<E>`, `List<String>`, `E`这些参数化类型,被称为不可具体化的类型,因为他们在运行期间会被泛型擦除,无法表达容器在运行时所需要的元素类型. 只有一种情况例外就是`<?>`无界通配符:

```java
List<?>[] list = new ArrayList<?>[1];
```

这种是可以编译通过的, 但是问题来了, 无界通配符容器是不允许被改变的, 这么做毫无意义.

## 使用 List<E> 代替 泛型数组定义 E[]

简单来讲就是你无法创建一个泛型数组, 但是可以通过`Collection.toArray()`获得一个泛型数组:

```java
E[] e = (E[]) someCollection.toArray(); //unchecked cast
```

这样我们就获得另一个泛型数组, 但是需要强转, 且这句话会出现警告, 因为在运行期间会出现泛型擦除, 所以编译器无法保证运行时类型转换正确, 所以会出现`unchecked cast`. 因此,我们需要稍微修改一下:

```java
List<E> lists = new ArrayList<>(someCollection);
```

虽然丧失了一些性能, 但是编译器能保证正确的类型转换.

# 偏好使用泛型

通俗来说就是,如果你在写一个容器类, 里面所需要容纳的元素类型,最好使用泛型来代替. 书中举例:

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
    if (size == 0)
    throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    public boolean isEmpty() {
        return size == 0;
    }
    private void ensureCapacity() {
        if (elements.length == size)
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

应该替换为:

```java
class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() throws EmptyStackException {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    public boolean isEmpty() {
        return size == 0;
    }
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
这条看上去是和上面一条有些冲突,上面一条说明优先使用泛型集合,而这里又使用泛型数组了;因为,这里实现的是`Stack`容器类,类似`List`容器类,所以使用泛型数组,而且`List`容器类的实现也是泛型数组实现.

# 偏好使用泛型方法

简单来说,就是任何容器实例,都应该使用泛型化来使用, 比如:

```java
...
public static add(set o, set  o2){
	o.addALL(o2);
}
...
```

应该改为:

```java
...
public static <E> void add(Set<E> o, Set<E> o2){
	o.addAll(o2)
}
...
```

**在方法修饰符和返回值之间的`<E>`,被称为类型参数列表.**


## 泛型单例工厂

返回一个`函数式对象`可以包含不同的`参数化类型`,例如: `Collections.reverseOrder()`和`Collections.emptySet()`. 下面是书中给的`恒等函数`示例: 
```java
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```


## 递归类型界限

`<E extends Comparable<E>>`: 类型参数`(E)`被涉及到类型参数本身的表达式`(<Comparable<E>>)`限制.
这种用法很少用到,一般是用在有些自我表达,自我操作的方法. 比如: 一个集合里面的排序, 最大值,最小值.(自我比较)


# 使用界限通配符增减API的灵活性

为了使方法拥有最大的灵活性,可以在表示消费或者生产的方法的参数上使用通配符类型来代表

- 上界通配符 `<? extends E>`
- 下界通配符 `<? super E>`
- PECS -- producers-extends and consumers-super. 代表生产时使用`extends`, 代表消费时使用`super`


```java
public static <E> void add1(List<? super E> list, E e){
    list.add(e);
}

public static <E> E get(List<? extends E> list){
    return list.get(0);
}

```
__如果在方法定义的时候类型参数只出现一次,那么应该使用通配符来代替(有界或者无界)__  

__不要在返回值上使用通配符__, 这将会迫使客户端也使用通配符.   

__如果一个用户使用这个方法要考虑通配符类型,那么这个`api`可是是错的,好的`api`应该是让用户感觉不到通配符的存在__ 


#  谨慎的将可变参数和泛型结合在一起

## 将值存进可变泛型数组是不安全的

在'为什么不能显示的创建泛型数组'中已经演示过,泛型数组的危害,这里稍微改变一下上述代码:  
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // Heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
```
这里其实就是将之前的代码中显示的创建泛型数组,改为可变参数泛型数组. 因为可变参数实际上编译完成之后,是将可变参数存进一个临时数组里面,所以上述代码实际上就等于隐式的创建了一个可变参数泛型数组,但是最终导致类型装换错误,原因之前已经分析过了.  

__结论1: 将值存入泛型数组是不安全的,泛型数组应该只是传递这些值,比如: Arrays.asList(T...t), 该方法只是将可变参数里面的值放到List里面然后返回该List__   
__结论2: 返回泛型数组引用也是不安全的, 比如下列代码:__
```java
// UNSAFE - Exposes a reference to its generic parameter array!
static <T> T[] toArray(T... args) {
    return args;
}
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError(); // Can't get here
}
public static void main(String[] args) {
    String[] attributes = pickTwo("Good", "Fast", "Cheap");
}
```

`toArray`方法直接诶返回了泛型数组, 而`main`方法调用`pickTwo`方法编译器会推断出该方法返回`string[]`数组,所以在`return toArray(x,x)`处会出现一个隐式的转换转换为`Object[]`数组. 显然`string[]`数组不是`object[]`数组的超类. 所以出现`ClassCastException`, 由此可以看出返回一个泛型数组的引用是多么的不安全, 除非是受到控制的方法, 即不对外暴露的方法, 由`api`方法编写者确认这个使用是安全的, 并且使用`@SafaVarargs`注解.  




# 类型安全的异构容器

一个很有意思的写法:   
```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

可以存入任意类型的`Class`(不能是原始类型), 然后获取对应的值.
