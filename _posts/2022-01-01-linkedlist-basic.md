---
title: "Linked List: Basic Implementation"
excerpt: "Basic implementation of a doubly-linked list"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-01
tags: [Data Structures, LinkedList]
---

Complements of the new year :tada:. This is the first in a series of posts about the linked list (doubly-linked list) data structure. We are going to start by describing what a linked list is and then implement a basic linked list. In subsequent posts we are going build upon the implementation, adding more algorithms. After that, we are going to go to [HackerRank](https://www.hackerrank.com/) and see if there are linked list problems that we can solve. All the code for this series can be accessed on [GitHub](https://github.com/vince-nyanga/data-structures).

## What Is a Linked List

A linked list (doubly-linked list in this case) is a data structure that consists of a series of nodes. Each node contains the value of the node, a reference to the next node and a reference to the previous node. I like to look at a linked list as a chain of nodes, with each node connected to the one before it and after it. Below is a simple image that illustrates how a doubly-linked list looks like.

<figure>
<img src="{{ site.baseurl }}/images/linkedlist.png" alt="Doubly-linked list">
<figcaption>Figure 1: Doubly-linked list</figcaption>
</figure>

The first node (head) does not have a reference to the previous node while the last node (tail) does not have reference to the next node.

## Basic Implementation

Now let us implement our own doubly-linked list. First, we create the `Node` class:

```csharp
public record Node<T>
{
    public T Value { get; init; }
    public Node<T> Previous { get; set; }
    public Node<T> Next { get; set; }
}
```

Next we create our custom linked list class with two methods (for now) -- `AddFirst` and `AddLast`:

```csharp
public class HDLinkedList<T>
{
    private int _count;
    private Node<T> _head;
    private Node<T> _tail;
    public Node<T> First { get => _head; }
    public Node<T> Last { get => _tail; }
    public int Count { get => _count; }

    public void AddFirst(T data)
    {
        _count++;
        Node<T> newNode = new(){ Value = data };

        if (_head is null)
        {
            _head = newNode;
            _tail = newNode;
            return;
        }
        newNode.Next = _head;
        _head.Previous = newNode;
        _head = newNode;
    }

    public void AddLast(T data)
    {
        _count++;
        Node<T> newNode = new(){ Value = data };

        if (_head is null)
        {
            _head = newNode;
            _tail = newNode;
            return;
        }
        _tail.Next = newNode;
        newNode.Previous = _tail;
        _tail = newNode;
    }
}
```

As you may have noticed, we are tracking the reference to the first and last node in the list. Tracking the reference for the last node gives us the ability to add a new node to the back of the list with only **_O(1)_** time complexity. Failure to do that would incur **_O(n)_** cost since we would need to traverse the entire list to find the last node.

That's it for today. I want to keep these posts as succinct as possible for my sake and yours. In the next post we will add one or two things to our linked list implementation. Thanks for taking your time to read and once again, Happy New Year.
