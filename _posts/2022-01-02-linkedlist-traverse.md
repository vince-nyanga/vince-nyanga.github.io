---
title: "Traversing A LinkedList"
excerpt: "In this post we are going to discuss how traverse a doubly-linked list..."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-02
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-basic) we implemented a basic doubly-linked list and added a few methods to it. In this post we are going to talk about how we traverse a doubly-linked list. This is going to be very short as promised in the previous post. We are going to traverse the list going forward as well as backwards.

## Traversing Going Forward

When traversing a doubly-linked list going forward we start at the head and then move to the next node until there is no next node. This has a time complexity of $$ O(n) $$.

```csharp
public void ListItems()
{
    var node = _head;
    while(node != null)
    {
        Console.WriteLine(node.Value); // do whatever you like here
        node = node.Next;
    }
}
```

## Traversing Going Backwards

To traverse a doubly-linked list going backwards we start at the tail and move to the previous node till we reach the head. Tracking the reference to the tail node makes our lives easy since we already have the starting point. This operation also has $$ O(n) $$ time complexity. However, had we not tracked the reference to the last node we would incur double that cost because we would have to find the last node first, before traversing going backwards.

```csharp
public void ListItemsReverse()
{
    var node = _tail;
    while (node != null)
    {
        Console.WriteLine(node.Value);
        node = node.Previous;
    }
}
```

That's it. This is how we traverse a doubly-linked list forwards or backwards. If you want to look at the complete code check it out on [GitHub](https://github.com/vince-nyanga/data-structures). Next time we are going to talk about searching for items in a linked list. Thanks for reading.
