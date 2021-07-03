---
title: Convert Sorted Array to Binary Search Tree
slug: convert-sorted-array-to-binary-search-tree
date: 2021-05-23T12:17:42.000Z
date_updated: 2021-05-23T12:30:16.000Z
tags: 
  - binary-search-tree
  - algorithm
  - java
categories:
  - Leetcode
---

Given an integer array `nums` where the elements are sorted in **ascending order**, convert *it to a **height-balanced** binary search tree*.

A **height-balanced** binary tree is a binary tree in which the depth of the two subtrees of every node never differs by more than one.

**Example 1:**
![](https://assets.leetcode.com/uploads/2021/02/18/btree1.jpg)
    Input: nums = [-10,-3,0,5,9]
    Output: [0,-3,9,-10,null,5]
    Explanation: [0,-10,5,null,-3,null,9] is also accepted:

![](https://assets.leetcode.com/uploads/2021/02/18/btree2.jpg)
**Example 2:**
![](https://assets.leetcode.com/uploads/2021/02/18/btree.jpg)
    Input: nums = [1,3]
    Output: [3,1]
    Explanation: [1,3] and [3,1] are both a height-balanced BSTs.

**Constraints:**

- `1 <= nums.length <= 104`
- `-104 <= nums[i] <= 104`
- `nums` is sorted in a **strictly increasing** order.

**Solution**
{% codeblock lang:java %}
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

class Solution {
    private TreeNode sortedArrayToBST(int[] nums, int p, int q) {
        if(p > q) return null;
        int mid = (q + p) / 2;
        TreeNode node = new TreeNode(nums[mid]);
        node.left = sortedArrayToBST(nums,p, mid -1);
        node.right = sortedArrayToBST(nums, mid + 1, q);
        return node;
    }
    public TreeNode sortedArrayToBST(int[] nums) {
        return sortedArrayToBST(nums, 0, nums.length - 1);       
    }
}
{% endcodeblock %}

