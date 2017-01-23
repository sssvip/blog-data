date: 2017/1/18
title: 算法4一起来--一步一步实现List接口--MiniArrayList的index方法测试并改进代码
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

接上一篇博文，[https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-addTest/](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-addTest/)

<!-- more -->

<div id="google_translate_element"></div>

### 4. indexTest
 reference API: 

 - MiniArrayList()  --constitution method
 - indexOf(Object o) --return the index of Object in List,return -1 if not exist
 - lastIndexOf(Object o)  --return the last index of Object in List,return -1 if not exist
 - clear() --clear the data of List
 - isEmpty() --return true if List have data,vice versa 

#### a. indexTest()

first check the code from repository.

```java
git checkout MiniArrayList_indexTest
```

Then in the MiniArrayListCompareTest.java you will find a indexTest method. code as follow:

```java
@Test
public void indexTest() {
    MiniArrayList<String> miniArrayList = new MiniArrayList<String>();
    int dataSize = 5;
    for (int i = 0; i < dataSize; i++) {
      miniArrayList.add("indexTest" + i);
    }
    miniArrayList.add("indexTest" + 2);
    ArrayList<String> arrayList = new ArrayList<String>();
    for (int i = 0; i < dataSize; i++) {
      arrayList.add("indexTest" + i);
    }
    arrayList.add("indexTest" + 2);
    //test print index direct
    System.out.println(miniArrayList.indexOf("indexTest1"));//1
    System.out.println(arrayList.indexOf("indexTest1"));//1
    
    //test print object last index
    System.out.println(miniArrayList.lastIndexOf("indexTest2"));//NullPointerException 
    System.out.println(arrayList.lastIndexOf("indexTest2"));//5
    
    //test print isEmpty
    System.out.println(miniArrayList.isEmpty());//false
    System.out.println(arrayList.isEmpty());//false
    
    //test clear and print isEmpty again
    miniArrayList.clear();
    arrayList.clear();
    System.out.println(miniArrayList.isEmpty());//true
    System.out.println(arrayList.isEmpty());//true
}
```

run the code you will find a Exception:

the code `miniArrayList.lastIndexOf("indexTest2")` will throw NullPointerException,more detail about this issue see this repository issue #4,so I will Fix it.

#### b. fix indexTest() reference issue
Fortunately,this time just a one issue.

- **issue list:**
MiniArrayList lastIndexOf() NullPointerException

- **issue detail:**
MiniArrayList lastIndexOf() in for cycle had a incorrect usage that control the value of i

- **how to solve the issues:?** 

    Open the lastIndexOf method source code will find the error easy.

    ```java
    @Override
    public int lastIndexOf(Object o) {
        for (int i = size; i > 0; i++) {  //--> for (int i = size-1; i >= 0; i--)
          if (data[i].equals(o)) {
            return i;
          }
        }
        return -1;
    }
    ```
    This is clearly mistake to control the value of i. why it will throw NullPointerExcption? Just because first enter for cycle,data[size] just a null value,it should be data[size-1].As we know,null value to invoke equals method will throw NullPointerException.

    So fix it via `for (int i = size-1; i >= 0; i--)`

#### c. indexTest() Run again

checkout the modified code use the command as follow:
```java
check out MiniArrayList_indexTest_fixed
```

run indexTest method in MiniArrayListCompareTest again,you will find the output will consistent with ArrayList's. 

```java
@Test
public void indexTest() {
    MiniArrayList<String> miniArrayList = new MiniArrayList<String>();
    int dataSize = 5;
    for (int i = 0; i < dataSize; i++) {
      miniArrayList.add("indexTest" + i);
    }
    miniArrayList.add("indexTest" + 2);
    ArrayList<String> arrayList = new ArrayList<String>();
    for (int i = 0; i < dataSize; i++) {
      arrayList.add("indexTest" + i);
    }
    arrayList.add("indexTest" + 2);
    //test print index direct
    System.out.println(miniArrayList.indexOf("indexTest1"));//1
    System.out.println(arrayList.indexOf("indexTest1"));//1

    //test print object last index
    System.out.println(miniArrayList.lastIndexOf("indexTest2"));//5
    System.out.println(arrayList.lastIndexOf("indexTest2"));//5

    //test print isEmpty
    System.out.println(miniArrayList.isEmpty());//false
    System.out.println(arrayList.isEmpty());//false

    //test clear and print isEmpty again
    miniArrayList.clear();
    arrayList.clear();
    System.out.println(miniArrayList.isEmpty());//true
    System.out.println(arrayList.isEmpty());//true

}
```


#### d. Conclusion about indexTest()
Although just a little mistake about to control the value of i in the for cycle,but it can show the careless when code.
More careful,more careful.


After fix issue #4,then you can see:[MiniArrayList_indexTest_fixed](https://github.com/sssvip/algorithms4th/tree/MiniArrayList_indexTest_fixed)

更多参见: [算法4系列计划](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

原文地址: [https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-indexTest/](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-indexTest/)

