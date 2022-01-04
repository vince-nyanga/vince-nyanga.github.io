---
title: "LinkedList: Add Before, After Current Node"
excerpt: "In this post we are going to discuss how to add a new node before or after a specific node in a doubly-linked list..."
header:
  overlay_image: /images/code.jpeg
date: 2022-01-04
tags: [Data Structures, LinkedList]
---

In the [previous post]({{ site.baseurl}}/linkedlist-search) we spoke about how to search for items in a linked list. In this post we are going to talk about how we can add a new node before or after a specific node in a doubly-linked list. This is simply a matter of moving references (pointers) around.

## AddBefore

To add a new node before a specified (current) node we need to update references for a couple of nodes. We need to set the `Next` reference for the new node to the current node, set the `Previous` reference for the new node to the current node's `Previous` reference and then update the current node's `Previous` to point to the new node. If the current node already had a `Previous` reference node, set that node's `Next` reference to the new node. Here's the code:

```csharp
public void AddBefore(Node<T> currentNode, T value)
{
    if (currentNode is null)
    {
        throw new ArgumentNullException(nameof(currentNode));
    }
    _count++;
    Node<T> newNode = new() { Value = value };
    var temp = currentNode;
    newNode.Next = currentNode;
    newNode.Previous = temp.Previous;
    currentNode.Previous = newNode;
    if (newNode.Previous != null)
    {
        newNode.Previous.Next = newNode;
    }

    if (newNode.Previous is null)
    {
        _head = newNode;
    }
}
```

<figure>
<img src="{{ site.baseurl }}/images/add-before.png" alt="Doubly-linked list">
<figcaption>Figure 1: Add before</figcaption>
</figure>

## AddAfter

To add a new node after the current node we set the new node's `Previous` to the current node, set the new node's `Next` to the current node's `Next` and set the current node's `Next` to the new node. Lastly, if the current node already had an existing `Next` reference we need to update it's `Previous` to point to the new node.

```csharp
public void AddAfter(Node<T> currentNode, T value)
{
    if (currentNode is null)
    {
        throw new ArgumentNullException(nameof(currentNode));
    }
    _count++;
    Node<T> newNode = new() { Value = value };
    Node<T> temp = currentNode;
    newNode.Previous = currentNode;
    newNode.Next = temp.Next;
    currentNode.Next = newNode;
    if (newNode.Next != null)
    {
        newNode.Next.Previous = newNode;
    }

    if (newNode.Next is null)
    {
        _tail = newNode;
    }
}
```

<figure>
<img src="{{ site.baseurl }}/images/add-after.png" alt="Doubly-linked list">
<figcaption>Figure 2: Add after</figcaption>
</figure>

In this post we spoke about how to add a new node before or after a specified node. This is all about updating the references (pointers) properly to ensure that our linked list 'chain' is not broken. In the next post we are going to attempt a HackerRank problem relating to a doubly-linked list. Till next time, have a good one.
