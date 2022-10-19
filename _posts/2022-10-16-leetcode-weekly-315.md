---
title: "LeetCode Weekly Contest 313-315"
tags:
  - LeetCode
  - 算法
---

## Weekly 313

### 2427. Number of Common Factors
```
Given two positive integers a and b, return the number of common factors of a and b.

An integer x is a common factor of a and b if x divides both a and b.
```

签到题
```python
class Solution:
    def commonFactors(self, a: int, b: int) -> int:
        min_val = min(a, b)
        
        count = 0
        for i in range(1, min_val + 1):
            if a % i == 0 and b % i == 0:
                count += 1
        
        return count
```

### 2428. Maximum Sum of an Hourglass

迭代可能的左上角并且计算
```python
class Solution:
    def calculate(self, i, j, grid):
        res = 0
        for k in range(3):
            res += grid[i][j + k]
            res += grid[i + 2][j + k]
        
        res += grid[i + 1][j + 1]
        return res
    
        
    def maxSum(self, grid: List[List[int]]) -> int:
        n, m = len(grid), len(grid[0])
        
        res = 0
        for i in range(n - 2):
            for j in range(m - 2):
                res = max(res, self.calculate(i, j, grid))
        
        return res
```

### 2429. Minimize XOR
```
Given two positive integers num1 and num2, find the positive integer x such that:

x has the same number of set bits as num2, and
The value x XOR num1 is minimal.
Note that XOR is the bitwise XOR operation.

Return the integer x. The test cases are generated such that x is uniquely determined.

The number of set bits of an integer is the number of 1s in its binary representation.
```

首先计算出num2中的1个数，然后思考怎样与num1的值才能是最小的，大概有三种情况
- 如果与num1中1的个数一样，那么答案直接就是0
- 如果小于num1中1的个数，那么把num1的高位改为0
- 如果大于num1中1的个数，那么从低位向高位将结果的0设置成1

```python
class Solution:
    def minimizeXor(self, num1: int, num2: int) -> int:
        set_bits = 0
        for i in range(64):
            set_bits += (num2 >> i) & 1
        
        temp = num1
        res = 0
        for i in range(63, -1, -1):
            if (temp >> i) & 1 == 1 and set_bits > 0:
                set_bits -= 1
                res |= (1 << i)
                
                if set_bits == 0:
                    break
        
        for i in range(64):
            if (res >> i) & 1 == 0 and set_bits > 0:
                set_bits -= 1
                res |= (1 << i)
                
                if set_bits == 0:
                    break
        
        return res
```


### 2430. Maximum Deletions on a String
```
You are given a string s consisting of only lowercase English letters. In one operation, you can:

- Delete the entire string s, or
- Delete the first i letters of s if the first i letters of s are equal to the following i letters in s, for any i in the range 1 <= i <= s.length / 2.
For example, if s = "ababc", then in one operation, you could delete the first two letters of s to get "abc", since the first two letters of s and the following two letters of s are both equal to "ab".

Return the maximum number of operations needed to delete all of s.
```

可以首先计算出lcs(longest common substring), lcs[i][j] = k表示s.substring(i, i + k) = s.substring(j, j + k)

之后可以可以使用dp[i]表示从i开始最多的删除次数，那么每当我们发现lcs[i][j] >= j - i就有dp[i] = max(dp[i], dp[j] + 1)

```python
class Solution:
    def deleteString(self, s: str) -> int:
        n = len(s)
        
        lcs = [[0 for _ in range(n + 1)] for _ in range(n + 1)]
        for i in range(n - 1, -1, -1):
            for j in range(n - 1, i - 1, -1):
                if s[i] == s[j]:
                    lcs[i][j] = 1 + lcs[i + 1][j + 1]
        
        dp = [0 for _ in range(n + 1)]
        for i in range(n - 1, -1, -1):
            for j in range(i, n):
                if lcs[i][j] >= j - i:
                    dp[i] = max(dp[i], dp[j] + 1)
        
        return dp[0]
```

为了快速判断字符串的两个substring是否相等，也可以使用字符串哈希
```python
class Solution:
    def get(self, l, r, f, p):
        """
        H(S + c) = H(S) * P + value[c]
        """
        return f[r] - f[l - 1] * p[r - l + 1]
        
    def deleteString(self, s: str) -> int:
        n = len(s)
        
        P = 131
        
        f = [0] * (2 * n)
        p = [0] * (2 * n)
        
        p[0] = 1
        for i in range(1, n + 1):
            f[i] = (f[i - 1] * P + ord(s[i - 1]))
            p[i] = (p[i - 1] * P)
        
        dp = [0 for _ in range(n + 1)]
        for i in range(n - 1, -1, -1):
            for j in range(i, n):
                if self.get(i + 1, j, f, p) == self.get(j + 1, j + j - i, f, p):
                    dp[i] = max(dp[i], dp[j] + 1)
            
        return dp[0]
```

## Weekly 314

### 2432. The Employee That Worked on the Longest Task

```
There are n employees, each with a unique id from 0 to n - 1.

You are given a 2D integer array logs where logs[i] = [idi, leaveTimei] where:

- idi is the id of the employee that worked on the ith task, and
- leaveTimei is the time at which the employee finished the ith task. All the values leaveTimei are unique.

Note that the ith task starts the moment right after the (i - 1)th task ends, and the 0th task starts at time 0.

Return the id of the employee that worked the task with the longest time. If there is a tie between two or more employees, return the smallest id among them.
```

虽然题目比较长，但是逻辑简单，算是签到题

```python
class Solution:
    def hardestWorker(self, n: int, logs: List[List[int]]) -> int:
        counter = Counter()
        
        for i in range(len(logs)):
            user_id, leave_time = logs[i]
            if i == 0:
                counter[user_id] = max(counter[user_id], leave_time)
            else:
                counter[user_id] = max(counter[user_id], leave_time - logs[i - 1][1])
        
        res = -1
        max_time = 0
                
        for key, val in counter.items():
            if val > max_time or (val == max_time and key < res):
                res = key
                max_time = val
        
        return res
```

### 2433. Find The Original Array of Prefix Xor
```
You are given an integer array pref of size n. Find and return the array arr of size n that satisfies:

- pref[i] = arr[0] ^ arr[1] ^ ... ^ arr[i].
Note that ^ denotes the bitwise-xor operation.

It can be proven that the answer is unique.
```

这道题目给出了前缀和，让我们求原数组，可以使用差分进行计算
```python
class Solution:
    def findArray(self, pref: List[int]) -> List[int]:
        n = len(pref)
        
        res = [0 for _ in range(n)]
        res[0] = pref[0]
        
        for i in range(1, n):
            res[i] = pref[i] ^ pref[i - 1]
        
        return res
```

### 

### 2434. Using a Robot to Print the Lexicographically Smallest String
```
You are given a string s and a robot that currently holds an empty string t. Apply one of the following operations until s and t are both empty:

- Remove the first character of a string s and give it to the robot. The robot will append this character to the string t.
- Remove the last character of a string t and give it to the robot. The robot will write this character on paper.

Return the lexicographically smallest string that can be written on the paper.
```

这道题目出得非常好，首先经过观察我们可以注意到t的作用其实相当于一个栈，那么这道题目的本质就是把给定string中的character逐渐加入栈中，同时可以在任意时刻出栈，在这种情况下构造一个字典序最小的字符串

由上面的分析可知，重点就是什么时候出栈，一开始可能会想到，在某个char即将入栈的时候，把栈里所有的小于它的char都弹出，但这种方法是错的. 参考给出的例子"bdda"，如果我们按照上面方法进行，那么结果就是"badd"，其实答案应该是"addb". 所以上面我们讨论的出栈条件是错误的，正确的出栈条件是，把小于等于还没加入栈中的最小元素的栈顶弹出，代码如下
```python
class Solution:
    def findMinChar(self, counter):
        for i in range(26):
            ch = chr(ord('a') + i)
            if counter[ch] > 0:
                return ch
        
        return 'a'

    def robotWithString(self, s: str) -> str:
        stack = []
        res = []
        
        counter = Counter(s)
        for ch in s:
            while stack and stack[-1] <= self.findMinChar(counter):
                res.append(stack.pop())
            stack.append(ch)
            counter[ch] -= 1
        
        while stack:
            res.append(stack.pop())
        
        return "".join(res)
```

### 2435. Paths in Matrix Whose Sum Is Divisible by K
```
You are given a 0-indexed m x n integer matrix grid and an integer k. You are currently at position (0, 0) and you want to reach position (m - 1, n - 1) moving only down or right.

Return the number of paths where the sum of the elements on the path is divisible by k. Since the answer may be very large, return it modulo 109 + 7.
```

看到题目中需要数数，大概率想到要用DP来做，另外使用第三维度记录余数，非常巧妙
```python
class Solution:
    def numberOfPaths(self, grid: List[List[int]], k: int) -> int:
        n, m = len(grid), len(grid[0])
        
        MOD = pow(10, 9) + 7
        
        dp = [
            [[0 for _ in range(k)] for _ in range(m + 1)]
            for _ in range(n + 1)
        ]
        dp[1][1][grid[0][0] % k] = 1
        
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                if i == 1 and j == 1:
                    continue
                
                for r in range(k):
                    dp[i][j][(r + grid[i - 1][j - 1]) % k] = (dp[i - 1][j][r] + dp[i][j - 1][r]) % MOD
        
        return dp[n][m][0]
```

## Weekly 315

### 2441. Largest Positive Integer That Exists With Its Negative

```
Given an integer array nums that does not contain any zeros, find the largest positive integer k such that -k also exists in the array.

Return the positive integer k. If there is no such integer, return -1.
```

签到题目，不多说
```python
class Solution:
    def findMaxK(self, nums: List[int]) -> int:
        if not nums:
            return -1

        counter = Counter(nums)
        
        res = float("-inf")
        for num in nums:
            if num > res and counter[-num] > 0:
                res = num
        
        if res == float("-inf"):
            return -1
        
        return res
```

### 2442. Count Number of Distinct Integers After Reverse Operations

```
You are given an array nums consisting of positive integers.

You have to take each integer in the array, reverse its digits, and add it to the end of the array. You should apply this operation to the original integers in nums.

Return the number of distinct integers in the final array.
```

签到题目，不多说
```python
class Solution:
    def countDistinctIntegers(self, nums: List[int]) -> int:
        uniques = set()
        for num in nums:
            num_str = str(num)
            uniques.add(num)
            # use int to account for starting with 0
            uniques.add(int(num_str[::-1]))
        
        return len(uniques)
```

### 2443. Sum of Number and Its Reverse

```
Given a non-negative integer num, return true if num can be expressed as the sum of any non-negative integer and its reverse, or false otherwise.
```
一开始考虑是不是有一些巧妙的算法，结果后来发现是暴力...
```python
class Solution:
    def sumOfNumberAndReverse(self, num: int) -> bool:
        if num == 0:
            return True

        for i in range(num):
            num1 = str(i)
            num2 = num1[::-1]
            
            if int(num1) + int(num2) == num:
                return True
        
        return False
```

### 2444. Count Subarrays With Fixed Bounds

```
You are given an integer array nums and two integers minK and maxK.

A fixed-bound subarray of nums is a subarray that satisfies the following conditions:

The minimum value in the subarray is equal to minK.
The maximum value in the subarray is equal to maxK.
Return the number of fixed-bound subarrays.

A subarray is a contiguous part of an array.
```

一开始想的是sliding window，但是不太好做，后来看了别人的讲解发现很巧妙。首先我们思考，对于目标subarray，所有元素的值必须在minK，maxK之间，因此可以根据不在minK和maxK之间的数据把整个数组分成不同的部分

在每一个部分中使用双指针算法，右侧指针不断向前运动，左侧维护两个指针minK和maxK的index，那么从这段数组开始到min(minK, maxK)的数据都可以作为满足条件数组的左侧端点
```python
class Solution:
    def countSubarrays(self, nums: List[int], minK: int, maxK: int) -> int:
        if not nums:
            return 0
        
        last = -1
        min_idx = -1
        max_idx = -1
        
        count = 0
        for i in range(len(nums)):
            if nums[i] < minK or nums[i] > maxK:
                last = i
                continue
            
            if nums[i] == minK:
                min_idx = i
            
            if nums[i] == maxK:
                max_idx = i
            
            if min(min_idx, max_idx) > last:
                count += min(min_idx, max_idx) - last
        
        return count
```