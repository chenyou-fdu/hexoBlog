title: "#算法笔记# How to Find a Solution 笔记"
date: 2015-04-25 15:25:44
tags: [笔记, 算法]
category: 算法笔记
---

最近在看TopCoder上的Tutorial，这里对[How to Find a Solution](https://www.topcoder.com/community/data-science/data-science-tutorials/how-to-find-a-solution/)这篇做一个简短的笔记。

## BFS搜索
* BFS类的问题一般要求从起点开始，寻找到某个点或者某个状态的最少步数（或者最短距离）。
* 从一个点（状态）到另一个点（状态）的cost一般都是__定值__，要么都是1，要么是其他值。这一点是使用BFS的一个关键特征。
* 通常题目会给一个$N\times{M}$大小的表，其中有些格能走，有些不能，然后要求从开始点到末尾点最短的距离、时间。这个表可能代表了一个迷宫、地图、城市或者其他的东西。
* BFS的时间复杂度一般是线性的，有时是$O(N^2)$或者是$O(NlogN)$。
* 注意不要把BFS问题和动态规划问题搞混，注意之前所述的关键特征。

## Flood Fill 
* 使用BFS寻找所有可以达到的点时，需要使用Flood Fill算法。这种情况下不需要求最短路径或者最小开销。
* 例如一个迷宫有些格可以走有些不能，需要找到所有能从左上起点到的格，就需要使用Flood Fill。这里DFS就不适用了。
* 使用这种算法主要解决的问题特征是有些点能达到，有些点达不到，需要找一个连续区域。
* 可参考这篇文章[Flood Fill Algorithm](http://acm.nudt.edu.cn/~twcourse/ConnectedComponentLabeling.html)。
<!-- more  -->

## Brute Force & Backtracking
* 经典的八皇后问题就是直接暴力法求解。
* 暴力法这类问题有两个特征，一个是约束条件中的量很小，例如八皇后问题只考虑八个棋子；另一个则是要找出所有满足条件的解。
* 回朔法类似于暴力法，可能根据不同问题要优化一些。

## Dynamic Programming
* 动态规划的问题规模不会很大，也不会很小。
* 从一个状态总有一种或者多种方法、规则转移到下一个状态，而且下一个状态只依赖于上一个状态。
* 参考[Dynamic Programming – From Novice to Advanced](https://www.topcoder.com/community/data-science/data-science-tutorials/dynamic-programming-from-novice-to-advanced/)这篇Tutorial。