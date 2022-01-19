---
title: "Popping The Root From A Binary Heap"
excerpt: "In this post we are going to talk about how to pop the root of a binary heap"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-19
tags: [Data Structures, Binary Heap]
---

In the [previous post]({{ site.baseurl}}/binary-heap-intro) we introduced a binary heap data structure and implemented an algorithm to insert an item into a binary min heap. In this post we are going to talk about how to pop the root of a binary heap.

## The Code

When popping (or deleting) the root, we replace the root with the last item in the list. After that we 'bubble down', starting from the root, comparing the node with its children and swapping them if it's greater than the child. The time complexity is $$ O(\log(n)) $$. Here's the code:

```csharp
public T Pop()
{
    if (_items.Count == 0)
    {
        throw new IndexOutOfRangeException();
    }

    var item = _items[0];
    ReplaceWithLastItem(0);
    return item;
}

private void ReplaceWithLastItem(int index)
{
    var i = _items.Count - 1;
    _items[index] = _items[i];
    _items.RemoveAt(i);

    BubbleDown(index);
}

private void BubbleDown(int index)
{
    while (HasLeftChild(index))
    {
        var leftChildIndex = GetLeftChildIndex(index);
        var rightChildIndex = GetRightChildIndex(index);
        var swapIndex = index;

        if (leftChildIndex < _items.Count &&
            _comparer.Compare(_items[swapIndex], _items[leftChildIndex]) > 0)
        {
             swapIndex = leftChildIndex;
        }

        if (rightChildIndex < _items.Count &&
            _comparer.Compare(_items[swapIndex], _items[rightChildIndex]) > 0)
        {
            swapIndex = rightChildIndex;
        }

        if (swapIndex != index)
        {
            Swap(swapIndex, index);
            index = swapIndex;
        }
        else
        {
            break;
        }
    }
}
```

That's it, this is how we pop the root of a binary heap. All the code for this post is available on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to make this more generic to allow us to remove any node in a binary heap. Till next time, thanks so much for taking your time to read. You are welcome to leave a comment if you have a question and/or suggestion.
