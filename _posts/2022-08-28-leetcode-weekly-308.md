---
title: "LeetCode Weekly Contest 308"
tags:
  - LeetCode
  - 算法
---

### 2389. Longest Subsequence With Limited Sum
题目中提到了是subsequence，所以可以直接排序然后计算
```python
class Solution:
    def answerQueries(self, nums: List[int], queries: List[int]) -> List[int]:
        if not queries:
            return []
        
        nums.sort()
        res = []
        for limit in queries:
            res.append(self.find(limit, nums))
        
        return res
    
    def find(self, limit, nums):
        count = 0
        
        curr_sum = 0
        for num in nums:
            if curr_sum + num > limit:
                break
            
            curr_sum += num
            count += 1
            
        return count
```

其实也可以在计算query的时候使用前缀和 + binary search，更快
```java
    public int[] answerQueries(int[] A, int[] queries) {
        Arrays.sort(A);
        int n = A.length, m = queries.length, res[] = new int[m];
        for (int i = 1; i < n; ++i)
            A[i] += A[i - 1];
        for (int i = 0; i < m; ++i) {
            int j = Arrays.binarySearch(A, queries[i]);
            res[i] = Math.abs(j + 1);
        }
        return res;
    }
```

### 2390. Removing Stars From a String
第一印象还想着要处理string，后来发现直接用stack即可
```python
class Solution:
    def removeStars(self, s: str) -> str:
        if not s:
            return s
        
        stack = []
        for ch in s:
            if ch != '*':
                stack.append(ch)
            else:
                stack.pop()
        
        return "".join(stack)
```

### 2391. Minimum Amount of Time to Collect Garbage
3种垃圾相互独立，可以分别计算，并且记录一下要算的travel距离
```python
class Solution:
    def splitGarbage(self, garbage):
        m_garbage = []
        p_garbage = []
        g_garbage = []
        
        for item in garbage:
            m_count = 0
            p_count = 0
            g_count = 0
            
            for ch in item:
                if ch == "M":
                    m_count += 1
                elif ch == "P":
                    p_count += 1
                elif ch == "G":
                    g_count += 1
                    
            m_garbage.append(m_count)
            p_garbage.append(p_count)
            g_garbage.append(g_count)
        
        return m_garbage, p_garbage, g_garbage
    
    def calcualteTime(self, garbage, travel):
        n = len(garbage)

        counter = Counter(garbage)
        if counter[0] == n:
            return 0
        
        clean_cost = 0
        travel_cost = 0
        for i in range(n):
            clean_cost += garbage[i]
            if i > 0:
                travel_cost += travel[i - 1]

            if i > 0 and garbage[i] > 0:
                clean_cost += travel_cost
                travel_cost = 0
        
        return clean_cost
            
    def garbageCollection(self, garbage: List[str], travel: List[int]) -> int:
        if not garbage:
            return 0
        
        m_garbage, p_garbage, g_garbage = self.splitGarbage(garbage)
        m_cost = self.calcualteTime(m_garbage, travel)
        p_cost = self.calcualteTime(p_garbage, travel)
        g_cost = self.calcualteTime(g_garbage, travel)
        
        return m_cost + p_cost + g_cost
```

但是后来发现这道题目更好的方法是记录每种垃圾的最后一个index，然后对travel使用前缀和
```java
public int garbageCollection(String[] gar, int[] travel) {
    int p=0, m=0, g=0, sum=0;

    for(int i=0; i<gar.length; i++){
        for(char ch : gar[i].toCharArray()){
            if(ch=='P') p = i;
            else if(ch=='M') m = i;
            else if(ch=='G') g = i;
            sum++;                         // add 1 min for every pick-up
        }
    }
    
    for(int i=1; i<travel.length; i++)
        travel[i] = travel[i]+travel[i-1];
    
    sum += p==0 ? 0 : travel[p-1];            // travel time till last P
    sum += m==0 ? 0 : travel[m-1];          // travel time till last M
    sum += g==0 ? 0 : travel[g-1];            // travel time till last G
    return sum;
}
```

### 2392. Build a Matrix With Conditions
非常惭愧这道题目没有AC，原因是想复杂了，一开始看到这道题直接联想起了N-Queens，于是先写了一个backtrack，试着提交一下发现超时。后来一直都在想着怎么样才能剪枝，继续优化backtrack，也想到了使用拓扑排序进行优化
```python
class Solution:
    def topoSort(self, conditions, k):
        graph = defaultdict(list)
        counter = Counter()
        
        for x, y in conditions:
            graph[x].append(y)
            counter[y] += 1
        
        res = []
        queue = deque()
        for num in range(1, k + 1):
            if counter[num] == 0:
                queue.append(num)
        
        while queue:
            node = queue.popleft()
            res.append(node)
            
            for adj_node in graph[node]:
                counter[adj_node] -= 1
                if counter[adj_node] == 0:
                    queue.append(adj_node)
        
        if len(res) < k:
            return []
        return res
        
    def buildMatrix(self, k: int, rowConditions: List[List[int]], colConditions: List[List[int]]) -> List[List[int]]:
        curr = [[0 for _ in range(k)] for _ in range(k)]
        
        row_sort = self.topoSort(rowConditions, k)
        col_sort = self.topoSort(colConditions, k)
        if not row_sort or not col_sort:
            return []
                
        rows = dict()
        cols = dict()
        self.dfs(0, 0, k, row_sort, 0, col_sort, 0, curr, 0, cols, colConditions)
        return curr
    
    def checkCols(self, cols, colConditions):
        for i in range(len(colConditions)):
            left, right = colConditions[i]
            if left not in cols or right not in cols:
                return True
            if cols[left] >= cols[right]:
                return False
        
        return True
    
    def dfs(self, i, j, k, row_sort, row_idx, col_sort, col_idx, curr, zero_count, cols, colConditions):
        if j == k:
            j = 0
            i += 1
        
        if i == k:
            if self.checkCols(cols, colConditions):
                return True
            return False
        
        if row_idx < len(row_sort) and row_sort[row_idx] not in cols:
            curr_num = row_sort[row_idx]
            cols[curr_num] = j
            curr[i][j] = row_sort[row_idx]
            if self.dfs(i, j + 1, k, row_sort, row_idx + 1, col_sort, col_idx, curr, zero_count, cols, colConditions):
                return True
            curr[i][j] = 0
            del cols[curr_num]
                
        if zero_count < k * k - k and self.dfs(i, j + 1, k, row_sort, row_idx, col_sort, col_idx, curr, zero_count + 1, cols, colConditions):
            return True
        
        return False
```

但最后还是超时了，其实这道题目想到拓扑排序已经可以了，关键是有了拓扑排序怎样构造后来的结果。对于行和列来说，数字排列的顺序是相互独立的，也就是说，在有了拓扑排序之后，直接分别按照下标的行和列的index放进去就可以了，根本不需要backtrack
```python
class Solution:
    def topoSort(self, conditions, k):
        graph = defaultdict(list)
        counter = Counter()
        
        for x, y in conditions:
            graph[x].append(y)
            counter[y] += 1
        
        res = []
        queue = deque()
        for num in range(1, k + 1):
            if counter[num] == 0:
                queue.append(num)
        
        while queue:
            node = queue.popleft()
            res.append(node)
            
            for adj_node in graph[node]:
                counter[adj_node] -= 1
                if counter[adj_node] == 0:
                    queue.append(adj_node)
        
        if len(res) < k:
            return []
        return res
        
    def buildMatrix(self, k: int, rowConditions: List[List[int]], colConditions: List[List[int]]) -> List[List[int]]:
        res = [[0 for _ in range(k)] for _ in range(k)]
        
        row_sort = self.topoSort(rowConditions, k)
        col_sort = self.topoSort(colConditions, k)
        if not row_sort or not col_sort:
            return []
                
        row_pos = dict()
        col_pos = dict()
        for i, num in enumerate(row_sort):
            row_pos[num] = i
        for i, num in enumerate(col_sort):
            col_pos[num] = i
        
        for i in range(1, k + 1):
            res[row_pos[i]][col_pos[i]] = i
        
        return res
```