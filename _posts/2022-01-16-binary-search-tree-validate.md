---
title: "Validate A Binary Search Tree"
excerpt: "In this post we are going solve another LeetCode problem where we validate a binary search tree"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-16
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-sorted-array) we solved a LeetCode problem that required us to create a height-balance binary search tree from a sorted array. In this problem we are going to look at another LeetCode problem. This time we are validating that a binary tree is a valid binary search tree.

## Problem

Given the `root` of a binary tree, determine if it is a valid binary search tree (BST).

A valid BST is defined as follows:

- The left subtree of a node contains only nodes with keys **less than** the node's key.
- The right subtree of a node contains only nodes with keys **greater than** the node's key.
- Both the left and right subtrees must also be binary search trees.

## Solution

In order to solve this problem we do an in-order traversal of the binary tree updating a global minimum value as we go along. Once we find a node that is less than (or equal) the minimum we know it's not a valid binary search tree since in-order traversal visits the nodes in ascending order. The time complexity of this algorithm is $$ O(n) $$ while the space complexity is $$ O(h) $$ where $$ h $$ is the height of the tree. Here's the code:

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
    public bool IsValidBST(TreeNode root) {
        var min = Int64.MinValue;
        return CheckValidity(root, ref min);
    }

    private bool CheckValidity(TreeNode node, ref Int64 min){
        if (node is null){
            return true;
        }


        if (!CheckValidity(node.left, ref min)){
            return false;
        }

        if (node.val <= min){
            return false;
        }

        min = node.val;

        return CheckValidity(node.right, ref min);
    }
}
```

In this post we solved another LeetCode problem that required us to check if a binary tree is a valid binary search tree. Again, I used the code I submitted to LeetCode so there's nothing on the GitHub repository. Till next time, thanks for reading and don't hesitate to leave a comment below.
