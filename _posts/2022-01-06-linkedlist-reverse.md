---
title: "Reverse Doubly-Linked List"
excerpt: "In this post we are going to talk about reversing a doubly-linked list"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-06
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-insert-sorted) we spoke about inserting a new node into a sorted doubly-linked list and ensure it stays sorted. In this post we are going to look at one of the famous algorithms for a doubly-linked list -- reversing it.

## Solution

Reversing a doubly-linked list is simply a matter of swapping the `Next` and `Previous` references for all the nodes. Time complexity for the algorithm is $$ O(n) $$ and auxiliary space complexity of $$ O(1) $$. Here's the code:

```csharp
public void Reverse()
{
    if (_count > 1)
    {
        Node<T> temp = null;
        Node<T> current = _head;
        _tail = _head;

        while(current != null)
        {
            temp = current.Previous;
            current.Previous = current.Next;
            current.Next = temp;
            current = current.Previous;
        }

        _head = temp.Previous;
    }
}
```

Another approach is to swap around the values and leave the references unchanged. To achieve this we go through the linked list pushing the values into a stack, and then come back again popping the values from the stack and update the nodes' values. This is not an efficient way of doing it since it has both time and auxiliary space complexity of $$ O(n) $$.

In the next post we are going to look at removing duplicates from a doubly-linked list. Check out [GitHub](https://github.com/vince-nyanga/data-structures) for the complete code. Till next time, thanks for taking your time to read.
