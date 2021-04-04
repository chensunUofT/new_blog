---
layout: post
title: LeetCode周赛#235
date: 2021-04-04
categories: [technology]
tags: [LeetCode, algorithm, coding]
comments: true
toc: true
excerpt: 咋还不如上回了呢
---

## 第一题
<https://leetcode.com/contest/weekly-contest-235/problems/truncate-sentence/>  
取前k个词，用空格join起来  
```python
class Solution:
    def truncateSentence(self, s: str, k: int) -> str:
        return ' '.join(s.split(" ")[:k])
```
用时01'09"，贼得意。

## 第二题
<https://leetcode.com/contest/weekly-contest-235/problems/finding-the-users-active-minutes/>  
感觉像是一道数据库/SQL的考题……先数出每个ID的UAM，然后构造所求数组  
```python
class Solution:
    def findingUsersActiveMinutes(self, logs: List[List[int]], k: int) -> List[int]:
        user_time = defaultdict(set)
        for uid, time in logs:
            user_time[uid].add(time)
        ans = [0] * k
        for uid in user_time:
            ans[len(user_time[uid]) - 1] += 1
        return ans
```

## 第三题
<https://leetcode.com/contest/weekly-contest-235/problems/minimum-absolute-sum-difference/>  
先是十分头铁地用O(n^2)的方法做了一通，果不其然TLE了，于是开始寻求时间优化的方法。Google了一下怎么在一个数组中找到与target最接近的数，果然要用到[bisect](https://docs.python.org/3/library/bisect.html#module-bisect)，不过区别在于，对于`bisect_left`返回的下标`i`（它满足：`all(val < x for val in a[lo:i]) for the left side and all(val >= x for val in a[i:hi]) for the right side.`），要比较`i`和`i-1`（如果不越界）哪个离target更接近。  
我想到的另一个优化（不知道为什么比赛的时候结果不太对……）是，既然已经用O(NlogN)的时间把`nums1`排序了，那不妨把每一对数字的绝对值差也降序排序，然后从绝对值差最大的那组数开始试着替换：如果替换数字的improvement（替换后的绝对值差-替换前的绝对值差）大于等于排序后下一个绝对值差，那么后边的替换便没有必要再试了，可以early stopping了。  
```python
class Solution:
    def minAbsoluteSumDiff(self, nums1: List[int], nums2: List[int]) -> int:
        
        N = len(nums1)
        M = 10**9 + 7
        diffs = [(i, abs(nums1[i] - nums2[i])) for i in range(N)]
        original_asd = sum([diffs[i][1] for i in range(N)])

        if original_asd == 0:
            return 0
        if len(set(nums1)) == 1:
            return original_asd % M
        
        max_imp = 0
        nums1_sorted = sorted(nums1)
        diffs_sorted = sorted(diffs, key = lambda x: -x[1])

        for i, diff in diffs_sorted:
            idx = bisect_left(nums1_sorted, nums2[i])
            if idx < N:
                max_imp = max(max_imp, diff - (nums1_sorted[idx] - nums2[i]))
            if idx > 0:
                max_imp = max(max_imp, diff - (nums2[i] - nums1_sorted[idx-1]))
            if i+1 < N and max_imp >= diffs_sorted[i+1][1]:
                break
        return (original_asd - max_imp) % M
```

## 第四题
<https://leetcode.com/contest/weekly-contest-235/problems/number-of-different-subsequences-gcds/>  
比赛的时候真是没有头绪，疯狂超时……  
正确思路如下：因为所有数字都小于最大值N=2\*10^5，所以我们只需要知道对于所有`i=1,2,...,N`，`i`是不是`nums`的某个子集的GCD。如果是，那么这个子集中的所有数必然是`i`的倍数`j`（否则`i`一定不是它的因数）。因此，我们只需要求出`nums`中所有`i`的倍数的GCD。显然，对一系列数求GCD只可能越求越小，而`i`的倍数的GCD最小又只能是`i`，因此如果对`j`的迭代过程中GCD已然为`i`，便可停止。若迭代结束时GCD的值与`i`相等，则表明`i`是`nums`某个子集的GCD，答案计数器加一。  
```python
class Solution:
    def countDifferentSubsequenceGCDs(self, nums: List[int]) -> int:
        N = max(nums)
        existed = [False] * (N + 1)
        ans = 0
        for num in nums:
            existed[num] = True
        for i in range(1, N+1):
            g = 0
            for j in range(i, N+1, i):
                if existed[j]:
                    g = math.gcd(g, j)
                    if g == i:
                        break
            ans += (g==i)
        return ans
```