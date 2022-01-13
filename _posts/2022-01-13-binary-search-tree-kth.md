---
title: "Finding The Kth Smallest Item In A Binary Search Tree"
excerpt: "In this post we are going to talk about how to find the kth smallest item in a binary search tree."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-13
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-lca) we spoke about how we can find the lowest common ancestor between two nodes. In the post we are going to tackle another [LeetCode](https://leetcode.com/problems/kth-smallest-element-in-a-bst/submissions/) problem -- finding the $$ k^{th} $$ smallest element in a binary search tree.

## The Problem

Given the root of a binary search tree, and an integer $$ k $$, return the $$ k^{th} $$ smallest value (1-indexed) of all the values of the nodes in the tree.

## Solution

To find the smallest $$ k^{th} $$ smallest element we need to do an in-order traversal of the tree while incrementing a counter. Once the counter is equal to $$ k $$ then we return the current node. Time complexity of the algorithm is $$ O(h) $$ and space complexity is also $$ O(h) $$. Here is the code:

```csharp
public TreeNode<T> GetKthSmallestNode(int k)
{
    var count = 0;
    return FindKthSmallest(_root, k, ref count);
}

private TreeNode<T> FindKthSmallest(TreeNode<T> node, int k, ref int count)
{
    if (node is null)
    {
        return null;
    }

    var left = FindKthSmallest(node.Left, k, ref count);

    if (left != null)
    {
        return left;
    }

    count++;

    if(count == k)
    {
        return node;
    }

    return FindKthSmallest(node.Right, k, ref count);
}
```

Please note that it is important to pass the `count` by reference (trust me, I got burned once) since we are recursively calling `FindKthSmallest`. Passing it by value throws everything under the bus since the function calls already on the call stack might be holding onto a wrong value.

This is how we find the $$ k^{th} $$ smallest element in a binary search tree. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). Till next time, thanks so much for taking your time to read.
