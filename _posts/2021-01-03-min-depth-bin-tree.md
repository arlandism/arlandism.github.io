---
layout: article
title: Minimum Depth Binary Trees
date: 2021-01-03 16:00:00PM
categories: programming
tags:
- programming
- python
- algorithms
---

In preparation for [Bradfield's CSI program](https://bradfieldcs.com/csi), I'm going through an [algos book](https://bradfieldcs.com/algos/) and solving some Leetcode problems that they recommend. Concurrently, I'm also reading [How to Make It Stick](https://www.amazon.com/Make-Stick-Science-Successful-Learning/dp/0674729013) which recommends, among other things, reflecting on your learnings to help flush them out of working memory and into long-term memory. In that spirit of that, I figured I might as well blog my exploits.

# Problem
Minimum Depth of Binary Tree

Given a binary tree structured as an array, return the number of nodes in the shortest path from the root of the binary tree to a leaf node.

Input:
```
[3,9,20,None,None,15,7]
```

Output:
```
2
```

Below is a poorly drawn visualization of this structure.
```
     3
9        20

    15       7
```

In this example, 9 is a leaf node because it has no children. All of the possible paths for this tree are:
3->9,
3->20->15,
3->20->7

Clearly the first path is the shortest, with only 2 nodes.

# Solution

So one solution to this problem is the following rough algorithm:
1) Find every path from the root to a leaf node
2) Pick the shortest

Finding a path from the root to a leaf node involves recursively moving down the tree and keeping track of the path we're taking.

Leetcode's representation of a tree node is:

```python
class TreeNode:
  def __init__(self, val=0, left=None, right=None):
    self.val = val
    self.left = left
    self.right = right
```

They also force a `Solution` class with a `minDepth` method, so here's my take:

``` python
def debug_list(l):
    print([node.val for node in l])

class Solution:
    def minDepth(self, root):
      smallest_path = self.find_path(root, [])

      return len(smallest_path)

    def find_path(self, node, path):
        if node:
          left_paths = []
          right_paths = []
          if node.left:
            left_paths = self.find_path(node.left, path + [node])

          if node.right:
            right_paths = self.find_path(node.right, path + [node])

          if not node.left and not node.right:
            return path + [node]

          if len(left_paths) == 0:
            return right_paths
          elif len(right_paths) == 0:
            return left_paths
          elif len(left_paths) <= len(right_paths):
            return left_paths
          else:
            return right_paths

        return path
```

It's a lot of bookkeeping around weird states and honestly doesn't feel very elegant. The basic idea is what I mentioned: recursively build out the left and right subtrees and take the lowest of the two. The sub cases are:

1) The node may not have a left and if that's the case I want the left_path to be empty so I can explicitly check for that state later.

2) Same point, but with the right.

3) If there's neither a left or right then it's a leaf node and I just want to return the path so far plus the node.

4) Here's the special case from (1). If the left is empty, return the right.

5) Same point, but with the right.

6) Otherwise, perform a normal comparison of the two paths and return the smallest.

To be honest I'm not especially pleased with this solution and have strong sense it could be more elegant. If I have time I'll probably come back to it.
