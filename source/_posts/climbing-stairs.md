---
title: Climbing Stairs
slug: climbing-stairs
date: 2021-05-23T05:48:13.000Z
date_updated: 2021-05-23T05:48:13.000Z
tags: 
  - fibonacci
  - algorithm
  - java
---
## Problem 
Each time you can either climb `1` or `2` steps. In how many distinct ways can you climb to the top?

    Example 1:
    
    Input: n = 2
    Output: 2
    Explanation: There are two ways to climb to the top.
    1. 1 step + 1 step
    2. 2 steps
    
    Example 2:
    
    Input: n = 3
    Output: 3
    Explanation: There are three ways to climb to the top.
    1. 1 step + 1 step + 1 step
    2. 1 step + 2 steps
    3. 2 steps + 1 step

## Solution

This is a good example of fibonacci implementation.
{% codeblock lang:java %}
class Solution {
    private int[] val = new int[46];
    public int getValue(int n) {
        if(n < 0) return 0;
        if(val[n] == 0) {
            val[n] = climbStairs(n);
        }
        return val[n];
    }
    
    public int climbStairs(int n) {
        if(n <= 0) return 0;
        if(n == 1) return 1;
        if(n == 2) return 2;
        return (getValue(n-1) + getValue(n-2));
    }
}
{% endcodeblock %}

