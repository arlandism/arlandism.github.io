---
layout: article
title: Valid Parentheses
date: 2021-01-07 22:55:00PM
categories: programming
tags:
- programming
- python
- algorithms
---

This problem was easy because I've been exposed to it before. I did it as recently as a couple of months ago. Good refresher though.

# Problem
Valid Parentheses

Given a string containing just the characters: `()[]{}`, determine if the input string is valid.

Input:

```
s = "()"
s = "(]"
```

Output:

```
true
false
```

The first string is perfectly balanced, while the second starts with an "(", followed by an "]", so the parens and brackets remain unbalanced.

# Solution

The basic algorithm here is:

1) Iterate through each character of the string.

2) For each character, if it's an "opener", push it onto the stack.

3) If it's a "closer", pop the stack. The popped value should be the opposite of the character under examination. If it's not, we've got a "closer" next to the wrong "opener".

4) If the stack is empty when we need to pop it then the string is unbalanced. It means we've got a "closer" without a corresponding "opener".

5) If we've gone through every character in the string and the stack is not empty then the string is unbalanced. It means we've got an "opener" without a corresponding "closer".

Translating this to code isn't bad:

```python
class Solution:
    def isValid(self, s):
        inverse = { '}': '{', ')': '(', ']': '['}
        stack = []
        for char in s:
          if char == "(" or char == "[" or char == "{":
            stack.append(char)
          elif stack:
            val = stack.pop()
            if inverse[char] != val:
                return False
          else:
            return False

        if stack:
            return False
        return True
```
