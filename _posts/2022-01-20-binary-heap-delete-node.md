---
title: "Deleting A Specific Element From A Binary Heap"
excerpt: "In this post we are going to talk about how to remove a specific node from a binary heap"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-20
tags: [Data Structures, Binary Heap]
---

In the [previous post]({{ site.baseurl}}/binary-heap-pop) we spoke about popping the root node from a binary heap. In this post we are going to implement a generic method that removes a specific node from the heap. It's going to be a very short post.

## Code

Removing a node from a specific index is just like popping the root. All we need to do is replace the value at that index with the last item in the list, remove the last item. After than we move downwards (bubble down) from the current index checking if the value is greater than any of its children, swapping them if it is until we have a valid min heap. The time complexity of this algorithm is $$ O(\log(n)) $$. Here's the code:

```csharp
public void RemoveAt(int index)
{
    if (index >= _items.Count)
    {
        throw new IndexOutOfRangeException();
    }

    ReplaceWithLastItem(index);
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

In this post we spoke about how to remove a specific node from a binary heap. It was a very short post indeed. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures) if you want to check it out. In the next post we are going to solve a LeetCode problem using a binary heap.
