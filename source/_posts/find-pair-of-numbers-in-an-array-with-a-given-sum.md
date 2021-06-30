---
title: Find pair of numbers in an array with a given sum
date: 2020-08-21 13:16:21
tags:
  - java
  - array
  - number theory
---

There are multiple ways to find the pair of numbers in a given array. The numbers in the array can be in two ways.

1. Sorted
2. Unsorted

If the numbers are sorted then we can get easily with the complexity of O(n). The brute force approach will be easily applicable for both the sorted and unsorted array of numbers.

### Brute force:

Brute force approach will check each and every number to determine if it equals to the given sum. The complexity of this approach is O(n²).

    public void findPair(int[] array, int sum) {
        for(int i = 0; i < array.length - 1; i++) {
            for(int j = i + 1; j < array.length; j++) {
                if(array[i] + array[j] == sum) {
                    // The pair is { array[i], array[j] }
                }
            }
        }
    }

### Sorted:

In this approach, we need to sort the unsorted array(if needed). In this case, the complexity depends on the complexity of the sorting algorithm. If we choose a sorting algorithm of complexity O(n) we can solve this problem in O(n) complexity.

    public void findPair(int[] array, int sum) {
        // any sort method can be used.
        Arrays.sort(array);
        
        int i = 0, j = array.length - 1;
        
        while(i < j) {
            int pairSum = array[i] + array[j];
            if(pairSum == sum) {
                // The pair is { array[i], array[j] }
                // if we want to continue i++ otherwise break
            } else if(pairSum > sum) {
                j--;
            } else {
                i++;
            }
        }
    }

### Hash / Flag:

In this approach, we can get the pairs in O(n) complexity. We can use hashmap or flag array to set counter for each value and check using negation from the given sum.

    public void findPair(int[] array, int sum) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < array.length; i++) {
            map.put(array[i], map.getOrDefault(array[i], 0) + 1);
        }
        for (int i = 0; i < array.length; i++) {
            if(map.containsKey(sum - array[i])) {
                // The pair is { array[i], sum - array[i] }
            }
        }
    }

In this way, we will get the pairs twice. Because in each occurrence it will check with the hashmap and make a pair. We can add a set to get unique pairs.

### Metrics:

I tried to run these to get some metrics. Sharing those might help to get some understanding.
100001000001000000Brute force22ms1453msToo longSorted O(n log(n))4ms20ms293msHashmap8ms44ms276ms
