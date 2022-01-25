---
title: "Removing A Vertex From A Graph"
excerpt: "In this post we are going to talk about how to remove a vertex from a graph"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-25
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-insert-edge) we spoke about how to insert an edge between two vertices in a graph. In this post we are going to talk about how to remove a vertex from a graph. This is also going to be a very short post.

## The Code

Removing a vertex from our graph model only takes $$ O(1) $$ time. However, before we do that we need to ensure that we remove all the edges between other vertices and the vertex we want to remove. This will take $$ O(k) $$ time, where $$ k $$ is the number of edges connected to the vertex of concern. Here's the code:

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

In this post we spoke about how to remove a vertex from a graph. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures). In the next post we are going to talk about how we can remove an edge from our graph model. That too is going to a very short post. I'm staying true to my word that my posts are going to be 2 min reads or less on average :smile:. If you have a question or suggestion don't hesitate to leave a comment below. Till next time, thanks so much for reading.
