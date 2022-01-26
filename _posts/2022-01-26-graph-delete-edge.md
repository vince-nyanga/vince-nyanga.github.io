---
title: "Removing Edge Between Two Vertices In A Graph"
excerpt: "In this post we are going to talk about how to remove an edge between two vertices"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-26
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-delete-vertex) we spoke about how to remove a vertex from a graph. In this post we are going to talk about how to remove an edge between two vertices.

## The Code

Since our graph is an undirected, when we remove the connection in the source vertex we need to do the same in the destination vertex. With the implementation that we have this will have a time complexity of $$ O(1) $$. Here's the code:

```csharp
public bool RemoveVertex(T vertex)
{
    if (!_adjacentList.ContainsKey(vertex))
    {
        return false;
    }

    foreach(var connection in _adjacentList[vertex])
    {
        _adjacentList[connection].Remove(vertex);
    }

    _adjacentList.Remove(vertex);
    return true;
}
```

That's it. This is how we remove an edge between two vertices in our graph model. If you want to take a look at the complete code check it out on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to talk about different ways to traverse a graph data structure. Till next time, thanks for taking your time to read.
