---
title: Naming Convention--命名规范
date: 2020-09-21 21:38:30
tags: java 
---
译文: [Clean Code (Best practice for naming) Part 1](https://blog.usejournal.com/clean-code-best-practice-for-naming-part-1-f67ebe8c0894)
<!--more-->
# 引言
  在代码开发过程中好的变量命名习惯是非常重要的,如果是i,j,k,l的命名,我相信不用同组小伙伴打死你,几个月以后的自己都想打死自己.所以,一个好的命名规范尤为重要,不仅要让编译器看懂,阅读代码的人更要是见名知意.

---
# 译文
  [原文地址](https://blog.usejournal.com/clean-code-best-practice-for-naming-part-1-f67ebe8c0894)  
  
  在你写代码时候,你觉得你是为谁在写?第一想法是我写代码是为了编译器能编译.想法是对的,编译器应该能'看懂'你得代码并且能编译执行这些代码.那有没有其他的读者阅读你的代码呢?在专业的开发中,你开发代码并不是隔离式的,你处在一个团队里面,并且团队里面可能有很多其他成员,所以你写的代码应该能让这些团队成员看懂.大概率会发生你需要重新审视这些代码来了解这些代码是什么,怎么工作的,为什么会这么写.大概率会发生在将来你无法回答这些问题的时候正好有其他人需要你这段代码.四个不同的组,最后面的3个组有共同点是非常重要的.所以我们需要确保我们写出的代码能被人类读懂.这就是代码整洁之道.只有遵循这些准则,才能确保你写的代码能被将来阅读它的人看懂,或者你的同组小伙伴code review的时候看懂,或者你自己将来做bug fix是看懂,或者你得交接人能添加一个新的功能.  

---- 
# 怎么样才是整洁的代码
  * 代码能被机器编译以及能被人类看懂
  * 代码名称简单易懂
  * 格式一致,格式影响代码的可读性
  * 容易改善,比凌乱的代码容易修复
  * 能清晰的表达出其意图.写代码的人能明白这段代码是做什么的

如下代码,该方法名称不够整洁规范,以至于无法说明方法的意图,这种情况你就需要一行一行的阅读代码才能了解他是做是你么的: 
```
fun s(arr: IntArray) {
	val n = arr.size
	for(i in 0 until n-1) {
		for (j in 0 until n-i-1)
			if (arr[j] > arr[j+1] {
				val temp = arr[j]
				arr[j] = arr[j+1]
				arr[j] = temp
			}
	}
}
```

整洁规范的代码: 
```
fun buuleSort(array: IntArray) {
	for (index in 0 until arraySize - 1) {
        for (pointerIndex in 0 until arraySize - index - 1)
            if (array[pointerIndex] > array[pointerIndex + 1]) {
                val temp = array[pointerIndex]
                array[pointerIndex] = array[pointerIndex + 1]
                array[pointerIndex + 1] = temp
            }
    }
}
```
这样你能一眼就看出来这个方法的意图,对,就是冒泡排序.

---
# 类命名规范
* 类名应该使用名词. $#x1F47D

```java
class Performer{}
class Performance{}
```
* 避免动词形式的类名.

```java
class Perform{}
class Performed{}
class Performing{}
```

* 使用形容词前缀表示时态.

```java
class ActivePerformance {}
class PastPerformer {}
````

* 类名不能仅使用形容词.
```java
class Huge {}
class Small{}
class Fast {}
class Slow {}
```

* 使用形容词前缀加名词作为类名.

```java
class SmallPerformance {}
class PastPerformer {}
```

* 避免使用模糊前缀.

```java 
class MyPerformer {}
class APerformer {}
class ThePerformer {}
class ThisPerformer {}
```

* 避免使用单个字母作为类名.

```java 
class P {}
class L {}
```
* 避免使用单字母前缀类名.

```java
class CPerformer() {}
class TPerformer() {}
```

* 避免使用__首字母缩写词__大写.

```java
class HTTPAPIPerformer {}
```

* 在单词连接处首字母大写(驼峰命名).

```java 
class HttpApiPerformer {}
````

* 避免使用缩略词.

```java
class Perf {}
```

* 避免使用复数作为类名.

```java 
class performers {}
````

* 使用复数作为集合类的类名.

```java 
class Currencies {
...// contain map of Currencies, and romat price for each currency

	val currencyMap = mapOf(
		Pair(RUSSIAN_RUBLE, "\u20BD"),
		Pair(UNITED_STATES_DOLLAR, "\$")
	)
...
}
```

---
# 整洁规范的方法名称  

* 使用一般现在时作为方法名称.

```go
func open() {}
func perform() {}
func close() {}
func validate() {}
```

* 避免使用动名词(现在进行时).

```go
func performing() {}
func validating() {}
func opening() {}
func closing() {}
```

* 避免使用一般过去时.

```go
func performed() {}
func opened() {}
func closed() {}
func validated() {}
```

* 使用`is`作为动名词前缀.

```go
func isRunning() {}
func isClosing() {}
func isServint() {}
```

* 使用`has`作为一般过去时前缀.

```go
func hasPerformed() {}
func hasOpened() {}
func hasClosed() {}
func hasValidated() {}
```

* 在应用系统中保持所有的命名标准和转换一致.
* 如果语言支持驼峰命名,则应该使用驼峰命名.
驼峰命名的准确率比下划线命名准去率要高(高出51.5%的几率)  

---
# 整洁规范的变量命名

* 使用单数名词作为原始类型和对象类型的变量名]

```java
int count = 0;
User user = new User();
```

* 使用复数名字作为数组和集合的变量名

```java
String[] names = new String[]("Alex", "Ali", "Aesop"};
List<String> names = new ArrayList<String>();
```

* 避免使用__动词__作为原始类型的变量名
```java
boolean create = false;
int perform = 12;
```

* 使用__名词__作为原始类型的变量名
```java
int performanceCode = 12;
boolean creationEnabled = false;
```

* 避免使用单个单词作为变量名
```java
int s = 12;
int i = 8;
```

* 使用有意义的变量名

```java
int size = 10;
int index = 9;
```

* 避免使用容易引起混淆的缩写和简写.

```java
String dbsqlSelAllNames = "select * from names";
```

* 使用大写分割变量名,并将简写展开
```java
String dbSqlSelectAllNames = "select * from names";
```

* 不要使用无用的复杂前缀,如__匈牙利前缀__
```java
String f_strFirstName = "Jefferson";
```

* 避免使用数据类型最为变量名后缀

```java
String lastNameString = "Amaya";
```

---
# 整洁规范的参数名

* 命名参数包含单个值时,使用单数名词

```java
public int add(int left, int right){
	return left + right;
}
```

* 命名参数包含多个值时,使用复数名词

```java
public int sum(List<Integer> values){
	return values.stream().collect(Collectors.summarizingInt(Integer::intValue)).getSum();
}
```
* 避免使用单个字母作为参数名称

```java
public int add(int i, int j) {
	return i+j;
}
```

* 避免使用简写作为参数名

```java
public void open(String FSP){}
```

* 参数首字母不应该大写

```java
public void random(int SeedGenerator){}
```

* 避免使用难懂的前缀

```java
public void persistName(String sName){}
```

---
# 整洁规范的常量名
* 常量名所有的字母都应该大写.
* 使用单数名词作为原始类型常量命名.
* 使用复数名词作为集合常量命名.
* 避免使用单字母和简写.
* 确保__首字母缩写词__之间的分割.


---
# 后记

这些规范是根据语言来定,对于其他语言可能有所变化
