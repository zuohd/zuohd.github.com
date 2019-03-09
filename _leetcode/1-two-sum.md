---
layout: page
date: 2017-05-27
title: Two Sum
tags: [Array, Hash Table]
---

# 问题

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

**例子**

Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].

**解决方案**

``` python
class Solution:
    def twoSum(self, num, target):
        tmp = {}
        for i in range(len(num)):
            if target - num[i] in tmp:
                return(tmp[target - num[i]], i)
            else:
                tmp[num[i]] = i;

```