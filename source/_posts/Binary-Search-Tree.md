---
title: 二叉查找树 Binary Search Tree
date: 2018-07-03 12:06:57
tags: [algorithm,Tree]
categories: algorithm
---
# 定义
一颗二叉查找树（BST）是一颗二叉树，其中每个结点都含有一个Comparable的键（以及相关联的的值），且每个结点的键都大于其左子树的结点的键而小于右子树的结点的键。

# 基本实现

## 结点
我们嵌套定义了一个*私有类*来表示二叉查找树的一个结点，每个结点都含有一个键、一个值、一条左链接、一条右链接、和一个结点计数器

```java
private class Node{
  private Key key;//键
  private Value val;//值
  private Node left,right;//指向左右子树的链接
  private int N ;//以该结点为根结点的子树中的结点总数

  public Node(Key key , Value val , int N){
    this.key = key ;
    this.val = val ;
    this.N = N ;
  }

}
```

私有方法size()会将空链接的值当做0；这样我们通过以下公式来计算结点x的N值
size(x) = size(x.left) + size(x.right) + 1 ;
```java
public int size(){
  return size(root);
}

private int size(Node x){
  if (x == null) {
    return 0 ;
  }else{
    return x.N;
  }
}
```

## 查找
一般来说，在符号表（二叉查找树是一个符号表）中查找一个键，可能有两种结果：
如果含有该键的结点存在于表中，我们的查找命中，返回结点的值；
否则未命中，返回null。

在二叉查找树中查找一个键的递归算法：
1. 如果树是空的，则查找未命中
2. 如果被查找的键和根结点的键相等，查找命中
3. 否则我们就递归地在适当的子树中查找

```java
public Value get(Key key){
  return get(root,key);
}

private Value get(Node x , Key key){
  //在以x结点为根结点的子树中查找并返回key所对应的值
  //如果找不到则返回null
  if (x == null) {
    return null ;
  }
  int cmp = key.compareTo(x.key);
  if (cmp < 0 ) {
    return get(x.left,key);
  }else if (cmp > 0 ) {
    return get(x.right,key);
  }else{
    return x.val;
  }
}
```

## 插入
在二叉查找树中递归插入一个结点的算法：
1. 如果树是空的，就安徽一个含有该键值对的新节点
2. 如果被查找的键小于根结点的键，我们会继续在左子树中插入该键
3. 否则在右子树中插入该键

```java
public void put(Key key , Value val){
  put(root,key,val);
}

private Node put(Node x , Key key,Value val){
  if (x == null) {
    return new Node(Key,val,1);
  }
    int cmp = key.compareTo(x.key);
  if (cmp < 0 ) {
    x = put(x.left,key,val);
  }else if (cmp > 0 ) {
    x = put(x.right,key,val);
  }else{
    x.val = val;
  }
  x.N = size(x.left) + size(x.right) + 1 ;
  return x ;
}
```

## 有序性相关的方法
二叉查找树可以保持**键的有序性**

### 最小键和最大键
最小键：
1. 如果根结点的左链接为空，那么一棵二叉查找树中最小的键就是根结点；
2. 如果左链接非空，那么树中最小的键就是左子树中的最小键

最大键：
1. 如果根结点的右链接为空，那么一棵二叉查找树中最大的键就是根结点；
2. 如果右链接非空，那么树中最大的键就是右子树中的最大键

递归实现查找最小键、最大键：
```java
public Key min(){
  return min(root).key;
}

private Node min(Node x){
  if (x.left == null)
    return x;
  else
    return min(x.left);
}

public Key max(){
  return max(root).key;
}

private Node max(Node x){
  if (x.right == null)
    return x;
  else
    return max(x.right);
}
```

### 向上取整和向下取整
向上取整：
1. 如果给定的key小于二叉查找树的根结点的键，那么*小于等于*key的最大键floor(key)一定在根结点的左子树中
2. 如果给定的键key大于二叉查找树的根结点，那么只有当根结点右子树中存在小于等于key的结点时，小于等于key的最大结点才会出现在右子树中，否则根结点就是小于等于key的最大键。

```java
public Key floor(Key key){
  Node x = floot(root,key);
  if (x == null) {
    return null ;
  }
  return x.key;
}

public Node floor(Node x , key){
  if (x == null) {
    return null ;
  }
  int cmp = key.compareTo(x.key);
  if (cmp == 0 ) {
    return x ;
  }
  if (cmp < 0 ) {
    return floor(x.left,key);
  }
  Node t = floor(x.right,key);
  if (t != null) { //当根结点右子树中存在小于等于key的结点时
    return t ;
  }else{
    return x ;
  }
}
```

向下取整：
```java
public Key ceiling(Key key){
    Node x = ceiling(root,key);
    if (x == null )
        return null;
    else
        return x.key;
}

private Node ceiling(Node x ,Key key){
  if (x == null)
      return null;
  int cmp = key.compareTo(x.key);
  if (cmp == 0 )
      return x;
  if (cmp > 0 )
      return ceiling(x.right,key);
  Node t = ceiling(x.left,key);
  if (t != null)
      return t ;
  else
      return x;
}
```

### 排名 select(int k)、rank(Key key)
假设我们要找排名为k的键（即树中正好有k个小于它的键）：
1. 如果左子树中的结点树t大于k，那么我们就递归地在左子树中查找排名为k的键
2. 如果t=k,我们就返回根结点键（根结点就是排名k）
3. 如果t<k,我们递归地在右子树中查找排名为 (k-t-1)的键

```java
public Key select(k){
  return select(root,k).key;
}

private Node select(Node x,int k){
  //返回排名为k的结点
  if (x==null) {
    return null;
  }
  int t = size(x.left);
  if (t > k) {
    return select(x.left,k);
  }else if (t < k ) {
    return select(x.right,k-t-1);
  }else{
    return x;
  }
}
```

返回给定Key的排名：
1. 如果给定的key和根结点的键相等，我们返回左子树的结点总数t；
2. 如果给定的键小于根结点，我们会返回该键在左子树中的排名（递归计算）；
3. 如果给定的键大于根结点，我们返回 t+1 + 它在右子树中的排名（递归计算）。

```java
public int rank(Key key){
  return rank(root,key);
}
private int rank(Node x , Key key){
  if (x == null ) {
    return 0 ;
  }
  int cmp = key.compareTo(x.key);
  if (cmp < 0) {
    return rank(x.left,key);
  }else if ( cmp > 0) {
    return size(x.left) + 1 + rank(x.right,key);
  }else{
    return x ;
  }
}
```
## 删除操作

### 删除最大键和删除最小键
删除最小键：
*  我们需要不断深入根结点的左子树，直到遇见一个空链接，然后将指向该结点的链接指向该结点的右子树

```java
public void deleteMin(){
  deleteMin(root);
}
private Node deleteMin(Node x ){
  if (x.left == null ) {
    return x.right;
  }
  x.left = deleteMin(x.left);
  x.N = size(x.left) + size(x.right) + 1 ;
  return x ;
}
```

删除最大键：
* 不断深入根结点的右子树，直到遇见一个空链接，然后将指向该结点的链接指向该结点的左子树

```java
public void deleteMax(){
  deleteMax(root);
}
private Node deleteMax(Node x){
  if (x.right == null) {
    return x.left;
  }
  x.right = deleteMax(x.right);
  x.N = size(x.left) + size(x.right) + 1 ;
  return x ;
}
```

### 删除操作
删除一个结点t：
1、找到以这个结点t为根结点的子树的最小结点x=min(t),x替换该结点t
2、x的左子树指向 被删结点t的左子树
3、x的右子树指向 删除了最小结点的原来的t的右子树（deleteMin(t.right)）

```java
public void delete(Key key){
  root = delete(root,key);
}

private Node delete(Node x , Key key){
  if (x == null) {
    return null ;
  }
  int cmp = key.compareTo(x.key);
  if (cmp < 0 ) {
    x.left = delete(x.left,key);
  }else if (cmp > 0 ) {
    x.right = delete(x.right,key);
  }else{
    //找到这个结点t
    if (x.right == null ) {
      return x.left ;
    }
    if (x.left == null ) {
      return x.right ;
    }

    Node t = x ;
    x = min(t.right);
    x.right = deleteMin(t.right);
    x.left = t.left;
    x.N = size(x.left) + size(x.right) + 1 ;
    return x ;
  }
}
```

## 中序遍历
将二叉排序树中的所有结点的键按顺序打印出来
```java
private void print(Node x){
  if (x == null) {
    return ;
  }
  print(x.left);
  System.out.println(x.key);
  print(x.right);
}
```

## 范围查找
返回指定范围内的键的keys()方法
将所有落在指定范围以内的键加入一个队列Queue，并跳过哪些不可能含有所查找键的子树
```java
public Iterable<Key> keys(){
  return keys(min(),max());
}

public Iterable<Key> keys(Key lo , Key hi){
  Queue<Key> queue = new Queue<Key>();
  keys(root,queue,lo,hi);
  return queue;
}

private void keys(Node x , Queue queue ,Key lo , Key hi ){
  if (x == null) {
    return ;
  }
  int cmplo = lo.compareTo(x.key);
  int cmphi = hi.compareTo(x.key);
  if (cmplo < 0 ) {
    keys(x.left,queue,lo , hi);
  }
  if (cmplo <= 0 && cmphi >= 0 ) {
    queue.enqueue(x.key);
  }
  if (cmphi > 0 ) {
    keys(x.right , queue,lo ,hi);
  }
}
```

# 性能分析
在一棵二叉查找树中，所有操作在最坏情况下所需的时间都和树的高度成正比。

# 完整实现代码

```java
package com.practice.algorithms;

import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;

public class BinarySearchTree<Key extends Comparable<Key>,Value>{

    private Node  root ; //二叉查找树的根节点

    private class Node{
        private Key key;
        private Value val;
        private Node left,right;
        private int N;

        public Node(Key key, Value val,int n) {
            this.key = key;
            this.val = val;
            N = n;
        }
    }

    public  int size(){
        return size(root);
    }

    private int size(Node x){
        if (x == null){
            return 0 ;
        }else{
            return x.N;
        }
    }

    public Value get(Key key){
        return get(root,key);
    }

    private Value get(Node x , Key key){
        //在以x为根节点的子树中查找并返回key所对应的值
        //如果找不到，则返回null
        if (x == null){
            return null;
        }
        int cmp = key.compareTo(x.key);
        if (cmp < 0 ){
            return get(x.left,key);
        }else if (cmp > 0 ){
            return get(x.right,key);
        }else{
            return x.val;
        }
    }

    public void put(Key key ,Value val){
        //查找可key，找到则更新它的值，否则为它创建一个新的节点
        root = put(root,key,val);
    }

    /**
     * 递归实现
     */
    private Node put(Node x , Key key , Value val){
        //如果key存在与以x为根节点的子树中，则更新它的值
        //否则将以key和val为键值对的新节点插入到该子树中
        if (x == null) {
            return new Node(key, val, 1);
        }
        int cmp = key.compareTo(x.key);
        if (cmp < 0 ){
            x.left = put(x.left,key,val);
        }else if (cmp > 0 ){
           x.right = put(x.right,key,val);
        }else{
            x.val = val ;
        }
        x.N = size(x.left) + size(x.right) + 1;
        return x;
    }

    public Key min(){
        return min(root).key;
    }

    private Node min(Node x){
        if (x.left == null)
            return x;
        else
            return min(x.left);
    }

    public Key max(){
        return max(root).key;
    }

    private Node max(Node x){
        if (x.right == null)
            return x;
        else
            return max(x.right);
    }

    public Key floor(Key key){
        Node x = floor(root,key);
        if (x == null )
            return null;
        else
            return x.key;
    }

    private Node floor(Node x ,Key key){
        if (x == null)
            return null;
        int cmp = key.compareTo(x.key);
        if (cmp == 0 )
            return x;
        if (cmp < 0 )
            return floor(x.left,key);
        Node t = floor(x.right,key);
        if (t != null)
            return t ;
        else return x;
    }

    public Key ceiling(Key key){
        Node x = ceiling(root,key);
        if (x == null )
            return null;
        else
            return x.key;
    }

    private Node ceiling(Node x ,Key key){
        if (x == null)
            return null;
        int cmp = key.compareTo(x.key);
        if (cmp == 0 )
            return x;
        if (cmp > 0 )
            return ceiling(x.right,key);
        Node t = ceiling(x.left,key);
        if (t != null)
            return t ;
        else
            return x;
    }

    public Key select(int k){
        return select(root,k).key;
    }

    /**
     * 以x为根节点，查找排名为k的节点
     * 如果左子树中的结点树t大于k，继续递归地在左子树中查找排名为k的键
     * 如果t=k，返回根节点中的键
     * 如果t<k。那么我们就递归地在右子树中查找排名为（k-t-1）的键
     */
    private Node select(Node x,int k){
        if (x == null)
            return null;
        int t = size(x.left);
        if ( t > k ) {
            return select(x.left, k);
        }else if (t < k ){
            return select(x.right,k-t-1);
        }else {
            return x;
        }
    }

    public int rank(Key key){
        return rank(root,key);
    }

    private int rank(Node x,Key key){
        //返回以x为根节点的子树中小于x.key的键的数量
        if (x== null )
            return 0 ;
        int cmp = key.compareTo(x.key);
        if (cmp < 0 ){
            return rank(x.left,key);
        }else if (cmp > 0){
            return rank(x.right,key);
        }else{
            return size(x.left);
        }
    }

    /**
     * 不断深入根节点的左子树，直至遇见一个空链接
     * 然后将指向该节点的链接指向该节点的右子树（只要在递归调用中返回它的右链接即可）
     */
    public void deleteMin(){
        root = deleteMin(root);
    }

    private Node deleteMin(Node x){
        if (x.left == null)
            return x.right;
        x.left = deleteMin(x.left);
        x.N = size(x.left) + size(x.right) + 1;
        return x ;
    }

    public void deleteMax(){
        root = deleteMax(root);
    }

    private Node deleteMax(Node x){
        if (x.right == null)
            return x.left;
        x.right = deleteMin(x.right);
        x.N = size(x.left) + size(x.right) + 1;
        return x ;
    }

    public void delete(Key key){
        root = delete(root,key);
    }

    private Node delete(Node x ,Key key){
        if (x == null)
            return null;
        int cmp = key.compareTo(x.key);
        if (cmp < 0 )
            x.left = delete(x.left,key);
        else if (cmp > 0)
            x.right = delete(x.right,key);
        else{
            //找到了这个要删除的节点了 x
            if (x.right == null)
                return x.right;
            if (x.left == null )
                return x.right;
            Node t = x ;
            x = min(t.right);
            x.right = deleteMin(t.right); //将x的右子树最小节点删除
            x.left = t.left;
        }
        x.N = size(x.left) + size(x.right) + 1;
        return x ;
    }

    private void print(Node x){
        if ( x == null)
            return ;
        print(x.left);
        StdOut.println(x.key);
        print(x.right);
    }

    public Iterable<Key> keys(){
        return keys(min(),max());
    }

    public Iterable<Key> keys(Key lo , Key hi){
        Queue<Key> queue = new Queue<Key>();
        keys(root,queue,lo,hi);
        return queue;
    }

    private void keys(Node x,Queue<Key> queue,Key lo , Key hi){
        if (x == null )
            return ;
        int cmplo = lo.compareTo(x.key);
        int cmphi = hi.compareTo(x.key);
        if (cmplo < 0)
            keys(x.left,queue,lo,hi);
        if (cmplo <= 0 && cmphi >= 0 ){
            queue.enqueue(x.key);
        }
        if (cmphi > 0 )
            keys(x.right,queue,lo,hi);
    }

    public static void main(String[] args) {
        BinarySearchTree<String,Integer> bst = new BinarySearchTree<String,Integer>();
        for (int i = 0; i < 1000000; i++) {
            String key = StdRandom.uniform(0,1000)+"";
            int val = 0 ;
            if (bst.get(key)!=null){
                val = bst.get(key) + 1;
            }
            bst.put(key,val);
        }

        for (String key:bst.keys()) {
            System.out.println("key="+key+",value="+bst.get(key));
        }

        System.out.println(bst.size());
        System.out.println(bst.min());
        System.out.println(bst.max());
        System.out.println(bst.floor("50"));

    }
}

```
