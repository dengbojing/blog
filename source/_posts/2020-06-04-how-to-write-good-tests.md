---
title: how_to_write_good_tests
date: 2020-06-04 21:35:35
tags: mockito
---
译文: [How to write good tests](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)
<!--more-->
# 引言 
  代码测试是一件非常重要的工作,在之前的工作中总是找各种借口(时间不够,写起来太繁琐,有些场景无法测试)等等原因做的不够完善.有时甚至不做,无心之中发现一篇非常好的代码测试文章.花些时间来翻译一下,提高一下英文水平,顺便也学习一下正经的代码测试该是什么样的.

---
# 译文 

  [原文地址](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)

  为我们的软件定制一个测试用例是件好事,但是实际上,一个**好的**的测试用例也是非常重要的.

  遵循一些固有的原则来热爱测试代码 

---
## 保持测试代码简洁和可读 
  
  要做到这一点,需要像对生产代码那样进行*无情的重构*.否则,让事情发展下去测试代码就会变成恐怖的`祖传代码`.如果测试代码不能轻松重构,那么意味着生产代码也不能重构,从而导致`祖传代码`.**总是要勇于重构.**  

---
## 避免编码重复

  例如, 测试代码与parser使用完全相同的`正则表达式`来生成内容.  

  通常来说人们不愿意重复测试与代码的逻辑,所以在测试中重复`正则表达式`或者其他代码是不可取的.设想以下测试情况,输入/输出结果`(f(input)->(output))`,例如,如果代码要处理模版,不要添加固定值,相反,应该根据计算结果添加值. 

  ```java
  // use
  Assertions.assertThat(processTemplate("param1", "param2")).isEqualTo("this is 'param1', and this is 'param2'"));

  // instead of
  Assertions.assertThat(processTemplate("param1", "param2")).isEqualTo(String.format("this is '%s', and this is '%s'", param1, param2));
  ```

---
## 覆盖尽可能多的情况,来显示正向的用例,特别是错误的代码位置 


  通常使用`测试驱动开发(TDD--Test Dirven Development)`是最佳的实践方式. 使用`TDD`人们能在设计阶段就找出什么地方会被破坏. 不要认为为一个小的代码片段编写简单的测试不值得，你永远不知道什么时候、因为什么而修改这段代码.

  可与使用PIT([突变检测系统](http://pitest.org/))来对测试代码的有效性进行检测. 

---
## 不要Mock一个你不拥有的类型 

  这并非一条硬性规定,但是如果不遵循该条规定会有影响(很可能会有). 

  > `TDD`的设计方面和测试方面同样重要.在模拟外部API时,无法使用测试来驱动设计,该API属于其他人;因此第三方也将可以更改API的方法签名和行为. 

  1. 设想一下代码mocks了一个第三方库,在更新了第三方库之后,三方库的逻辑可能改变了一点,但是测试代码依然能够执行成功,因为他被mock了.所以在这之后,所有的事情看起来很美好,构建也成功了,但是软件部署到正式环境--爆炸! 
  2. 这也可能导致当前的设计和第三方库不够松耦合. 
  3. 另一个问是第三方库可能非常复杂需要mock许多东西才能运行,这就导致了大量的特定测试和复杂的测试装置, 而这本身就损害了简洁性和可读性的目标.或者由于模拟外部系统的复杂性而没有充分覆盖代码的测试. 

  相反,最常见的方式是创建一个第三方库的`warpper`来包装他们,不过应该注意`抽象泄漏`([什么是abstraction leakage?](https://zhuanlan.zhihu.com/p/26803553))的风险,因为太多的底层API,概念或者异常超过了`warpper`的边界.为了验证第三方提供API的可用性,请使用`集成测试`,并尽可能的是它们简洁可读. 

  下面是其他人在mock了非他所有的类型库遇到的痛苦和总结的经验: 

  + http://davesquared.net/2011/04/dont-mock-types-you-dont-own.html
  + http://www.markhneedham.com/blog/2009/12/13/tdd-only-mock-types-you-own
  + http://blog.8thlight.com/eric-smith/2011/10/27/thats-not-yours.html
  + http://stackoverflow.com/questions/1906344/should-you-only-mock-types-you-own

---
## 反模式: Mock一切 
  
  如果所有的代码都mock了,那么我们怎么测试业务代码?不要害怕不使用Mock的方法. 

---
## 不要Mock值对象 

  为什么会有人要这么做呢? 
  > 因为实例化一个对象非常痛苦? => 不是一个很好的理由

  如果创建一个对象非常困难,那么这是代码需要严重重构的一个信号.一种可行的方法就是为你的`值对象`构造一个`builder`(构造者模式)--有很多工具可以使用比如`IDE 插件`,`Lombok`等等. 还可以在测试环境中创建有意义的工厂方法.
  ```java
  abstract class CustomerCreations {
    public static Customer customer_with_a_single_item_in_the_basket() {
	    // long init sequence
    }
  }
  ```
  [`Mockito`](https://github.com/mockito)更加关注对象交互,这也是`面向对象`的重要要素. 

---
## (原文)推荐阅读[`Growing Object Oriented Software Guided by Tests`](https://book.douban.com/subject/4156589/)

必读,这本书阐释了功能完整的应用程序在从无到有过程中, 开发的许多方面以及如何在项目生命周期的各个阶段实现测试.

如果遇到一些不理解不确定的事情,可以发邮件给作者.
  

