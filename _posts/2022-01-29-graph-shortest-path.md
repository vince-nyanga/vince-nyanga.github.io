---
title: "Find The Shortest Path Between Vertices In A Graph"
excerpt: "In this post we are going to implement an algorithm that finds the shortest path between two vertices in a graph"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-29
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-check-path) we implemented an algorithm that checks if there is a path between two vertices in a graph. In this post we are going to implement an algorithm that finds the shortest path between two vertices in a graph.

## The Problem

Given a graph $$ G $$ return the smallest total number of edges between two vertices $$ a $$ and $$ b $$ -- the shortest path between the two vertices. If no path exists, return $$ -1 $$.

## Solution

To solve this problem we are going to use breadth first traversal. However, instead of just storing the vertices in our queue we are going to store a tuple that contains a vertex and its distance from the source vertex (the number of edges between them). As we enqueue a vertex's neighbors we should just make sure to increment the distance so that every neighbor has 1 more distance value than the current vertex. If we find the destination vertex then we just return the distance value. The time complexity for this algorithm is $$ O(n + k) $$ where $$ n $$ is the number of vertices and $$ k $$ is the number of edges in the graph. The space complexity is $$ O(n) $$. The code is similar to the one we used to check if there is a path between two vertices. Here it is:

```csharp
public int GetShortestPath(T source, T destination)
{
    if (!_adjacentList.ContainsKey(source) || !_adjacentList.ContainsKey(destination))
    {
        return -1;
    }
    Queue<(T Vertex, int Distance)> queue = new();
    HashSet<T> visited = new();
    queue.Enqueue((source, 0));

    while (queue.Count > 0)
    {
        var (vertex, distance) = queue.Dequeue();
        if (vertex.CompareTo(destination) == 0)
        {
            return distance;
        }

        if (!visited.Contains(vertex))
        {
            visited.Add(vertex);
            foreach (var neigbor in _adjacentList[vertex])
            {
                queue.Enqueue((neigbor, distance + 1));

        }
    }
    return -1;
}
```

In this post we implemented an algorithm that finds the shortest path (if it exists) between two vertices in a graph. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures) if you want to check it out. Till next time thanks so much for taking your time to read.
