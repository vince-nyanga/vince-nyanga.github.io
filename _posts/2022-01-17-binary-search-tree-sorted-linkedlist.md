---
title: "Converting A Sorted LinkedList Into A Balanced BST"
excerpt: "In this post we are going to solve another LeetCode problem where we convert a sorted linked list into a balanced BST."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-17
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-validate) we solved a LeetCode problem that required us to check if a binary tree is a valid binary search tree. In this post we are going to solve yet another LeetCode problem. We are going to create a height-balanced binary search tree from a sorted singly linked list.

## Problem

Given the `head` of a singly linked list where elements are sorted in **ascending order**, convert it to a height balanced BST. For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.

## Solution

Just like when we create a binary search tree from a sorted array, we start in the middle of the linked list and create our root node. We then recursively to the same for the left and right subtrees. To find the middle of a singly linked list we are going to use the runner technique where the `fastNode` moves forward twice as fast as the `slowNode`. When the `fastNode` reaches the end of the list, the `slowNode` will be at the middle of the list. The time complexity for this algorithm is $$ O(n\log(n)) $$. Here's the code:

```csharp
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public int val;
 *     public ListNode next;
 *     public ListNode(int val=0, ListNode next=null) {
 *         this.val = val;
 *         this.next = next;
 *     }
 * }
 */
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
    public TreeNode SortedListToBST(ListNode head) {
        if (head is null)
        {
            return null;
        }

        if (head.next is null)
        {
            return new TreeNode(head.val);
        }

        var slowNode = head;
        var fastNode = head;
        ListNode previous = null;

        while(fastNode != null && fastNode.next != null){
            previous = slowNode;
            slowNode = slowNode.next;
            fastNode = fastNode.next.next;
        }

        previous.next = null;
        var node = new TreeNode(slowNode.val);
        node.left = SortedListToBST(head);
        node.right = SortedListToBST(slowNode.next);
        return node;
    }
}
```

In this post we solve another LeetCode problem. We had to create a height-balanced binary search tree from a sorted singly linked list. This will be the last post in the binary search tree series. Next time we are going to look at binary heaps. Once again, thank you for taking your time to read.
