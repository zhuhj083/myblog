---
title: 堆排序
date: 2018-07-02 11:52:57
tags: [algorithm,sort]
categories: algorithm
---
# 二叉堆
## 定义
定义：当一颗二叉树的每个几节点都大于等于它的子节点时，它被称为堆有序。

定义：二叉堆是一组能够用**堆有序**的**完全二叉树**排序的元素，并在数组中按照层级存储（a[0]不使用。）
<br>在位置k的节点的父节点位置为k/2 ；
<br>位置k的节点的左右子节点分别是 2k,2k+1;

## 插入元素、删除元素

**比较大小 交换**
```java
private boolean less (int i , int j){
  return pq[i].compareTo(pq[j]) < 0 ;
}

private void exch(int i , int j){
  T t =pq[i];
  pq[i] = pq[j];
  pq[j] = t ;
}
```


**插入元素**
将新元素添加到数组的尾，增加堆的大小，并让这个元素上浮到合适的位置

上浮算法：
```java
private void swim(int k){
  while( k > 1 &&  less( k/2 , k )){
    exch(k/2,k);
    k = k/2 ;
  }
}
```

**删除最大元素**
我们删除a[0],并将数组的最后一个元素放到顶端，减小堆的大小，并将这个元素下沉到合适的位置

下沉算法：
```java
private void sink(int k){
  while ( 2*k <= N ){//N为数组长度
    int j = 2*k ;
    //比较左右子节点哪个大，取大的那个
    if (j < N && less(j , j +1 )) {
      j++;
    }
    if (!less(k,j)) {
      break;
    }
    exch(k, j);
    k = j ;
  }
}
```

<!--more-->

# 堆排序

```java
public class HeapSort {

    public static void main(String[] args) {
        String a [] = {"S","O","R","T","E","X","A","M","P","L","E"};
        sort(a);
        System.out.println(isSorted(a));
        show(a);
    }

    public static void sort(Comparable[] a){
       int N = a.length;

       for ( int k = N / 2 ; k >= 1 ; k-- ){ //从最后一个子树开始
           sink( a ,k , N );
       }

       //此时，最大的元素在根节点
       //从根节点开始，将最大的元素依次交换到最后
       while (N > 1 ){
           exch(a,1,N--);
           sink(a,1,N);
       }

    }

    /**
     * 将a[i] 到 a[j] 之间的元素排序
     */
    private static void sink(Comparable[] a , int k , int n ){
        while( 2 * k <= n ){
            int j = 2 * k ;

            if (j < n && less(a,j , j +1)){
                j++;   //寻找两个子几点中大的那个
            }
            if (!less(a,k,j)){  //如果父结点比大的那个子结点大。循环结束
                break;
            }
            exch(a,k ,j);    //交换
            k = j ;       //下沉一级
        }
    }

    private static void exch(Comparable[] a ,int i,int j){
        Comparable t = a[i-1];
        a[i-1] = a[j-1];
        a[j-1] = t;
    }

    private static boolean less(Comparable[] a,int i,int j){
        return a[i-1].compareTo(a[j-1]) < 0 ;
    }

    private static void show(Comparable[] a){
        //单行中打印数组
        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i] + " ");
        }
        System.out.println();
    }

    public static boolean isSorted(Comparable[] a){
        for (int i = 1 ; i < a.length; i++) {
            if (less(a,i+1 ,i))
                return false;
        }
        return true;
    }

}

```
