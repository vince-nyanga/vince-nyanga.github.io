---
title: "Removing A Node From A Binary Search Tree"
excerpt: "In this post we are going to talk about removing a node from a binary search tree"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-14
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-kth) we looked at finding the $$ k^{th} $$ smallest node in a binary search tree. In this post we are going to look at removing a node from a binary search tree.

## Code

If the node to be deleted is a leaf node then we just simply delete it. If the node has one child then we replace the node with the child and remove the child. However, it gets a bit tricky when the node has two children. If a node has two children we replace it with its in-order successor and remove the successor. This will be the smallest node in the right subtree of the node. Time complexity for this algorithm is $$ O(h) $$ where $$ h $$ is the height of the tree. Here's the code:

```csharp
public TreeNode<T> Delete(T value)
{
    return RemoveNode(_root, value);
}

private TreeNode<T> RemoveNode(TreeNode<T> node, T value)
{
    if (node is null)
    {
        return null;
    }

    if (node.Value.CompareTo(value) > 0)
    {
        node.Left = RemoveNode(node.Left, value);
        return node;
    }

    if (node.Value.CompareTo(value) < 0)
    {
        node.Right = RemoveNode(node.Right, value);
        return node;
    }

    if (node.Left is null)
    {
        return node.Right;
    }

    if (node.Right is null)
    {
        return node.Left;
    }

    var parent = node;
    var successor = node.Right;

    while(successor.Left != null)
    {
        parent = successor;
        successor = successor.Left;
    }

    if (parent != node)
    {
        parent.Left = successor.Right;
    }
    else
    {
        parent.Right = successor.Right;
    }

    node.Value = successor.Value;
    return node;
}
```

In this post we spoke about how to remove a node from a binary search tree. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to look at another LeetCode problem where we create a balanced binary search tree from a sorted array. Till next time, thanks so much for reading.
