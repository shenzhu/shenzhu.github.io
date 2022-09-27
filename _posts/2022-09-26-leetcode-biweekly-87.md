---
title: "LeetCode Biweekly Contest 87"
tags:
  - LeetCode
  - 算法
---

### 2409. Count Days Spent Together
这道题目在讨论区有很多人说stupid，自己一开始的实现也不是很优雅，不过它还是一道很好的考察区间相交的题目，区间相交有具体的公式
```
max(0, min(end1, end2) - max(start1, start2))
```
我们可以根据这个公式进行计算，另外对于每年两个日子相隔的计算，我们可以把它们转化成两个第某某天之间的相减。总的来说，这道题还是不错的
```python
class Solution:
    def countDaysTogether(self, arriveAlice: str, leaveAlice: str, arriveBob: str, leaveBob: str) -> int:
        return self.count(max(arriveAlice, arriveBob), min(leaveAlice, leaveBob))
    
    def count(self, start, end):
        days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
        
        month1, day1 = int(start[:2]), int(start[3:])
        month2, day2 = int(end[:2]), int(end[3:])
        
        start_day = sum(days[:month1 - 1]) + day1
        end_day = sum(days[:month2 - 1]) + day2
        
        if start_day > end_day:
            return 0
        
        return end_day - start_day + 1
```

### 2410. Maximum Matching of Players With Trainers
比较明显的贪心 + 双指针
```python
class Solution:
    def matchPlayersAndTrainers(self, players: List[int], trainers: List[int]) -> int:
        if not players or not trainers:
            return 0
        
        n, m = len(players), len(trainers)
        players.sort()
        trainers.sort()
        
        res = 0
        i, j = 0, 0
        while i < n and j < m:
            if players[i] <= trainers[j]:
                res += 1
                i += 1
                j += 1
            else:
                j += 1
        
        return res
```

### 2411. Smallest Subarrays With Maximum Bitwise OR
OR的性质是越OR越大，所以我们可以从后向前计算，得到从每个index开始能得到的最大OR value，然后再从前向后判断到哪个位置可以达到最大OR value，这时一开始的想法，时间复杂度为`O(n^2)`，当然是超时了...
```python
class Solution:
    def smallestSubarrays(self, nums: List[int]) -> List[int]:
        if not nums:
            return []
        
        n = len(nums)

        max_vals = [0 for _ in range(n)]
        max_vals[-1] = nums[-1]
        
        val = nums[-1]
        for i in range(n - 2, -1, -1):
            max_vals[i] = max_vals[i + 1] | nums[i]
        
        res = []
        for i in range(n):
            j = i
            curr_val = nums[i]
            while curr_val != max_vals[i] and j < n:
                curr_val = curr_val | nums[j]
                j += 1
            
            res.append(max(j - i, 1))
        
        return res
```
正确的做法是从后向前按位计算，我们采用一个长度为32的数组，记录每个1出现的最后index，对于每个starting index，它需要的最短数组是所有1最后index的最大值，因此根据这个思路我们可以写代码
```python
class Solution:
    def smallestSubarrays(self, nums: List[int]) -> List[int]:
        n = len(nums)
        
        last_digits = [n for _ in range(32)]
        res = [0 for _ in range(n)]

        for i in range(n - 1, -1, -1):
            k = i
            for j in range(32):
                curr_digit = (nums[i] >> j) & 1
                if curr_digit == 1:
                    last_digits[j] = i
                elif last_digits[j] < n:
                    k = max(k, last_digits[j])
            res[i] = k - i + 1
        
        
        return res
```
注意其中有一个`last_digits[j] < n`的判断，也就是说对于没有遇到过的1的位置我们可以不用管它

### 2412. Minimum Money Required Before Transactions
这道题目的思路也很难想，意义不算太大，本质上这道题目是计算下面的公式
```
money - (a^0 - b^0) - (a^1 - b^1) - ... >= a^i
                                  money >= (a^0 - b^0) +  (a^1 - b^1) + ... + a^i
```
题目中的要求是对于任意的顺序，所以我们要求的是`(a^0 - b^0) +  (a^1 - b^1) + ...`和`a^i`的最大值，后者我们可以进行枚举，而前者的最大值即为所有a > b的输入
```python
class Solution:
    def minimumMoney(self, transactions: List[List[int]]) -> int:
        pos_sum = 0
        for a, b in transactions:
            pos_sum += max(0, a - b)
        
        res = 0
        for a, b in transactions:
            res = max(res, pos_sum - max(0, (a - b)) + a)
        
        return res
```