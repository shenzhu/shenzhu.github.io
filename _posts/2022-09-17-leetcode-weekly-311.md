---
title: "LeetCode Weekly Contest 311"
tags:
  - LeetCode
  - 算法
---

### 2413. Smallest Even Multiple
签到题
```python
class Solution:
    def smallestEvenMultiple(self, n: int) -> int:
        for i in range(2, 2 * n + 1):
            if i % 2 == 0 and i % n == 0:
                return i
        
        return 2 * n
```

### Length of the Longest Alphabetical Continuous Substring
每次遇到不连续的就换一个
```python
class Solution:
    def longestContinuousSubstring(self, s: str) -> int:
        if not s:
            return 0
        
        res = 1
        
        curr = 1
        for i in range(1, len(s)):
            if ord(s[i]) - ord(s[i - 1]) == 1:
                curr += 1
            else:
                res = max(res, curr)
                curr = 1
        
        res = max(res, curr)
        
        return res
```

### 2415. Reverse Odd Levels of Binary Tree
一开始的想法是用BFS把所有的node value都找出来，然后从这些value中构建完美二叉树
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, values, index):
        node = TreeNode(values[index])
        
        if 2 * index + 2 < len(values):
            node.left = self.buildTree(values, index * 2 + 1)
            node.right = self.buildTree(values, index * 2 + 2)
        
        return node
        
    def reverseOddLevels(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root:
            return root
        
        values = []
        queue = deque()
        queue.append(root)
        level = 0
        
        while queue:
            curr = []
            size = len(queue)
            for _ in range(size):
                node = queue.popleft()
                curr.append(node.val)
                
                if node.left and node.right:
                    queue.append(node.left)
                    queue.append(node.right)
            
            if level % 2 != 0:
                curr.reverse()
            values.extend(curr)
            level += 1
        
        return self.buildTree(values, 0)
```
比赛完看了一下使用DFS更加方便，在recursion的过程中交换value
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def reverseOddLevels(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        self.reverse(root.left, root.right, 1)
        return root
    
    def reverse(self, node1, node2, level):
        if not node1 or not node2:
            return
        
        if level % 2 != 0:
            node1.val, node2.val = node2.val, node1.val
        
        self.reverse(node1.left, node2.right, level + 1)
        self.reverse(node1.right, node2.left, level + 1)
```

### 2416. Sum of Prefix Scores of Strings
一开始想复杂了，还以为要在Trie node中记录是否为word以及word count，但其实并不需要，直接每个Trie node的count + 1就行
```python
class TrieNode:
    def __init__(self, ch):
        self.ch = ch
        self.count = 0
        self.word = ""
        self.children = dict()

class Solution:
    def buildTrie(self, words):
        root = TrieNode('')
        
        for word in words:
            curr = root
            for ch in word:
                if ch not in curr.children:
                    curr.children[ch] = TrieNode(ch)
                curr = curr.children[ch]
                curr.count += 1
            
            # curr.word = word
            # curr.count += 1
        
        return root
    
    def query(self, root, word):
        curr = root
        
        res = 0
        for ch in word:
            if ch not in curr.children:
                return 0
            
            curr = curr.children[ch]
            res += curr.count
        
        return res

    def sumPrefixScores(self, words: List[str]) -> List[int]:        
        root = self.buildTrie(words)
        
        res = []
        for word in words:
            curr_res = self.query(root, word)
            res.append(curr_res)
        
        return res
```