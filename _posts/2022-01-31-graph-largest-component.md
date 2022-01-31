---
title: "Find The Largest Component In A Graph"
excerpt: "In this post we are going to implement an algorithm that finds the largest component in a graph"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-31
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-component-count) we implemented an algorithm that counts the number of connected components in a graph. In this post we are going to talk about how we can find the largest connected component -- the one with the most vertices.

## The Code

This algorithm is very similar to the one we used to count the number of connected components. Instead of returning a boolean value from our recursive function, we will return the number of vertices in a component. Here's the code:

```csharp
public int GetLargestComponentSize()
{
    HashSet<T> visited = new();
    if (_adjacentList.Count == 0)
    {
        return 0;
    }
    var largestSize = 0;
    foreach(var vertex in _adjacentList.Keys)
    {
        var size = GetComponentSize(vertex, ref visited);
        if (size > largestSize)
        {
            largestSize = size;
        }
    }

    return largestSize;
}

private int GetComponentSize(T vertex, ref HashSet<T> visited)
{
    if (visited.Contains(vertex))
    {
        return int.MinValue;
    }

    var componentSize = 1;
    visited.Add(vertex);

    foreach(var neighbor in _adjacentList[vertex])
    {
        componentSize += GetComponentSize(neighbor, ref visited);
    }

    return componentSize;
}
```

In this post we spoke about how to find the largest connected component size in a graph. This is going to be the last post in this series. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). Till next time, thanks for reading (I always want to type _thanks for watching_ :stuck_out_tongue_winking_eye:)
