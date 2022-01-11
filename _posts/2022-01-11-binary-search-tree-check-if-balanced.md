---
title: "Check If A Binary Search Tree Is Balanced"
excerpt: "In this post we are going to talk about how to check if binary search tree is balanced."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-11
tags: [Data Structures, Binary Search Tree]
---

In the [previous post]({{ site.baseurl}}/binary-search-tree-traversal) we spoke about how we can traverse a binary search tree. In this post we are going to talk about how to check if a binary search tree is balanced.

In the [introduction post]({{ site.baseurl}}/binary-search-tree-intro) we spoke about the characteristics of a balanced binary tree. A binary tree is balanced when the height of the left subtree and the height of the right subtree do not differ by more than 1 for all subtrees in the tree. If you want a mathematical equation here it is:

Let $$ ls $$ = left subtree, $$ rs $$ = right subtree and $$ h(x) $$ = the height of a tree. A binary tree is balance if and only if the following is true for all subtrees:

$$ |h(ls) - h(rs)| \leq 1$$

## The Code

The code I used to verify if a binary tree is balanced is from Gayle McDowell's [Cracking the Coding Interview](https://www.amazon.com/Cracking-Coding-Interview-Programming-Questions/dp/0984782850). The reason I used it is because it is way more clean than what I had initially written. Therefore all credit goes to the Gayle McDowell :slightly_smiling_face:.

```csharp
public bool IsBalanced()
{
    return GetTreeHeight(_root) != int.MinValue;
}

private int GetTreeHeight(TreeNode<T> node)
{
    if (node is null)
    {
        return -1;
    }

    var leftHeight = GetTreeHeight(node.Left);
    if (leftHeight == int.MinValue)
    {
        return int.MinValue;
    }

    var rightHeight = GetTreeHeight(node.Right);
    if (rightHeight == int.MinValue)
    {
        return int.MinValue;
    }

    if (Math.Abs(leftHeight - rightHeight) > 1)
    {
        return int.MinValue;
    }
    else
    {
        return Math.Max(leftHeight, rightHeight) + 1;
    }
}
```

The method `GetTreeHeight()` recursively gets the maximum height between the left and right subtrees. The nice thing about it is that it checks if the two subtrees are balanced first. If they are not then it fails fast by returning `int.MinValue`. So to verify that the tree is balanced is simply to check if `GetTreeHeight()` did not return `int.MinValue`. This code runs with a time complexity of $$ O(n) $$ and auxiliary space complexity of $$ O(h) $$ where $$ h $$ is the height of the tree.

In this post we spoke about how we can check if a binary tree is balanced. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to talk about finding the lowest common ancestor between two nodes. Till then, have a good one and thanks for reading.
