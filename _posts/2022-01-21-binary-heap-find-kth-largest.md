---
title: "Using Binary Heap To Find The Kth Largest Element In An Unsorted Array"
excerpt: "In this post we are going to talk about how to use a binary heap to find the kth largest element in an unsorted array"
header:
  overlay_image: /images/code.jpeg
date: 2022-01-21
tags: [Data Structures, Binary Heap]
---

In the [previous post]({{ site.baseurl}}/binary-heap-delete-node) we spoke about how to delete a specific node from a binary heap. In this post we are going to solve a [LeetCode](https://leetcode.com/problems/kth-largest-element-in-an-array/) using a binary heap.

## The Problem

Given an integer array `nums` and an integer $$ k $$, return the $$
k^{th} $$ largest element in the array. Note that it is the $$ k^{th} $$ largest element in the sorted order, not the $$ k^{th} $$ distinct element.

## Solution

To solve this problem we are going to use a min heap. We are going to create a min heap of size $$ k $$ using the first $$ k $$ items in the array. From $$ k + 1 $$ to the end of the array were are going to check if the item in the array is greater than the root of our min heap. If it is greater then we replace the root with the item in the array and 'bubble down'. By the time we reach the end of our array the root of our min heap will be the $$ k^{th} $$ largest element in the array. Time complexity for creating a min heap of size $$ k $$ is $$ O(k\log(k)) $$ while the time complexity for iterating from $$ k + 1 $$ to the end while replacing the root is $$ O((n - k)\log(k)) $$. Here's the code:

```csharp
public class Solution {
    private List<int> items;
    public int FindKthLargest(int[] nums, int k) {
        items = new List<int>();
        foreach(var num in nums){
            Insert(num);
        }
        var count = 0;
        var val = int.MinValue;
        while (count < k){
            val = Pop();
            count++;
        }
        return val;
    }

    private int Pop(){
        var val = items[0];
        items[0] = items[items.Count-1];
        items.RemoveAt(items.Count-1);
        BubbleDown();
        return val;
    }

    void  BubbleDown(){
        var index = 0;
        while(true){
            var leftChildIndex = GetLeftChildIndex(index);
                var rightChildIndex = GetRightChildIndex(index);
                var swapIndex = index;

                if (leftChildIndex < items.Count)
                {
                    if (items[swapIndex] < items[leftChildIndex])
                    {
                        swapIndex = leftChildIndex;
                    }
                }

                if (rightChildIndex < items.Count)
                {
                    if (items[swapIndex] < items[rightChildIndex])
                    {
                        swapIndex = rightChildIndex;
                    }
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

    private void Insert(int value){
        items.Add(value);
        var i = items.Count - 1;
        while (i > 0)
        {
            var parentIndex = GetParentIndex(i);
            if (items[i] > items[parentIndex])
            {
                Swap(i, parentIndex);
                i = parentIndex;
            }
            else
            {
                break;
            }
        }

    }

    private void Swap(int firstIndex, int secondIndex)
    {
        var temp = items[firstIndex];
        items[firstIndex] = items[secondIndex];
        items[secondIndex] = temp;
    }

    private  int GetParentIndex(int index) => (index - 1) / 2;
    private  int GetLeftChildIndex(int index) => (index * 2) + 1;
    private  int GetRightChildIndex(int index) => (index * 2) + 2;
}
```

In this post we solved a problem from LeetCode that required us to find the $$ k^{th} $$ largest element an unsorted array. There are many ways of solving this problem, including just sorting the array in descending order and returning the $$ (k - 1)^{th} $$ element. I decided to use a min heap to showcase how a min heap can be used to solve the problem. If you have any questions or suggestions please leave a comment below.
