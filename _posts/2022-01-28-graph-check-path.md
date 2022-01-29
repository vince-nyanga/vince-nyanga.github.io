---
title: "Check If There Is A Path Between Two Vertices In A Graph"
excerpt: "In this post we are going to implement an algorithm that checks if there is a path between two vertices in a graph"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-28
tags: [Data Structures, Graph]
---

In the [previous post]({{ site.baseurl}}/graph-traversal) we spoke about two different ways to traverse a graph. In this post we are going to solve a [LeetCode](https://leetcode.com/problems/find-if-path-exists-in-graph/) problem (slightly edited) that requires us to check if a path exists between two vertices in a graph.

## Code

We are going to use a breadth first traversal for this problem. This is because if there is a path between two vertices will will find it faster than if we used depth first traversal. Remember, depth first traversal takes a branch and goes all the way to the end. If we happened to take a wrong branch then we will be on a wild goose chase till the end of the branch while there might be an edge connecting our two nodes. We are going to run our traversal until we find the destination vertex then return `true` otherwise, when we complete our traversal and we haven't found the destination vertex then we return `false`. The time complexity for this algorithm is $$ O(n + k) $$ where $$ n $$ is the number of vertices and $$ k $$ is the number of edges in the graph. The space complexity is $$ O(n) $$. Here's the code:

```csharp
public bool HasPath(T source, T destination)
{
    if (!_adjacentList.ContainsKey(source) || !_adjacentList.ContainsKey(destination))
    {
        return false;
    }
    Queue<T> queue = new();
    HashSet<T> visited = new();
    queue.Enqueue(source);

    while(queue.Count > 0)
    {
        var current = queue.Dequeue();
        if (current.CompareTo(destination)==0)
        {
            return true;
        }

        if (!visited.Contains(current))
        {
            visited.Add(current);
            foreach (var neigbor in _adjacentList[current])
            {
                queue.Enqueue(neigbor);
            }
        }
    }
    return false;
}
```

In this post we implemented an algorithm that checks if a path exists between two vertices in a graph. All the code is available on [GitHub](https://github.com/vince-nyanga/data-structures) if you want to check it. Thanks for reading and don't hesitate to leave a comment below if you have a question or suggestion.
