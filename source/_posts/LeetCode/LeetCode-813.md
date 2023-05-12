---
title: 以LeetCode_813为例,从递归到记忆化递归到DP
author: Ray
top: true
cover: false
date: 2023-05-13 16:14:10
categories: 技术
mathjax: true
tags: 
  - 力扣之旅
  - java
  - Dp
  - 递归
---

### Q:

给定数组 `nums` 和一个整数 `k` 。我们将给定的数组 `nums` 分成 **最多** `k` 个相邻的非空子数组 。 **分数** 由每个子数组内的平均值的总和构成。

注意我们必须使用 `nums` 数组中的每一个数进行分组，并且分数不一定需要是整数。

返回我们所能得到的最大 **分数** 是多少。答案误差在 `10-6` 内被视为是正确的。

<!-- more -->

### Input:

```
输入: nums = [9,1,2,3,9], k = 3
输出: 20.00000
解释: 
nums 的最优分组是[9], [1, 2, 3], [9]. 得到的分数是 9 + (1 + 2 + 3) / 3 + 9 = 20. 
我们也可以把 nums 分成[9, 1], [2], [3, 9]. 
这样的分组得到的分数为 5 + 2 + 6 = 13, 但不是最大值.
```

### S:

#### 一

解决这个问题最直观的想法是，我们可以列举每个案例，最后得到最优答案。所以，我们可以通过递归枚举来解决问题，通过枚举每个分区的情况，得到最终的最大值 
递归代码如下：

```java
class Solution {
    public double largestSumOfAverages(int[] A, int K) {
        return dfs(A, 0, K);
    }
    private double dfs(int A[] ,int index,int K){
        if(K==0){
            return 0.0;
        }
        if(K==1){// k=1时直接返回整个数组的均值
            int sum = 0;
            for(int i=index;i<A.length;i++){
                sum+=A[i];
            }
            return (double)sum/(A.length-index);
        }
        double sum = 0.0;
        double res = 0.0;
        for(int i=index;i<=A.length-K;i++){
            sum+=A[i];//枚举每个分离点
            double avage = sum/(i-index+1);
            double temp = dfs(A,i+1,K-1);//下一个组的均值
            res = Math.max(res, avage+temp);//取最大
        }
        return (double)res;
    }
}

```

#### 二

​		在本题中，递归会超时，一般情况下，递归都不是最优解，因为他会进行许多重复运算，所以，我们可以使用记忆化递归，所谓记忆化就是使用一个数组记录过已经得到的递归值，当再次进入该分支后可以快速得到解。 在理解了上面的递归代码后稍微修改一下可以很容易得到记忆化递归代码

```java
class Solution {
    double [][] memo ;
    public double largestSumOfAverages(int[] A, int K) {
        this.memo = new double [A.length+1][K+1];
        return dfs(A, 0, K);
    }
    private double dfs(int A[] ,int index,int K){
        if(K==0){
            return 0.0;
        }
        if(memo[index][K]!=0.0)   return memo[index][K];
        if(K==1){
            int sum = 0;
            for(int i=index;i<A.length;i++){
                sum+=A[i];
            }
            memo[index][K] = (double)sum/(A.length-index);
            return memo[index][K];
        }
        double sum = 0.0;
        double res = 0.0;
        for(int i=index;i<=A.length-K;i++){
            sum+=A[i];
            double avage = sum/(i-index+1);
            // double temp = dfs(A,i+1,K-1);
            // memo[i+1][K-1] = temp;
            memo[i+1][K-1] = dfs(A,i+1,K-1);
            res = Math.max(res, avage+memo[i+1][K-1]);
        }
        memo[index][K] =res;
        return (double)res;
    }
}
```

#### 三

​		记忆递归实际上与动态规划非常相似，只是一个是自上而下的表法，另一个是自下而上的表法。基于记忆递归的思想，我们可以将记忆递归改写为DP

```java
public double largestSumOfAverages(int[] A, int K) {
        double[][] dp = new double[A.length+1][K+1];
        double [] preSum = new double[A.length+1];
        for(int i=0;i<A.length;i++){
            preSum[i+1]= preSum[i]+A[i];
            dp[i+1][1] = preSum[i+1]/(i+1);//初始化
        }
        for(int i=1;i<=A.length;i++){
            for(int j=2;j<=Math.min(K, i);j++){
                //dp[i][j]的最大均值 应该是 dp[i'][j-1]的均值+i'-i的均值和  中所有的可能中的最大值
                for(int t = 0;t<i;t++){
                    dp[i][j] = Math.max(dp[i][j], dp[t][j-1]+(preSum[i]-preSum[t])/(i-t));
                }
            }
        }
        return dp[A.length][K];
    }

```

