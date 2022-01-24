---
title: "Inserting An Edge Into A Graph"
excerpt: "In this post we are going to talk about how to insert an edge into a graph"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-24
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-intro) we introduced a graph data structure and implemented a basic graph model. In this very short post we are going to talk about adding an edge between two vertices as well as checking if an edge exists between vertices.

## Adding An Edge

Our graph is an undirected graph so when we add an edge to one vertex's list we need to also do the same in the other vertex's list. Before we do that we just need to make sure the vertices exist. Time complexity for this is $$ O(1) $$ -- the reason why I used a hash map and a hash set. Here's the code:

```csharp
public void AddEdge(T source, T destination)
{
    AddVertex(source);
    AddVertex(destination);

    _adjacentList[source].Add(destination);
    _adjacentList[destination].Add(source);
}
```

## Checking If Edge Exists

Checking if an edge exists between two vertices can also be done in $$ O(1) $$ time. Here's the code:

```csharp
public bool HasEdge(T source, T destination)
{
    if (!_adjacentList.ContainsKey(source))
    {
        return false;
    }

    return _adjacentList[source].Contains(destination);
}
```

This was a very short post where we spoke about how to add an edge between two vertices in a graph. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures) if you want to check it out. Like always, thanks for reading.
