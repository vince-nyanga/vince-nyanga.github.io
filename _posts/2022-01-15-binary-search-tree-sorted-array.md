---
title: "Converting A Sorted Array Into A Balanced BST"
excerpt: "In this post we are going to solve another LeetCode problem where we convert a sorted array into a binary search tree"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-15
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-remove-node) we spoke about how to remove a node from a binary search tree. In this post we are going to tackle another problem from [LeetCode](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) where we convert a sorted array into a balance binary search tree.

## Problem

Given an integer array `nums` where the elements are sorted in **ascending order**, convert it to a **_height-balanced_** binary search tree. A height-balanced binary tree is a binary tree in which the depth of the two subtrees of every node never differs by more than one.

## Solution

To solve this problem we start in the middle of the array and create our root node. Then we recursively do the same for the left and right subtrees. The time complexity for this algorithm is $$ O(n) $$ while the space complexity is $$ O(h) $$ where $$ h $$ is the height of the tree. For the first time, I'm going to use the code I submitted to LeetCode because I'm feeling very lazy :stuck_out_tongue_winking_eye:

```csharp
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     public int val;
 *     public TreeNode left;
 *     public TreeNode right;
 *     public TreeNode(int val=0, TreeNode left=null, TreeNode right=null) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
public class Solution {
    public TreeNode SortedArrayToBST(int[] nums) {
        return InsertIntoTree(nums, 0, nums.Length - 1);
    }

    private TreeNode InsertIntoTree(int[] nums, int start, int end){
        if (start > end){
            return null;
        }

        int mid = start + end >> 1;
        var node = new TreeNode(nums[mid]);

        node.left = InsertIntoTree(nums, start, mid - 1);
        node.right = InsertIntoTree(nums, mid + 1, end);

        return node;
    }
}
```

Using a bitwise right shift `>>` helped me to improve the performance, albeit by a negligible margin. For those who may not know how bitwise shifts work here's simplified explanation: A right bitwise shift `a >> b` is equivalent to $$\frac{a}{2^b} $$ while a left bitwise shift `a << b` is equivalent to $$ a \times 2^b $$.

In this post we solved a LeetCode problem that required us to create a height-balanced binary search tree from a sorted array. I'm pretty sure there are better solutions out there so feel free to leave a comment below.
