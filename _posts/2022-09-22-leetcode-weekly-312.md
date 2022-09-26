---
title: "LeetCode Weekly Contest 312"
tags:
  - LeetCode
  - 算法
---

### 2418. Sort the People
签到题，不多说
```python
class Solution:
    def sortPeople(self, names: List[str], heights: List[int]) -> List[str]:
        if not names or not heights:
            return []
        
        values = [(name, height) for name, height in zip(names, heights)]
        values.sort(key=lambda v:v[1], reverse=True)
        
        return [item[0] for item in values]
```

### 2419. Longest Subarray With Maximum Bitwise AND
这道题还是需要想一下的，如果我们已经知道了整个数组中的最大数字，其他任何数字与它AND都会变小，所以数组中AND的最大值就是数组中的最大数字

因此，接下来我们要找的就是数组中最长的连续最大值
```python
class Solution:
    def longestSubarray(self, nums: List[int]) -> int:
        max_val = max(nums)
        
        res = 0
        count = 0
        
        for num in nums:
            if num == max_val:
                count += 1
                res = max(res, count)
            else:
                count = 0
        
        return res
```

### 2420. Find All Good Indices
这道题目一开始想的非常复杂，我们需要知道某个index之前和之后的non-increasing和non-decreasing数组的长度是否大于某个值，因此当时脑子里想到的第一个解法是使用单调队列，每次移动的时候把单调队列中不符合要求的元素移除，之后检查单调队列的长度是否大于目标值
```python
class Solution:
    def goodIndices(self, nums: List[int], k: int) -> List[int]:
        """
        [2,1,1,1,3,4,1]
         0,1,2,3,4,5,6
         
         
        [878724,201541,179099,98437,35765,327555,475851,598885,849470,943442]
              0,     1,     2,    3,    4,     5,     6,     7,     8,     9
        """
        before = deque()
        after = deque()
        
        n = len(nums)
        for i in range(k):
            while before and nums[before[-1]] < nums[i]:
                before.pop()
            before.append(i)
        
        for i in range(k + 1, min(n, k + k + 1)):
            while after and nums[after[-1]] > nums[i]:
                after.pop()
            after.append(i)
        
        res = []
        if len(before) >= k and len(after) >= k:
            res.append(k)
            
        # print(k)
        # print(before)
        # print(after)
        
        for i in range(k + 1, n - k):
            while before and before[0] < i - k:
                before.popleft()
            
            while before and nums[before[-1]] < nums[i - 1]:
                before.pop()
            before.append(i - 1)
            
            while after and after[0] <= i:
                after.popleft()
            
            while after and nums[after[-1]] > nums[i + k]:
                after.pop()
            after.append(i + k)
            
            # print(i)
            # print(before)
            # print(after)
            
            if len(before) >= k and len(after) >= k:
                res.append(i)
        
        return res
```

实在是太复杂了，比赛的时候错了两次，之后参考别人的解法，最好的方法是对数组进行预处理
```python
class Solution:
    def goodIndices(self, nums: List[int], k: int) -> List[int]:
        n = len(nums)
        
        prefix = [1 for _ in range(n)]
        suffix = [1 for _ in range(n)]
        
        for i in range(1, n):
            if nums[i - 1] >= nums[i]:
                prefix[i] = prefix[i - 1] + 1
        
        for i in range(n - 2, -1, -1):
            if nums[i] <= nums[i + 1]:
                suffix[i] = suffix[i + 1] + 1
        
        res = []
        for i in range(k, n - k):
            if prefix[i - 1] >= k and suffix[i + 1] >= k:
                res.append(i)
    
        return res
```

### 2421. Number of Good Paths
比赛的时候写了一个暴力版本，不出意外地超时了
```python
class Solution:
    def numberOfGoodPaths(self, vals: List[int], edges: List[List[int]]) -> int:
        tree = defaultdict(list)
        for x, y in edges:
            tree[x].append(y)
            tree[y].append(x)
                    
        paths = set()
        n = len(vals)
        
        for i in range(n):
            visited = set()
            visited.add(i)
            self.dfs(i, tree, vals, paths, [i], visited)
        
        return len(paths) + n
    
    def dfs(self, node, tree, vals, paths, curr, visited):
        if len(curr) > 1 and vals[curr[0]] == vals[node]:
            sorted_nodes = sorted(curr)
            curr_path = ",".join(map(str, sorted_nodes))
            paths.add(curr_path)
        
        for adj_node in tree[node]:
            if vals[adj_node] <= vals[curr[0]] and adj_node not in visited:
                # print("selected " + str(adj_node))
                curr.append(adj_node)
                visited.add(adj_node)
                self.dfs(adj_node, tree, vals, paths, curr, visited)
                visited.discard(adj_node)
                curr.pop()
```

后来看了别人的解法，感觉确实很强，自己怎么想不到呢...首先看题目的条件 `All nodes between the starting node and the ending node have values less than or equal to the starting node`，这就引起我们思考能不能先对所有的点按照值从小到大排序，然后依次处理，那么我们怎么判断连通性呢？自然是想到了union-find

所以总体地算法就是，按照值从小到大的顺序处理节点，每次把所有相同值的节点找到，根据连通性把它们分成几组(因为不能保证这些相同值的节点目前完全连通)，在每个组内这些点可以互为头尾，比如某个组内有3个节点，可以组成的path就是3 * 2 / 2 + 3 = 6
```python
class Solution:
    def find(self, roots, x):
        if roots[x] != x:
            roots[x] = self.find(roots, roots[x])
        return roots[x]

    def numberOfGoodPaths(self, vals: List[int], edges: List[List[int]]) -> int:
        n = len(vals)

        tree = defaultdict(list)
        for x, y in edges:
            tree[x].append(y)
            tree[y].append(x)
        
        nodes = [(idx, val) for idx, val in enumerate(vals)]
        nodes.sort(key=lambda n: n[1])
                
        roots = [i for i in range(n)]
        
        res = 0
        
        i = 0
        while i < n:
            j = i
            while j < n and nodes[j][1] == nodes[i][1]:
                j += 1
                
            for k in range(i, j):
                node = nodes[k][0]
                curr_root = self.find(roots, node)
                for adj_node in tree[node]:
                    if vals[adj_node] <= vals[node]:
                        roots[self.find(roots, adj_node)] = curr_root
            
            group_count = Counter()
            for k in range(i, j):
                node = nodes[k][0]
                group_count[self.find(roots, node)] += 1
                        
            for count in group_count.values():
                res += (count - 1) * count // 2
                res += count
            
            
            i = j
        
        return res
```