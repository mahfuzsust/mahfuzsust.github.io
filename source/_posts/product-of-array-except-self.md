---
title: Product of Array Except Self
date: 2020-08-16 13:23:19
tags:
  - java
  - array
---

Given an array `nums` of *n* integers where *n* > 1, return an array `output` such that `output[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

**Example:**

**Input:**`[1,2,3,4]`
**Output:**`[24,12,8,6]`

Problem link: [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self)

Solution:

Let’s start with a brute-force solution.

    public int[] productExceptSelf(int[] nums) {
        int[] arr = new int[nums.length];
        for(int i = 0; i < nums.length; i++) {
            int mul = 1;
            for (int j = 0; j < nums.length; j++) {
                if(i == j) {
                    continue;
                }
                mul *= nums[j];
            }
            arr[i] = mul;
        }
        return arr;
    }

Clearly, this is not an efficient solution and the time complexity for this solution is O(n²)

We can improve the implementation using the help of division. We can get the total multiplication value and for each element we divide the item.

    public int[] productExceptSelf(int[] nums) {
        int mul = 1;
        for(int i = 0; i < nums.length; i++) {
            mul *= nums[i];
        }
        
        for (int i = 0; i < nums.length; i++) {
            nums[i] = mul / nums[i];
        }
        return nums;
    }

This solution seems okay. But we missed an important edge case here. The existence of zero(0). If there is any input 0, it will cause division by zero exception.

To solve the division by zero exception, we can think of two use cases

1. One 0 exists in the input
2. 0 exists more than once.

For the use case for 0 existing more than once, we can easily define the result array will have all zeroes inside.

But, if we have only one 0, then we will only get output for that item only.

**Input:**`[1,2,0,4,5]`
**Output:**`[0,0,40,0,0]`

we can solve the problem using zero counter easily.

    public int[] productExceptSelf(int[] nums) {
        int mul = 1, countZero = 0;
        for(int i = 0; i < nums.length; i++) {
            if(nums[i] == 0) {
                countZero++;
            } else {
                mul *= nums[i];
            }
        }
        if(countZero > 1) {
            return new int[nums.length];
        }
        for (int i = 0; i < nums.length; i++) {
            if(countZero > 0) {
                if(nums[i] == 0) nums[i] = mul;
                else nums[i] = 0;
            } else {
                nums[i] = mul / nums[i];
            }
        }
        return nums;
    }

There is a note in the problem

**Note: **Please solve it **without division** and in O(*n*).

To solve without division, we first need to find pattern of the problem. Let’s find it with provided example input.

**Input:**`[1,2,3,4]`
**Output:**`[24,12,8,6]`

We can distribute the input in left and right section, so that we can multiply left and right to get our result.

    Index: 0
    Nums[Index] = 1 (default)
    Left[Index] = 1
    Right[Index] = 2 * 3 * 4 = 24
    Result[Index] = Left[Index] * Right[Index] = 1 * 24 = 24
    
    Index: 1
    Nums[Index] = 2
    Left[Index] = 1
    Right[Index] = 3 * 4 = 12
    Result[Index] = Left[Index] * Right[Index] = 1 * 12 = 12
    
    Index: 2
    Nums[Index] = 3
    Left[Index] = 1 * 2 = 2
    Right[Index] = 4
    Result[Index] = Left[Index] * Right[Index] = 4 * 2 = 8
    
    Index: 3
    Nums[Index] = 4
    Left[Index] = 1 * 2 * 3 = 6
    Right[Index] = 1 (default)
    Result[Index] = Left[Index] * Right[Index] = 6 * 1 = 6

To generate our left array

    left[i] = left[i - 1] * hums[i - 1]

And our right array

    right[i] = right[i + 1] * hums[i + 1]

So, our solution will be

    public int[] productExceptSelf(int[] nums) {
        int[] left = new int[nums.length];
        left[0] = 1;
        for (int i = 1; i < nums.length; i++) {
            left[i] = left[i - 1] * nums[i - 1];
        }
        int[] right = new int[nums.length];
        right[nums.length - 1] = 1;
        for (int i = nums.length - 2; i >= 0; i--) {
            right[i] = right[i + 1] * nums[i + 1];
        }
        for (int i = 0; i < nums.length; i++) {
            nums[i] = left[i] * right[i];
        }
        return nums;
    }

Please share any other better solution for this problem.
