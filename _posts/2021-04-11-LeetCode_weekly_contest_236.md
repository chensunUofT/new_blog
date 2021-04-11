---
layout: post
title: LeetCode周赛#236
date: 2021-04-11
categories: [technology]
tags: [LeetCode, algorithm, coding]
comments: true
toc: true
excerpt: 记第一次四题全A
---

# 成绩
用时1:22:18（其中包括15min的罚时，哭），排名337/12115（实际完赛人数9038），可圈可点。  
下一个目标，挺进1:00:00大关！

# 第一题
<https://leetcode.com/contest/weekly-contest-236/problems/sign-of-the-product-of-an-array/>  
有零则返回零，有负数则变号。
```python
class Solution:
    def arraySign(self, nums: List[int]) -> int:
        res = 1
        for num in nums:
            if num == 0:
                return 0
            elif num < 0:
                res *= -1
        return res
```

# 第二题
<https://leetcode.com/contest/weekly-contest-236/problems/find-the-winner-of-the-circular-game/>  
是一个大一C语言课上学过的[约瑟夫环问题](https://zh.wikipedia.org/wiki/%E7%BA%A6%E7%91%9F%E5%A4%AB%E6%96%AF%E9%97%AE%E9%A2%98)，没想到自己居然写了那么久，主要是纠结是用数值还是用下标，是0到N-1还是1到N，来来回回改了好多次，其实没那么复杂……  
```python
class Solution:
    def findTheWinner(self, n: int, k: int) -> int:
        players = list(range(1, n+1))
        i = 0
        while len(players) > 1:
            i = (i + k-1) % len(players) 
            players.pop(i)
        return players[0]
```

# 第三题
<https://leetcode.com/contest/weekly-contest-236/problems/minimum-sideway-jumps/>
刚看到这个题的时候就觉得是Greedy或者DP中的一个，肯定希望它是Greedy因为好想一点，然而事与愿违，Greedy是行不通的，只能是DP，所幸（在被罚了十分钟之后）做出来了。

我的DP思路：用`dp[i][j]`表示青蛙走到第`i`列(`0`到`N-1`，因为第`N`列不用考虑)、第`j`条道(`0`到`2`，对应`obstacles`值的`1`到`3`)所需要的最少的side jump（横跳）数。  
* 若`(i,j)`有石头，则不可能走到，`dp[i][j]`值为无穷；
* 若`(i,j)`无石头，则需要根据前一列`i-1`的`dp`值计算`dp[i][j]`：
	* 若`(i-1,j)`无石头，则显然可以从`(i-1,j)`直接走到`(i,j)`，因此`dp[i][j] = dp[i-1][j]`
	* 若`(i-1,j)`有石头：
		* 若第`i`列也有石头（因为每一列最多有一块石头，因此此时第`i-1`列和第`i`列的石头一定不在同一条道），则需要从这一列除去石头的另一条道横跳至此，那一条道的下标`lane`应为`{0，1，2}`三个数中既不是`obstacles[i]-1`（减一是因为obstacles的`{1，2，3}`对应下标`{0,1,2}`）也不是`obstacles[i-1]-1`的那个，即`dp[i][j] = 1 + dp[i][lane]`，而`lane`道上的第`i`和`i-1`列均无障碍，所以`dp[i][lane] = dp[i-1][lane]`，结合两个式子，得到这个情况下的递推式`dp[i][j] = 1 + dp[i-1][lane]`
		* 若第`i`列没有石头，则`(i,j)`可从同一列另外两点中的任意一点横跳到达，而这两点在前一列均没有障碍，因此`dp[i][j] = 1 + min(dp[i-1])`

最后，返回最后一列中的最小值即可。
```python
class Solution:
    def minSideJumps(self, obstacles: List[int]) -> int:
        N = len(obstacles) - 1
        dp = [[float('inf') for _ in range(3)] for _ in range(N)]
        dp[0][1] = 0
        dp[0][0] = dp[0][2] = 1
        for i in range(1, N):
            for j in range(3):
                if j != obstacles[i]-1:
                    if j != obstacles[i-1] - 1:
                        dp[i][j] = dp[i-1][j]
                    else:
                        if obstacles[i]:
                            lane = 3 - (obstacles[i]-1) - (obstacles[i-1] - 1)
                            dp[i][j] = 1 + dp[i-1][lane]
                        else:
                            dp[i][j] = 1 + min(dp[i-1])
        return min(dp[-1])
```
优化：因为`dp[i]`只取决于`dp[i-1]`，所以可以简化为一维DP：若`lane`处有障碍，则**首先**（避免同一列更新时利用这个未更新的值）将其`dp`值赋值为无穷；对于不含障碍的`lane`，可以从上一列前进或同一列横条而到达，故更新`dp`的方程为`dp[lane] = min(dp[lane], 1 + min(dp))`。
```python
class Solution:
    def minSideJumps(self, obstacles: List[int]) -> int:
        dp = [1,0,1]
        for ob in obstacles:
            if ob:
                dp[ob-1] = float('inf')
            for lane in range(3):
                if lane != ob - 1:
                    dp[lane] = min(dp[lane], 1 + min(dp))
        return min(dp)
```

# 第四题
<https://leetcode.com/contest/weekly-contest-236/problems/finding-mk-average/>  
讲道理，这题的分应该算我捡来的，因为这个题目设计的时候肯定是没想让我直接sort就能过的。。。  
```python
class MKAverage:

    def __init__(self, m: int, k: int):
        self.m = m
        self.k = k
        self.nums = []

    def addElement(self, num: int) -> None:
        self.nums.append(num)

    def calculateMKAverage(self) -> int:
        if len(self.nums) < self.m:
            return -1
        else:
            lastM = self.nums[-self.m:]
            a = sorted(lastM)[self.k:-self.k]
            return math.floor(sum(a)/len(a))
```
经过了一个多小时的纠结，还是没搞明白总的时间复杂度应该怎么算，直接贴一个答案在这儿好了。。  
<https://leetcode.com/problems/finding-mk-average/discuss/1152629/Python3-solution-w-SortedList-O(logM)-add-O(1)-calculate>  
不搞了，做CS229去了……