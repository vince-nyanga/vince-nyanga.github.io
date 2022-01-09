---
title: "Searching For An Item In A Binary Search Tree"
excerpt: "In this post we are going to talk about searching for items in a binary search tree"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-09
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-intro) we introduced a binary search tree and implemented a basic binary search tree. We also spoke about how to insert an item into a binary search tree. In this post we are going to talk about how we can search for items in a binary search tree. This, like all the other posts, is going to be a very short read.

## Searching For Items

To search for an item in a binary search tree we recursively go through the tree, moving left if the value we are looking for is less than the current node or right if it's greater. We continue doing this until we have found what we are looking for or have reached a leaf node. The average time complexity for this algorithm is $$ O(\log(n)) $$ if the tree is fairly balance. Here is the code:

```csharp
public TreeNode<T> Search(T value)
{
    return FindNode(_root, value);
}

private TreeNode<T> FindNode(TreeNode<T> node, T value)
{
    if (node is null)
    {
        return null;
    }

    if (value.CompareTo(node.Value) == 0)
    {
        return node;
    }

    if (value.CompareTo(node.Value)< 0)
    {
        return FindNode(node.Left, value);
    }

    return FindNode(node.Right, value);
}
```

In this post we spoke about how we can search for items in a binary search tree. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to talk about how traverse a binary search tree.
