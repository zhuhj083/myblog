---
title: 红黑二叉查找树
date: 2018-07-03 16:28:09
tags: [algorithm,tree]
categories: algorithm
---
# 2-3查找树
定义：一棵2-3查找树或为一棵空树，或由以下结点组成：
* 2-结点，含有一个键（及其对应的值）和两条链接，左链接指向的2-3树中的键都小于该结点，右链接指向的2-3树中的键都大于该结点。
* 3-结点，还有两个键（及其对应的值）和三条链接，左链接指向的2-3树中的键都小于该结点，中链接指向的2-3树中的键都位于该结点的两个键之间，右链接指向的2-3树中的键都大于该结点。
* 我们将指向一棵空树的链接称为空链接。

![2-3查找树](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/2-3tree.PNG "2-3查找树")

# 红黑二叉查找树

## 基本思想：

我们将树中的链接分为两种类型：
* 红链接：将两个2-结点连接起来构成一个3-结点
* 黑链接：则是2-3树中的普通链接

我们将3-结点表示为由一条左斜的红色链接相连的2个结点

![由一条红色左链接相连的两个2-结点表示一个3-结点](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/redblack_1.PNG "由一条红色左链接相连的两个2-结点表示一个3-结点")

红黑二叉查找树既是二叉查找树，又是2-3查找树，所以我们可以将两个算法的有点结合起来，二叉查找树简洁高效的查找方法和2-3树中高效的平衡插入算法。

# 实现

## 颜色表示
为方便起见，每个结点都只会有一条指向自己的链接（从它的父节点指向它），我们将链接的颜色保存在表示结点的Node数据类型的布尔变量中，如果指向它的链接是红色的，那么该变量为true，黑色为false，我们约定空链接为黑色。

颜色表示的代码实现
```java
private static final boolean RED = true;
private static final boolean BLACK = false;

private Node root ;

private boolean isRed(Node x ){
  if (x == null) {
    return false ;
  }
  return x.color == RED;
}


private class Node{
  Key key ;
  Value val;
  Node left , right ;
  int N ;
  boolean color;

  public Node(Key key , Value val ; int N , boolean color){
    this.key = key;
    this.val = val ;
    this.N = N;
    this.color = color;
  }

}
```

## 旋转与颜色转换

### 左旋转
假设我们有一条红色的右链接，需要被转化为左链接，这个操作叫做 左旋转。
只是将用两个键中较小者作为根结点变为将较大者作为根结点。

![左旋转](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/rotateLeft_1.PNG "左旋转")

```java
private Node rotateLeft(Node h){
  assert isRed(h.right);
  Node x = h.right;
  x.left = h ;
  x.color = h.color;
  h.color = RED ;
  x.N = h.N;
  h.N = size( h.left ) + size( h.right ) + 1 ;
  return x ;
}
```

### 右旋转
原理和左旋转一样

![右旋转](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/rotateRight_1.PNG "右旋转")

```java
private Node rotateRight(Node h){
  assert isRed(h.left);
  Node x = h.left;
  x.right = h ;
  x.color = h.color;
  h.color = RED ;
  x.N = h.N;
  h.N = size( h.left ) + size( h.right ) + 1 ;
  return x ;
}
```

### 颜色转换
当一个结点有两个红色结点的时候，除了需要将子结点的颜色由红变黑，还需要将父节点的颜色由黑变红

![颜色转换](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/flipColors.PNG "颜色转换")

```java
private  void flipColors(Node h){
  assert !isRed(h);
  assert isRed(h.left);
  assert isRed(h.right);
  h.color = RED;
  h.left.color = BLACK;
  h.right.color = BLACK;
}
```

### 根结点总是黑色
颜色转换会使根结点变成红色，这时候我们将根结点设为黑色。注意，每当根结点由红变黑时，树的黑链接高度就会加1


## 插入新键
插入新键总是用红色链接将新节点和它的父结点相连，然后需要的时候，再进行旋转，颜色变换操作。

### 向单个2-结点中插入新键（这个2-结点是根结点）
1. 如果新插入的结点的键小于老键，直接新增一个红色的结点即可
2. 如果新插入的结点的键大于老键，新增的红色结点会产生一条红色的右链接，执行左旋转即可。

### 树底部的2-结点插入新键
同*向单个2-结点中插入新键*

### 向一棵双键树（即一个3-结点）中插入新键
1. 新键小于树中的2个键
2. 两者之间
3. 新键大于树中的两个键

过程如图所示：

![向一棵双键树中新插入一个新键的三种情况](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/redblack_3.PNG "向一棵双键树中新插入一个新键的三种情况")


### 插入实现
总之，只要谨慎地使用左旋转、右旋转和颜色转换这三种简单的操作即可。
1. 如果右子结点是红色的，而左子结点是黑色的，进行左旋转
2. 如果左子结点是红色的，且它的左子结点也是红色的，进行右旋转
3. 如果左右子结点都是红色的，进行颜色转换

```java
public void put(Key key , Value val){
  //查找key，找到则更新其值，否则为它新建一个结点。
  root = put(root，key，val);
  root.color = BLACK; //根结点总是黑色
}

private Node put(Node h , Key key , Value val){
  if (h == null) {
    //标准的插入操作，和父节点用红链接相连
    return new Node(key , val , 1, RED);
  }
  int cmp = key.compareTo(x.key);
  if (cmp < 0 ) {
    h.left = put(h.left,key,val);
  }else if (cmp > 0 ) {
    h.right = put(h.right , key ,val );
  }else{
    h.val = val ;
  }

  //除了这三条if语句，红黑树中put()的递归实现和二叉查找树中的put()的实现完全相同
  if (isRed(h.right) && !isRed(h.left)) {
    h = rotateLeft(h);
  }
  if (isRed(h.left) && isRed(h.left.left) ) {
    h = rotateRight(h);
  }
  if (isRed(h.left) && isRed(h.right)) {
    flipColors(h);
  }

  h.N = size(h .left)+ size (h.right )  + 1 ;
  return h;
}
```
