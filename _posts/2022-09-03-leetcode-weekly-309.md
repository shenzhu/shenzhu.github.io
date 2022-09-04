---
title: "LeetCode Weekly Contest 309"
tags:
  - LeetCode
  - 算法
---

### 2399. Check Distances Between Same Letters
算是签到题，但是比较悲剧的是看错了题，直接返回了计算出的结果数组，其实要看的是结果和给定的数组是否一致
```python
class Solution:
    def checkDistances(self, s: str, distance: List[int]) -> bool:        
        indexes = dict()
        for i, ch in enumerate(s):
            if ch in indexes:
                if distance[ord(ch) - ord('a')] != i - indexes[ch] - 1:
                    return False
            else:
                indexes[ch] = i
        
        return True
```

### 2400. Number of Ways to Reach a Position After Exactly k Steps
看到要求结果数量和对结果取余，第一反应是用DP，写了一个二维数组dp[k][i]表示第k步在位置i的结果数量，结果超时了
```python
class Solution:
    def numberOfWays(self, startPos: int, endPos: int, k: int) -> int:
        if startPos + k < endPos:
            return 0
        
        dist = endPos - startPos
        if (k - dist) % 2 != 0:
            return 0
        
        MOD = pow(10, 9) + 7

        dp = defaultdict(Counter)
        dp[0][startPos] = 1
        
        MOD = pow(10, 9) + 7
        
        for i in range(1, k + 1):
            for j in range(startPos - k, startPos + k + 1):
                left =  dp[i - 1][j - 1]
                right = dp[i - 1][j + 1]
                
                dp[i][j] = (dp[i][j] + left + right) % MOD
        
        return dp[k][endPos] % MOD
```
后来又写了一个BFS，但其实比赛完试了下，DP是可以过的，应该是比赛的时候出了点问题
```python
class Solution:
    def numberOfWays(self, startPos: int, endPos: int, k: int) -> int:
        if startPos + k < endPos:
            return 0
        
        dist = endPos - startPos
        if (k - dist) % 2 != 0:
            return 0
        
        MOD = pow(10, 9) + 7
        
        queue = deque()
        queue.append((startPos, 1))
        
        step = 0
        while queue and step < k:
            size = len(queue)
            next_level = Counter()
            for _ in range(size):
                pos, ways = queue.popleft()
                
                next_level[pos - 1] += ways
                next_level[pos + 1] += ways
            
            for pos, ways in next_level.items():
                queue.append((pos, ways))
            
            step += 1
        
        final_pos = dict()
        for pos, ways in queue:
            final_pos[pos] = ways
        
        if endPos not in final_pos:
            return 0
        
        return final_pos[endPos] % MOD
```
比赛完看了下discussion，这道题目用top-down DP + cache可能写起来会更方便，初始可以用一个二维数组把所有值都设为-1表示不可达

### 2401. Longest Nice Subarray
比赛的时候写了一个sliding window
```python
class Solution:
    def check(self, bits):
        for count in bits:
            if count > 1:
                return False
        
        return True

    def longestNiceSubarray(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        bits = [0 for _ in range(64)]
        n = len(nums)
        
        res = 0
        i, j = 0, 0
        while j < n:
            num = nums[j]
            
            for k in range(63):
                bits[k] += (num >> k) & 1
            
            while i < j and not self.check(bits):
                prev_num = nums[i]
                for k in range(63):
                    bits[k] -= (prev_num >> k) & 1
                i += 1
            
            res = max(res, j - i + 1)
            j += 1
        
        return res
```
其实可以优化一下，去掉`check`函数，使用一个变量表示大于1的位数，然后在内部的while循环中不断增加i直到这个变量位0，或者使用位运算，但不是很好想(主要是自己不熟...)
```python
class Solution:
    def longestNiceSubarray(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        n = len(nums)
        
        res = 0
        i, j = 0, 0
        state = 0
        while j < n:
            while i < j and (state & nums[j]) > 0:
                state ^= nums[i]
                i += 1
            state |= nums[j]
            
            res = max(res, j - i + 1)
            j += 1
        
        return res
```

### 2402. Meeting Rooms III
这道题目一开始写的版本使用了两个heap，一个heap维护正在开的会，另一个heap维护可以使用的room，但写了个版本结果过了`80/81`...在比赛的时候没有AC，比赛完后仔细看了看，把代码好好写了一遍，就过了...可能是比赛的时候比较紧张某个地方打错了
```python
class Solution:
    def mostBooked(self, n: int, meetings: List[List[int]]) -> int:
        if not meetings:
            return 0
        
        meetings.sort(key=lambda m: m[0])

        free_rooms = [i for i in range(n)]
        heapify(free_rooms)
        curr_meetings = []
        
        counter = Counter()
        for start, end in meetings:
            while len(curr_meetings) > 0 and curr_meetings[0][0] <= start:
                _, prev_room = heappop(curr_meetings)
                heappush(free_rooms, prev_room)
            
            if len(free_rooms) > 0:
                curr_room = heappop(free_rooms)
                counter[curr_room] += 1
                heappush(curr_meetings, (end, curr_room))
            else:
                prev_end, prev_room = heappop(curr_meetings)
                counter[prev_room] += 1
                heappush(curr_meetings, (end + prev_end - start, prev_room))
        
        max_count = 0
        res = 0
        for room, count in counter.items():
            if (count > max_count) or (count == max_count and room < res):
                max_count = count
                res = room
        
        return res
```