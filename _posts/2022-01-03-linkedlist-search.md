---
title: "Searching For Items In A LinkedList"
excerpt: "In this post we are going to discuss how search to for items in a doubly-linked list..."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-03
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-traverse) we spoke about how to traverse a doubly-linked list going forward as well as going backwards. In this post we are going to talk about how we can search for items in a linked list. We are going to implement to methods -- `FindFirst()` and `FindLast()`.

## FindFirst

In this method we are going to start at the head on the list and move forward until we find the first item that matches our criteria. The best case scenario is when the head matches the criteria which means we will have $$ O(1) $$ complexity. However, $$ O(n) $$ should be the most common complexity. Here's the implementation:

```csharp
public Node<T> FindFirst(T value)
{
    var node = _head;
    while (node != null)
    {
        if (node.Value.Equals(value))
        {
            break;
        }

        node = node.Next;
    }

    return node;
}
```

## FindLast

To find the last item that matches our search criteria, we start at the tail and then move backwards until we find the first item that matches our criteria. Like `FindFirst()`, the best case scenario is when the tail matches our criteria, of which the time complexity is only $$ O(1) $$. The general complexity for this algorithm is $$ O(n) $$.

The other way of going about it is to start at the head and move forward until we find the last item that matches our criteria. This is not very efficient since we will miss the opportunity to get what we want with $$ O(1) $$ time complexity if the tail matches our criteria. Doing it this way will have a guaranteed $$ O(n) $$ complexity. I have to admit, that's how I had initially implemented it until I realised while writing this post that there is actually a more efficient way of doing it. Here's the implementation:

```csharp
public Node<T> FindLast(T value)
{
    var node = _tail;
    while (node != null)
    {
        if (node.Value.Equals(value))
        {
            break;
        }

        node = node.Previous;
    }

    return node;
}
```

This is how I implemented `FindFirst()` and `FindLast()`. There might be better ways of implementing it. If you know of any please leave a comment below. In the next post we will talk about adding a new node before or after a specific node. Till next time, thanks for taking your time to read. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures).
