date: 2017/1/17
title: 算法4一起来--一步一步实现List接口--MiniArrayList的add方法测试并改进代码
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

接上一篇博文，[https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList/](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList/)

<!-- more -->

### 3. addTest
reference API: 

- MiniArrayList()  --constitution method
- add(T t) --add data
- toArray()  --output data as array
- toString() --output data as String

#### a. addTest() 添加方法对比小测
```html
@Test
public void addTest() {
    MiniArrayList<String> miniArrayList = new MiniArrayList<String>();
    int dataSize = 5;
    for (int i = 0; i < dataSize; i++) {
      miniArrayList.add("test" + i);
    }
    ArrayList<String> arrayList = new ArrayList<String>();
    for (int i = 0; i < dataSize; i++) {
      arrayList.add("test" + i);
    }
    //test print size
    System.out.println(miniArrayList.size());//stackoverflow before fix #1
    System.out.println(arrayList.size());//5
    
    //test print out direct
    System.out.println(miniArrayList);//fundamentals.list.MiniArrayList@1376c05c  see issue #2
    System.out.println(arrayList);//[test0, test1, test2, test3, test4]
    
    //test print out by array
    System.out.println(Arrays.toString(miniArrayList.toArray()));//[test0, test1, test2, test3, test4, null, null, null, null, null] see issue #3
    System.out.println(Arrays.toString(arrayList.toArray()));//[test0, test1, test2, test3, test4]
}
```

#### b. fix addTest() reference issue 修复addTest方法中涉及的问题 
**issue list:**
 - issue #1: MiniArrayList size() stackoverflow
 - issue #2: MiniArrayList print out direct just the memery address not the data
 - issue #3: MiniArrayList print out by array will output the null value

**issue detail:**
- issue #1 detail: MiniArrayList's size method incorrect recursion lead to stackoverflow error.
- issue #2 detail: System.out.println(MiniArrayList's instance) will output the memory address not similar to ArrayList just output the data.
- issue #3 detail: System.out.println(Arrays.toString(miniArrayList.toArray())) will output the null value but ArrayList not.

more issue detail see: [https://github.com/sssvip/algorithms4th/issues](https://github.com/sssvip/algorithms4th/issues)

**how to solve the issues:?**
 
- issue #1: **solve incorrect recursion**
  ```html
  public int size() {
      return this.size(); --> return this.size; 
    }
  ```
- issue #2: **thinking and solving process**
    1. I guess it must to override the toString method.
    2. I find the ArrayList class's toString to see its toString()
        > but ArrayList don't have any toString shadow... Then I guess it must extend from his super class.I see toString in his super 
         class AbstractList,AbstractCollection etc. I found the method in the AbstractCollection class.
         
        AbstractCollection class's toString() code as follow:
        ```html
        public String toString() {
            Iterator<E> it = iterator();
            if (! it.hasNext())
                return "[]";
    
            StringBuilder sb = new StringBuilder();
            sb.append('[');
            for (;;) {
                E e = it.next();
                sb.append(e == this ? "(this Collection)" : e);
                if (! it.hasNext())
                    return sb.append(']').toString();
                sb.append(',').append(' ');
            }
         }
        ```
    3. Analyze this situation I add toString method in MiniArrayList,code as follow:
       ```html
       @Override
       public String toString() {
           if (this.size < 1) {
             return "[]";
           }
           StringBuilder stringBuilder = new StringBuilder();
           stringBuilder.append("[");
           for (int i = 0; i < this.size; i++) {
             stringBuilder.append(data[i].toString());
             if (i != this.size - 1) {
               stringBuilder.append(',').append(' ');
             }
           }
           return stringBuilder.append("]").toString();
         }
       ```

- issue #3: **thinking and solving process**
    1. Now MiniArrayList toArray() will return the all of Object[],it include null space
    2. I must to see ArrayList class's toArray(),I guess it must handled the data array.
    3. After see ArrayList class's toArray(),That's a surprise,oh my god,I didn't think I should copy the value data to user,I just return the Object[].
    4. Let's see the code of ArrayList class's toArray():
        ```html
        public Object[] toArray() {
            return Arrays.copyOf(elementData, size);
        }
        ```
    5. So I update MiniArrayList toArray() as follow:
        ```html
        @Override
        public Object[] toArray() {
           return data; -->Arrays.copyOf(data, size);
        }
        ```
        
#### c. addTest() Run again
    
```html
@Test
public void addTest() {
    MiniArrayList<String> miniArrayList = new MiniArrayList<String>();
    int dataSize = 5;
    for (int i = 0; i < dataSize; i++) {
      miniArrayList.add("test" + i);
    }
    ArrayList<String> arrayList = new ArrayList<String>();
    for (int i = 0; i < dataSize; i++) {
      arrayList.add("test" + i);
    }
    //test print size
    System.out.println(miniArrayList.size());//5
    System.out.println(arrayList.size());//5
    
    //test print out direct
    System.out.println(miniArrayList);//[test0, test1, test2, test3, test4]
    System.out.println(arrayList);//[test0, test1, test2, test3, test4]
    
    //test print out by array
    System.out.println(Arrays.toString(miniArrayList.toArray()));//[test0, test1, test2, test3, test4]
    System.out.println(Arrays.toString(arrayList.toArray()));//[test0, test1, test2, test3, test4]
}
```

As you can see, in addTest() MiniArrayList's output is consistent with ArrayList.

#### d. Conclusion about addTest()

- forget use System.copyOf() reference API,if it suit you situation, you should use JDK'S API first,not implement again.
  
  reason as follow:
  
  1. the JDK's API through thousands of people check,error rate lower than your implement.
  2. your implement may be not consider many you didn't consider situation,such as the arguments null.

- about the this.size() issue,this is a mistake,so you know the knowledge may be you will code error.
- Maybe use many times ArrayList,but just know a little knowledge about ArrayList,so should see the source code more and more.


After fix issue #1,#2,#3,then you can see:[MiniArrayList_V0.2](https://github.com/sssvip/algorithms4th/tree/MiniArrayList_V0.2)

更多参见: [算法4系列计划](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

原文地址: [https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-addTest/](https://blog.dxscx.com/2017/01/17/algorithm/fundamental/list/arraylist/MiniArrayList-addTest/)

