---
title: 买卖股票的最佳时机 III
date: 2019-01-23 21:22:45
tags: [algorithm]
categories: [algorithm]
---

# 1   算法描述

给定一个数组，它的第 *i* 个元素是一支给定的股票在第 *i* 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 *两笔* 交易。

**注意:** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```
输入: [3,3,5,0,0,3,1,4]
输出: 6
解释: 在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
```

**示例 2:**

```
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。   
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。   
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例 3:**

```
输入: [7,6,4,3,1] 
输出: 0 
解释: 在这个情况下, 没有交易完成, 所以最大利润为 0。
```



# 2 解法

对于任意一天考虑 2 个变量: buy[times-1]  sell[times-1] 买入时最大收益，卖出时候最大收益

buy[times-1]: 在该天第 times - 1  次买入股票可获得的最大收益
sell[times-1]: 在该天第 times -1 次卖出股票可获得的最大收益

buy[times]: 在该天第 times 次买入股票可获得的最大收益
sell[times]: 在该天第 times 次卖出股票可获得的最大收益

分别对四个变量进行相应的更新, 最后 一次卖出sell[maxTimes-1] 就是最大
收益值(sell[times] >= sell[times-1])


```java
public int maxProfit(int[] prices) {
    if (prices != null && prices.length > 1){
        int preBuy = Integer.MIN_VALUE ;
        int preSell = 0;
        int buy = Integer.MIN_VALUE;
        int sell = 0 ;

        for (int p : prices){
            //第1次买
            preBuy = Math.max(preBuy, -p);
            preSell = Math.max(preSell, preBuy + p);
            buy = Math.max(buy, preSell - p);
            sell = Math.max(sell, buy + p );
        }
        return  sell ;
    }
    return 0;
}
```


