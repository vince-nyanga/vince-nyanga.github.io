---
title: "Insert Item Into A Sorted LinkedList"
excerpt: "In this post we are going to try a HackerRank challenge relating to a doubly-linked list"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-05
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-add-before-after) we spoke about how we can insert a new node before or after a specified node in a doubly-linked list. In this post we are going to tackle a [HackerRank](https://www.hackerrank.com/challenges/insert-a-node-into-a-sorted-doubly-linked-list/problem?isFullScreen=true&h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=linked-lists) problem. Given a sorted doubly-linked list, we want to insert a new node in its correct position to keep the linked list sorted.

## Solution

Before we look at the solution to the problem we need to update our `HDLinkedList` class to ensure that our generic property implements `IComparable`. We want it to implement `IComparable` so we may be able to compare the new value to those already in the list in order to insert it in its proper location.

```csharp
public class HDLinkedList<T> where T: IComparable
{
    // ...
}
```

Now that our generic property implements `IComparable`, let's get started on implementing the algorithm to insert a new node while keeping the list sorted. If the list is empty then we just add the item and the list is already sorted (list containing one element is sorted). If the list is not empty then we check if the head's value is greater than the new value. If it is then `AddFirst()` otherwise keep moving until you reach a node whose value is greater than the new value and add before it. The time complexity of this algorithm is **_O(n)_** unless if the list is empty or the head is greater than the new value where the complexity will be **_O(1)_**. Here's the code:

```csharp
public void InsertSorted(T value)
{
    if (_head is null)
    {
        AddFirst(value);
    }
    else if (_head.Value.CompareTo(value) >= 0)
    {
        AddFirst(value);
    }
    else
    {
        var node = _head;
        while(node.Next != null && node.Next.Value.CompareTo(value) < 0)
        {
            node = node.Next;
        }
        AddAfter(node, value);
    }
}

```

In this post we spoke about how we can insert a new node into a sorted doubly-linked list and ensure that it stays sorted. This problem can be found on [HackerRank](https://www.hackerrank.com/challenges/insert-a-node-into-a-sorted-doubly-linked-list/problem?isFullScreen=true&h_l=interview&playlist_slugs%5B%5D=interview-preparation-kit&playlist_slugs%5B%5D=linked-lists), albeit with different models. If you want the complete code for this post check it out on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to talk about reversing a doubly-linked list. Thanks so much for taking your time to read.
