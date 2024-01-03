---
title: "Leetcode Problem Palindrome Number"
date: 2024-01-02
---

This problem is from Leetcode with a description as follows:

> Given an integer x, return true if x is a palindrome, and false otherwise.

My initial solution using recursion exceeded the time limit of Leetcode.

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0) return false;

        int scale = 1;
        while (x >= scale * 10) {
            scale = scale * 10;
        }
        
        return isPalindromeHelper(x, scale);
    }

    public boolean isPalindromeHelper(int x, int scale) {
        if (scale < 10) return true;

        int first = (int) Math.floor(x / scale);
        int last = x % 10;

        if (first == last) {
            x = x - first * scale;
            x = (int) Math.floor(x / 10);
            scale = scale / 100;

            return isPalindromeHelper(x, scale);
        } else {
            return false;
        }
    }
}
```

Finally, I came up with this solution. I realized that using while loop is more efficient than the recursive approach. It turned out not exceeding time limit of Leetcode.

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0) return false;
        return isPalindromeHelper(x, 1);
    }

    public boolean isPalindromeHelper(int x, int scale) {
        while (x / scale >= 10) {
            scale *= 10;
        }

        while (x != 0) {
            int first = x / scale;
            int last = x % 10;

            if (first != last) {
                return false;
            }

            x = (x % scale) / 10;
            scale /= 100;
        }

        return true;
    }
}
```
