---
title: "Remove Duplicates From Linked List"
excerpt: "In this post we are going to talk about removing duplicate items from a doubly-linked list..."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-07
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-reverse) we spoke about reversing a doubly-linked list. In this post we are going to talk about removing duplicate items from a doubly-linked list.

## Solution

We are going to make use of another data structure in order to achieve our objective. We are going to use a dictionary (hash map in Java). Basically what we are going to do is iterate through the nodes in our linked list, checking if the item exists in our dictionary. If it exists, we remove it from the list by changing the `Next` reference of the node's `Previous` as well as the `Previous` reference of the node's `Next` node. The time complexity of this algorithm is $$ O(n) $$ and the auxiliary space complexity is also $$ O(n) $$. Here is the code:

```csharp
public void RemoveDuplicates()
{
    Dictionary<T, bool> visitedList = new();
    var current = _head;
    while(current != null)
    {
        if (visitedList.ContainsKey(current.Value))
        {
            if (current.Previous != null)
            {
                _count--;
                current.Previous.Next = current.Next;
                if (current.Next is null)
                {
                    _tail = current.Previous;
                }
            }
            if (current.Next != null)
            {
                current.Next.Previous = current.Previous;
            }
        }
        else
        {
            visitedList.Add(current.Value, true);
        }

        current = current.Next;
    }
}
```

This is how you can remove duplicates from a doubly-linked list. If you know of another way of doing it please leave a comment below. Complete code for this series can be found on [GitHub](https://github.com/vince-nyanga/data-structures). Once again, thanks for taking your time to read. Hopefully you gained something.
