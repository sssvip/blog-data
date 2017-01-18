date: 2017/1/17
title: 算法4一起来--一步一步实现List接口--MiniArrayList初步实现
tags: 
- algorithm
- algorithm 4
- plan
- data structure
- java
- List
- ArrayList
---
MiniArrayList是实现List接口，初步实现，未考虑太多，简化初步学习的噪音，预期达到最后和ArrayList的效果，当然性能最后也会测试对比，希望用这种方式来提高自己。（此部分全部代码见：[https://github.com/sssvip](https://github.com/sssvip/algorithms4th/tree/master/src/fundamentals/list)）

这次的初步实现采用tag形式进行打包记录，方便以节点形式进行管理：[MiniArrayList_V0.1](https://github.com/sssvip/algorithms4th/tree/'MiniArrayList_V0.1')

# MiniArrayList Change Log

> This just a Implement process about List interface, I want to record the whole process of Implement the list. 

> 仅仅是想完整记录List整个实现过程,从中去发现问题，学习JDK中ArrayLists,如果对你有帮助那就更好了。

<!-- more -->

### 1. Implement Simple 简单实现(暴露问题)

```html 
git checkout MiniArrayList_V0.1
```

As you can see a MiniArrayList in the list folder, and this class Implement the List interface.
You should know this MiniArrayList version just a simple Implement of the List interface,a few methon not to Implement really,such as
`spliterator()`,`listIterator()`etc. 

Another important point you should know that this version hadn't test,I want to test and compare with ArrayList(in JDK) at next step.

此次实现的MiniArrayList仅仅是一个简单的版本，如spliterator(),listIterator()等方法未真正实现，因为我想很多在初期实现先不必考虑太多“高级用法”（对目前的我来说），去除一些学习“噪音”。

这个简单版本仅仅是实现，然后提交记录，仅仅是为了暴露出更多问题，下一步进行测试并且和JDK中ArrayList进行对比，找出自己的不足（处理效率和设计上的优化），为什么这么思考。


### 2. Test and Compare with ArrayList 与ArrayList的对比性测试(发现问题并一步一步解决问题)

**MiniArrayListCompareTest.java**show this step all code.this step just to test and compare with ArrayList,need to keep uniform on the function, not to consider the performance.

In order to review the situation,this test not to fix the MiniArrayList function or performance error(or bad code),so this step
just find the difference on the function. In order to simplify test case just use String to test.

**MiniArrayListCompareTest.java**存储了次阶段测试代码,这个步骤仅仅考虑与ArrayList的对比测试，该修正的修正，首要保证的是找出在功能上的差异，找出对Api的理解错误导致实现错误的地方，最后达到方法正确性的修正，性能是下一步要纳入考虑范围的事情。

为了方便问题的重现，所以这一步仅仅测试找出差异，找出自己疑问的地方，下一版本针对疑问和在方法的功能性上做出修复，并再次测试。

还值得说一句的是虽用泛型实现，但这里为简化测试同意用String类型做测试。


in practice,I found can't find all issue then fix it, it must to step by step. so I plan to test at the same time fix it.

实践证明，不能等发现所有问题后再进行修复解决，这是一个循序渐进的过程(因为后面很多方法也会依赖前面测试的方法，如果不进行修复，明显就是后面依赖的方法全部会错误，下面的size问题你会看到)，所以我决定一边发现一边修复，不过我会版本控制，希望它可以尽可能好的重现。

后面会进行与ArrayList的对比测试，与ArrayList靠齐，并优化自己代码。下一篇[add方法测试并改进代码](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-addTest/)


更多参见: [算法4系列计划](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

原文地址: [https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList/](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList/)

