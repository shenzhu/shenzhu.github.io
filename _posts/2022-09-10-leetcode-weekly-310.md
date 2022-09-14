---
title: "LeetCode Weekly Contest 310"
tags:
  - LeetCode
  - 算法
---

### 2404. Most Frequent Even Element
签到题，直接用dict做
```python3
class Solution:
    def mostFrequentEven(self, nums: List[int]) -> int:
        if not nums:
            return -1
        
        counter = Counter()
        max_num, max_count = -1, 0
        for num in nums:
            if num % 2 == 0:
                counter[num] += 1
                if (counter[num] > max_count) or (counter[num] == max_count and max_num > num):
                    max_num = num
                    max_count = counter[num]
        
        return max_num
```

### 2405. Optimal Partition of String
同样用dict，每次遇到重复的character就重新计算
```python3
class Solution:
    def partitionString(self, s: str) -> int:
        if not s:
            return 0
        
        counter = Counter()
        res = 1
        
        for ch in s:
            if counter[ch] > 0:
                counter = Counter()
                res += 1
            counter[ch] += 1
        
        return res
```

### 2406. Divide Intervals Into Minimum Number of Groups
和meeting rooms很像，使用heap按照group的end升序排列，当想要加入新的interval时，先把不冲突的全拿出来
```python3
class Solution:
    def minGroups(self, intervals: List[List[int]]) -> int:
        if not intervals:
            return 0
        
        intervals.sort(key=lambda item: item[0])
        
        heap = []
        res = 0
        for left, right in intervals:
            while heap and heap[0][0] < left:
                heappop(heap)
            
            heappush(heap, (right, left))
            res = max(res, len(heap))
        
        return res
```

### 2407. Longest Increasing Subsequence II
一开始用了DP，但是超时了，比赛完之后才发现要用线段树，即使是有线段树的想法，这道题的思路也不是很容易，可以参考[这个链接](https://leetcode.com/problems/longest-increasing-subsequence-ii/discuss/2560085/Python-Explanation-with-pictures-Segment-Tree)。另外这道题目的线段树模板要背下来，核心是`update`和`query`函数，可以对它们变换来处理max, min, sum的情况，要注意的是线段树从1开始，所以这道题目对于1要做一下特殊处理
```python3
class Solution:
    def update(self, segs, p, l, r, idx, val):
        if l == r:
            segs[p] = val
            return
        
        mid = (l + r) // 2
        if idx <= mid:
            self.update(segs, p * 2, l, mid, idx, val)
        else:
            self.update(segs, p * 2 + 1, mid + 1, r, idx, val)
        segs[p] = max(segs[p * 2], segs[p * 2 + 1])
    
    def query(self, segs, p, l, r, L, R):
        if L <= l and r <= R:
            return segs[p]
        
        res = 0
        mid = (l + r) // 2
        if L <= mid:
            res = self.query(segs, p * 2, l, mid, L, R)
        if R > mid:
            res = max(res, self.query(segs, p * 2 + 1, mid + 1, r, L, R))
        
        return res
        
    def lengthOfLIS(self, nums: List[int], k: int) -> int:
        if not nums:
            return 0
        
        mx = max(nums)
        segs = [0 for _ in range(4 * mx)]
        
        res = 1
        for num in nums:
            if num == 1:
                self.update(segs, 1, 1, mx, 1, 1)
            else:
                prev_max = self.query(segs, 1, 1, mx, max(num - k, 1), num - 1)
                res = max(res, prev_max + 1)
                self.update(segs, 1, 1, mx, num, prev_max + 1)
        
        return res
```