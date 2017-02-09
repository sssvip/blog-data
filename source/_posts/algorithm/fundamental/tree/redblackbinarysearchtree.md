date: 2017/02/03
title: 算法4一起来--红黑二分查找树--RedBlackBinarySearchTree
tags: 
- algorithm
- algorithm 4
- plan
- data structure
- java
- tree
- redblackbinarysearchtree
- redblacktree
---

The last article discussed the binary search tree, we know that the tree is unbalanced. The appearance is determined by the order in which the data are inserted.

Therefore, the ` Red Black Binary Search Tree` appeared.
see source code:[RedBlackBinarySearchTree.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/tree/redblackbinarysearchtree/RedBlackBinarySearchTree.java)

<!-- more -->

If you want to know more about `Red Black Binary Search Tree`, I suggest you to know 2-3 tree first. Then you will understand RBTree more deeply.

#### About Rotate Node Rules

1. if right subnode is red and left subnode is black,then execute left rotate
2. if left subnode is red and its left subnode also is red, then execute right rotate
3. if left subnode and right subnode both are red, the execute color flip. (current node's color change to red,left subnode and right subnode's color change to black)


see source code:[RedBlackBinarySearchTree.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/tree/redblackbinarysearchtree/RedBlackBinarySearchTree.java)

See more:[Algorightm 4th plan](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

Original Address: [https://blog.dxscx.com/2017/02/03/algorithm/fundamental/tree/redblackbinarysearchtree/](https://blog.dxscx.com/2017/02/03/algorithm/fundamental/tree/redblackbinarysearchtree/)



        