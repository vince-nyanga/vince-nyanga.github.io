---
title: "Solving Jesse And Cookies Problem On HackerRank"
excerpt: "In this post we are going to use a binary heap to solve the Jesse and cookies problem on HackerRank"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-22
tags: [Data Structures, Binary Heap]
---

In the [previous post]({{ site.baseurl}}/binary-heap-find-kth-largest) we solved a LeetCode problem that required us to find the $$ k^{th} $$ largest element in an unsorted array. In this post we are going to solve the Jesse and cookies problem on [HackerRank](https://www.hackerrank.com/challenges/jesse-and-cookies/problem).

## Problem

Jesse loves cookies and wants the sweetness of some cookies to be greater than value $$ k $$ . To do this, two cookies with the least sweetness are repeatedly mixed. This creates a special combined cookie with:

sweetness $$ = (1 \times \, $$ Least sweet cookie $$ + \,2 \times \,$$ 2nd least sweet cookie).

This occurs until all the cookies have a sweetness $$ \geq k$$.

Given the sweetness of a number of cookies, determine the minimum number of operations required until we have all cookies with sweetness $$ \geq k$$. If it is not possible, return $$ -1 $$.

## Solution

We are going to solve this problem using a min heap. Firstly, we add all the elements in the list to a min heap. This has time complexity of $$ O(n\log(n)) $$. Then after that we are going to iterate through our heap. If the root's value is less than $$ k $$, we `pop()` it and the one after it, mix them together using the formula above, and insert the new cookie back into the min heap then increment `count`. We keep doing this until the root has a value greater or equal to $$ k $$ or there's one item left in the heap. If there's one item left and it's less than $$ k $$ we return $$ -1 $$. Here is the code:

```csharp
class Result
{

    /*
     * Complete the 'cookies' function below.
     *
     * The function is expected to return an INTEGER.
     * The function accepts following parameters:
     *  1. INTEGER k
     *  2. INTEGER_ARRAY A
     */

    static List<int> _items;
    public static int cookies(int k, List<int> A)
    {
        _items = new List<int>();
        int count = 0;
        foreach (var item in A){
            Insert(item);
        }

        while (Peek() < k && _items.Count > 1){
            Insert(Pop() + 2 * Pop());
            count++;
        }

        if (Peek() < k){
            count = -1;
        }

        return count;
    }

    static int GetParentIndex(int i) => (i - 1) / 2;
    static int GetLeftIndex(int i) => 2 * i + 1;
    static int GetRightIndex(int i) => 2 * i + 2;

    static int Peek() => _items[0];

    static int Pop() {
        var val = _items[0];
        var i = _items.Count - 1;
        _items[0] = _items[i];
        _items.RemoveAt(i);
        BubbleDown();
        return val;
    }

    static void Insert(int val){
        _items.Add(val);
        BubbleUp();
    }

    static void BubbleUp(){
        var i = _items.Count - 1;
        while (i > 0){
            var parent = GetParentIndex(i);
            if (_items[i] < _items[parent]){
                Swap(i, parent);
                i = parent;
            }else{
                break;
            }
        }
    }

    static void BubbleDown(){
        int i = 0;
        while(true){
            var left = GetLeftIndex(i);
            var right = GetRightIndex(i);
            var min = i;
            if (left < _items.Count && _items[min] > _items[left]){
                min = left;
            }

            if (right < _items.Count && _items[min] > _items[right]){
                min = right;
            }

            if (min == i){
                break;
            }

            Swap(i, min);
            i = min;
        }

    }

    static void Swap(int x, int y){
        var temp = _items[x];
        _items[x] = _items[y];
        _items[y] = temp;
    }

}
```

In this post we solved the Jesse and cookies problem on HackerRank using a min heap. This is going to be the last post in the binary heap mini series. In the next post we are going to start a mini series on graphs. Please feel free to leave a question and/or suggestion. Thanks so much for reading.
