date: 2017/02/07
title: 算法4一起来--图--Graph
tags: 
- algorithm
- algorithm 4
- plan
- data structure
- java
- graph
---

The Graph Process Pattern
The representation and implement of Graph was separated.We will create many implement about each kind of graph process algorithm.
<!-- more -->

### Representation

#### Graph
This implement edition was not include:
1. add a vertex
2. delete a vertex
3. delete an edge
4. check v-w edge whether exist

The reason that this edition did not implement the methods above:
1. Usage case sometimes needn't add and delete a vertex.
2. Simplify implement and more focus on Graph research process.

More details:[Graph.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/graph/Graph.java)

### Implement

#### DepthFirstSearch
main methods:
1. public boolean marked(int v) {...}

    Mark the vertex was visited.
    
2. public int count() {...}

    The vertexes connected `s` total count.

The key thinking is recursion. When you reach a vertex, mark it marked.And visit all connect with current vertex,
until visit all of those.

More details:[DepthFirstSearch.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/graph/DepthFirstSearch.java)

#### DepthFirstPaths
main methods:
1. public boolean hasPathTo(int v) {...}

    return the result that start vertex is reachable to the vertex `v`
    
2. public Iterable<Integer> pathTo(int v) {...}

    return one of the all paths that start vertex to the vertex `v`
    
The key thinking is that `DepthFirstPaths` hold an array called `edgeTo`.

The array to store the last vertex, e.g. if 0 can reach 2 by vertex 1 (0->1->2),then store 1 at edgeTo[2].

e.g.
```html
paths:
0->1
1->6
0->2
2->3
2->4
4->5
```


the graph to a tree reachable represent:
```html
     0
  1      2
    6  3   4
              5
```

if use 0 as start vertex,the array of edgeTo data as follows；

edgeTo as follows:
```html
index->last reachable vertex
0->0
1->0
2->0
3->2
4->2
5->4
6->1
```
so edgeTo----> {0,0,0,2,2,4,1}

More details:[DepthFirstPaths.java](https://github.com/sssvip/algorithms4th/blob/master/src/fundamentals/graph/DepthFirstPaths.java)


See more:[Algorightm 4th plan](https://blog.dxscx.com/2017/01/12/algorithm/plan/)

Origin Adress: [https://blog.dxscx.com/2017/02/07/algorithm/fundamental/graph/graph/](https://blog.dxscx.com/2017/02/07/algorithm/fundamental/graph/graph/)



        