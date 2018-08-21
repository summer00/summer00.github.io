---
title:  "LintCode 网易算法题解答"
date:   2018-08-20 15:00:00 +0800
categories: [algorithm]
---

题目： https://www.lintcode.com/ladder/55/

<!--more-->
# 1. [Fibonacci](https://www.lintcode.com/problem/fibonacci/description)

```java
  public static int fibonacci3(int n) {
    int a = 0, b = 1, c = 0;

    if (n == 1) {
      return a;
    }
    if (n == 2) {
      return b;
    }

    for (int i = 3; i <= n; i++) {
      c = a + b;
      a = b;
      b = c;
    }
    return c;
  }
```

# 2. [Valid Palindrome II](https://www.lintcode.com/problem/valid-palindrome-ii/description)

```java
  public static boolean validPalindrome(String s) {
    int i = 0, j = s.length() - 1;
    boolean deleted = false;

    while (j - i > 0) {
      if (s.charAt(i) == s.charAt(j)) {
        i++;
        j--;
      } else if (s.charAt(i) == s.charAt(j - 1) && !deleted) {
        j--;
        deleted = true;
      } else if (s.charAt(i + 1) == s.charAt(j) && !deleted) {
        i++;
        deleted = true;
      } else {
        return false;
      }
    }

    return true;
  }
```

# [Binary Tree Longest Consecutive Sequence](https://www.lintcode.com/problem/binary-tree-longest-consecutive-sequence/description)

```java
  public int longestConsecutive(TreeNode root) {
    return recursiveHelper(root, null, 0);
  }

  public int recursiveHelper(TreeNode curr, TreeNode parent, int depth) {
    if (curr == null) {
      return 0;
    }
    int currDepth;
    if (parent != null && parent.val + 1 == curr.val) {
      currDepth = depth + 1;
    } else {
      currDepth = 1;
    }
    return Math.max(currDepth, Math.max(recursiveHelper(curr.left, curr, currDepth), recursiveHelper(curr.right, curr, currDepth)));
  }
```