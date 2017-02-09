date: 2017/01/31
title: 算法4一起来--二分查找树--BinarySearchTree
tags: 
- algorithm
- algorithm 4
- plan
- data structure
- java
- tree
- binarysearchtree
---

As you know, `Binary Search` is a method to search data in a sorted array. `Binary Search Tree` is a search tree derived from the `Binary Search` thinking.

see source code:[BinarySearchTree.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/tree/binarysearchtree/BinarySearchTree.java)

<!-- more -->
#### put data test
```java
/**
* Put test.
*/
@Test
public void putTest() {
    BinarySearchTree tree = new BinarySearchTree();
    Integer[] integers = new Integer[] {100, 90, 110, 80, 95, 105, 120};
    for (Integer i : integers) {
      tree.put(i, i + "-");
    }
    /*
    *  the tree preview as follow
    *
    *           100
    *     90          110
    *  80    95    105   120
    *
    * */
    //the output as follow
    tree.print();//80 90 95 100 105 110 120
}

```
#### delete min node test

```java

/**
* Delete min test.
*/
@Test
public void deleteMinTest() {
	BinarySearchTree tree = new BinarySearchTree();
    Integer[] integers = new Integer[] {100, 90, 110, 80, 95, 105, 120};
    for (Integer i : integers) {
      tree.put(i, i + "-");
    }
    //get the array list of the tree
    List<Node> nodes = tree.toArrayList();
    for (int i = 0; i < nodes.size(); i++) {
      System.out.print(nodes.get(i).key + " ");
    }
    //delete min node and assert it
    Assert.assertEquals(nodes.get(0).key, tree.deleteMin().key); //80
    //delete min node and assert it
    Assert.assertEquals(nodes.get(1).key, tree.deleteMin().key); //90
    //delete min node and assert it
    Assert.assertEquals(nodes.get(2).key, tree.deleteMin().key); //95
    //delete min node and assert it
    Assert.assertEquals(nodes.get(3).key, tree.deleteMin().key); //100
    //delete min node and assert it
    Assert.assertEquals(nodes.get(4).key, tree.deleteMin().key); //105
    //assert min change to 105
    Assert.assertEquals(nodes.get(5).key, tree.min());
}

```

The binary search tree search and put efficiency depending on the order of data input.

#### data example 1: 100 90 80 50 110

the tree will be this behaviour
```html
          100
       90    110
    80
 50
```
#### data example 2: 90 80 50 100 110

the tree will be this behaviour
```html
           90
       80     100
    50            110

```

The tree is not balanced and exist problems of efficiency about search and insert.

#### simple conclusion about BinarySearchTree operation
> All conclusion you can compare with the [source code](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/tree/binarysearchtree/BinarySearchTree.java) to view

- **int size()**: 

get node's size via recursive method

- **Value get(Key key)**: 

return the Value if key equal current node's key; if key less than current node's key find continue in the left subtree vice versa. 

- **void put(Key key,Value value)**: 

put from root node,if root node is null then create a new instance of node as root node. if the key greater than current node's key, put it in left subtree vice versa.

- **Key max()**:

find current node's right node until current node's right node is null, now current node's left node is null return current node otherwise return current node's left nod.

- ** Key min()**:

similar to max method but reverse. 

- **Key floor(Key key)**:

this method indicate that return return the key that greater than or equal it, and it must be in the tree

if current node's key equal then return current node's key. if current node's key greater than key find continue in the left subtree. They both are not true,then find in the right subtree.

- **Key ceiling(Key key)**:

similar to floor method but reverse. 

- **Key select(int k)**:

This method to find the k-th node's key.

The most important thinking come from this line code `select(node.right, k - size - 1);`. It can indicate the relationship about `k` with `size` clearly.

- **int rank(Key key)**:

This method to find the key's rank in the tree.

similar to select method that the most important thinking is the code `1 + size(node.left) + rank(node.right, key);`, you should know the node's size is node's left subnode and right subnode's size sum. But node's right subnode is greater than current node, you need deal carefully.

- **void delete(Key key)**:

Delete method is the most important method in the `Binary Search Tree`. When you delete a node, you must deal the subnode carefully. 
And whatever how to deal the node, the key point is that you must keep the order of the tree.
Generally, you will meet the few kinds situations as follow:
1. current node don't has any subnode
the most simple situation,just set its parent node relational pointer it to null and update relational node's size.
2. current node has left subnode or right subnode only
set current node's parent node relational pointer to current node's left subnode or right subnode, then current node set to null and update relational node's size.
3. current node has left subnode and right subnode
the key thinking is that set current node's right subtree's min node to current node and delete the min node, meanwhile remain the current node's left node and update relational node's size.

and convert above thinking to recursion code:

```java
private Node delete(Node node, Key key) {
    if (node == null) {
      return null;
    }
    int cmp = key.compareTo(node.key);
    if (cmp < 0) {
      node.left = delete(node.left, key);
    } else if (cmp > 0) {
      node.right = delete(node.right, key);
    } else {
      if (node.right == null) {
        return node.left;
      }
      if (node.left == null) {
        return node.right;
      }
      Node temp = node;
      node = min(temp.right);
      node.right = deleteMin(temp.right);
      node.left = temp.left;
    }
    node.N = size(node.left) + size(node.right) + 1;
    return node;
}
```
- **Node  deleteMin()**:

This method to delete min node. The key point is to find left subnode until it is null and delete it,meanwhile you need to set the node(left subnode is null) right subnode to the node's parent node's relational pointer.

- **Node  deleteMax()**:

This method similar to `deleteMin` that to delete max node.

- **Iterator<Node> keys()**:

This method is to change the data to a iterator type data. In order to reduce process, I use `toArrayList().iterator()` in this method.

see source code:[BinarySearchTree.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/tree/binarysearchtree/BinarySearchTree.java)

See more:[Algorightm 4th plan](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

Original Address: [https://blog.dxscx.com/2017/01/31/algorithm/fundamental/tree/binarysearchtree/](https://blog.dxscx.com/2017/01/31/algorithm/fundamental/tree/binarysearchtree/)



        