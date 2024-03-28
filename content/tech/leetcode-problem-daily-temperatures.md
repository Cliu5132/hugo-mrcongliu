---
title: "Leetcode Problem Daily Temperatures"
date: 2024-03-28
---

This [problem](https://leetcode.com/problems/daily-temperatures/) is from Leetcode with a description as follows:

> Given an array of integers temperatures represents the daily temperatures, return an array answer such that answer[i] is the number of days you have to wait after the ith day to get a warmer temperature. If there is no future day for which this is possible, keep answer[i] == 0 instead.

My initial solution was very hard to read and failed to use stack efficiently.

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        days = []
        p = 0
        size = len(temperatures)
        answer = [0] * size
        while p<size:
            if len(days)>0:
                t = temperatures[p]
                day = days[len(days)-1]
                if t>temperatures[day]:
                    answer[day]=p-day
                    days.pop()
                else:
                    days.append(p)
                    p=p+1
            else:
                days.append(p)
                p=p+1

        return answer
```

Though it worked, its runtime only beat 23% of users with Python3. That's ~~embracing~~ embarrassing!

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/c410618a-7271-0247-22a8-30ce11df3299.png)

Finally, I came up with this solution. (With the help of ChatGPT 🤣)

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        stack = []  # Stack to store indices of temperatures
        result = [0] * len(temperatures)

        for i, temp in enumerate(temperatures):
            # Check if the current temperature is higher than temperatures at indices in the stack
            while stack and temp > temperatures[stack[-1]]:
                prev_index = stack.pop()
                result[prev_index] = i - prev_index
            stack.append(i)  # Add the current index to the stack

        return result
```

As a result, its runtime now beat 73% of users with Python3.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/a363fcbc-b737-c7db-dc99-bf4e22c97e56.png)

Why am I so bad at Leetcode? 🤔