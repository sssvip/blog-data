date: 2017/1/22
title: 算法4一起来--排序--Sortable抽象类
tags: 
- algorithm
- algorithm 4
- plan
- data structure
- java
- sort
- sortable
---


There is no surprise,reading progress more faster than implement. Now,I was reading the section 2.5 in algorithm 4th.But I am just start plan to implement the sort part.I very like a saying:"slower is faster". So just do yourself better.

<!-- more -->

<div id="google_translate_element"></div>

#### why to define a abstract class called `Sortable`?

1. Extract the common code part of sort,reduce code repeat

2. More specifical to implement a kind of sort,not mess

This is a good method to implement sort. Because I don't mess more,I just divide implement to more step. I just focus per step how to implement,not here and there meanwhile.

As sort, it can be divide into two necessary steps that compare and exchange.

#### why to define a abstract class called Sortable?
I try to define a interface not a abstract class,then I found that not a good practice.

Although define a Sortable interface can ensure the subclass must implement each method.If I implement the sortable interface, I must to implement the each method in `QucikSort`,`MergeSort`,`SelectSort`,etc. But think it seriously,not need implement each method in a specific Sort Class. Many method just common code for many specific Sort Class.

So I remember it can use abstract class to define,if not specific method, just use default implement. If some method not common code,it must to implement specific,it can declare a abstract method.Then in specific Sort Class implement it specifically.

```java
/**
 * The abstract class Sortable.
 */
public abstract class Sortable {
	//compare
	public boolean less(Comparable v, Comparable w) {
		...omission some code
	}

	//exchage
	public void exch(Comparable[] a, int i, int j) {
		...omission some code
	}

	//print to show
	public void show(Comparable[] a) {
		...omission some code
	}

	//check sorted
	public boolean isSorted(Comparable[] a) {
		...omission some code
	}

	//sort from 0 to a.length
	public void sort(Comparable[] a) {
		...omission some code
	}

	//specific sort class implement specifically
	public abstract void sort(Comparable[] a, int fromIndex, int toIndex);
	
	//check index offset
	public void checkOffset(Comparable[] a, int fromIndex, int toIndex) {

	}
}
```

#### Two most important methods in Sortable Class
1. less method
```java
  /**
   * compare value of v and w,if v less than w return true, vice versa
   *
   * @param v the v
   * @param w the w
   * @return the boolean
   */
  public boolean less(Comparable v, Comparable w) {
    return v.compareTo(w) < 0;
  }
```
2. exch method
```java
  /**
   * exchange index of i,j 's data of array a
   *
   * @param a the a
   * @param i the
   * @param j the j
   */
  public void exch(Comparable[] a, int i, int j) {
    Comparable t = a[i];
    a[i] = a[j];
    a[j] = t;
  }
```
see source code:[Sortable.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/sort/common/Sortable.java)

See more:[Algorightm 4th plan](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

Origin Adress: [https://blog.dxscx.com/2017/01/22/algorithm/fundamental/sort/sortable/](https://blog.dxscx.com/2017/01/22/algorithm/fundamental/sort/sortable/)



        