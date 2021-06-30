---
title: How to implement cache in Java
date: 2020-08-25 12:45:43
tags:
  - java
  - cache
---

According to [wikipedia](https://en.wikipedia.org/wiki/Cache_%28computing%29), a cache is a hardware or software component that stores data so that future requests for that data can be served faster.

#### Benefits:

- Faster access of data in O(1)
- Computation complexity once for the first time

#### Types:

- Memory cache
- Database cache
- Disk cache, etc

In this post, we’ll go through the steps to create a in memory cache in java.

To create a cache, we can simply use a map / dictionary data structure and we can get the expected result of O(1) for both get and put operation.

But, we can’t store everything in our cache. We have storage and performance limits.

A cache eviction algorithm is a way of deciding which element to evict when the cache is full. To gain optimized benefits there are many algorithms for different use cases.

- Least Recently Used (LRU)
- Least Frequently Used (LFU)
- First In First Out (FIFO)
- Last In First Out (LIFO) etc.

> There are only two hard things in computer science, Cache invalidation and naming things. — Phil Karlton

In our design, we will use

- *HashMap* to get and put data in O(1)
- Doubly linked list

![](https://cdn-images-1.medium.com/max/800/1*enbSTt8WK0VFPxMCu9nJHw.png)Cache implementation strategy
We are using doubly linked list to determine which key to delete and have the benefit of adding and deleting keys in O(1).

Initially we will declare a model to store our key-value pair, hit count and reference node to point previous and next node.
{% codeblock lang:java %}
public class CacheItem<K, V> {
    private K key;    
    private V value;    
    private int hitCount = 0; // LFU require this    
    private CacheItem prev, next;    
    
    public CacheItem(K key, V value) {        
        this.value = value;        		
        this.key = key;    
    }        
    public void incrementHitCount() {        
        this.hitCount++;    
    }    
    // getter / setter
}
{% endcodeblock %}


Now, we can outline our cache class with basic functionalities of get, put, delete and size.
{% codeblock lang:java %}
public class Cache<K, V> {
    private Map<K, CacheItem> map;
    private CacheItem first, last;
    private int size;
    private final int CAPACITY;
    private int hitCount = 0;
    private int missCount = 0;
    public Cache(int capacity) {
        CAPACITY = capacity;  // 
        map = new HashMap<>(CAPACITY);
    }
    
    public void put(K key, V value) {}

    public V get(K key) {}

    public void delete(K key) {
        deleteNode(map.get(key));
    }

    public int size() {
        return size;
    }
    public int getHitCount() {
        return hitCount;
    }

    public int getMissCount() {
        return missCount;
    }
}
{% endcodeblock %}


We also included hit count and miss count to determine the performance of our cache.

Hit ratio = total hit / (total hit + total miss)

For our put method

1. If key already exists, update the value
2. Otherwise, If size exceeds capacity
3. Delete existing node using appropriate strategy
4. Add new node in the top
{% codeblock lang:java %}
public void put(K key, V value) {
    CacheItem node = new CacheItem(key, value);
    if(map.containsKey(key) == false) {
        if(size() >= CAPACITY) {
            deleteNode(first);        
        }        
        addNodeToLast(node);    
    }    
    map.put(key, node);
}
{% endcodeblock %}


In our delete method

- Remove node from map
- Delete all references associated with that node.
{% codeblock lang:java %}
private void deleteNode(CacheItem node) {    
    if(node == null) {        
        return;    
    }    
    if(last == node) {        
        last = node.getPrev();    
    }    
    if(first == node) {        
        first = node.getNext();    
    }    
    map.remove(node.getKey());    
    node = null; // Optional, collected by GC    
    size--;
}
{% endcodeblock %}


We can add node in the top. The last pointer will reference to the last inserted node.
{% codeblock lang:java %}
private void addNodeToLast(CacheItem node) {
    if(last != null) {
        last.setNext(node);
        node.setPrev(last);
    }

    last = node;
    if(first == null) {
        first = node;
    }
    size++;
}
{% endcodeblock %}


In our get method,

- Get data from map
- Increment the counter for that item. ( Useful for lease frequently used )
- Reorder the linked list.
{% codeblock lang:java %}
public V get(K key) {
    if(map.containsKey(key) == false) {
        return null;
    }
    CacheItem node = (CacheItem) map.get(key);
    node.incrementHitCount();
    reorder(node);
    return (V) node.getValue();
}
{% endcodeblock %}


Reorder is the key for this implementation. For different algorithm this reorder to delete method will change.

#### Least Recently Used (LRU)

Delete candidate is the oldest used entry.

- The latest accessed node will be at the last end along with newly added items. In this way, we can delete from the first easily.
{% codeblock lang:java %}
private void reorder(CacheItem node) {
    if(last == node) {
        return;
    }
    if(first == node) {
        first = node.getNext();
    } else {
        node.getPrev().setNext(node.getNext());
    }
    last.setNext(node);
    node.setPrev(last);
    node.setNext(null);
    last = node;
}
{% endcodeblock %}


#### Least Frequently Used (LFU)

Delete candidate is the least accessed entry.

We have to sort items based on the frequency the nodes being accessed.

To avoid getting deleted, for each accessed items needs to reach top based on their frequency.

- Iterated a loop, which swaps the node if the frequency is greater than it’s next node frequency.
{% codeblock lang:java %}
private void reorder(CacheItem node) {
    if(last == node) {
        return;
    }
    CacheItem nextNode = node.getNext();
    while (nextNode != null) {
        if(nextNode.getHitCount() > node.getHitCount()) {
            break;
        }
        if(first == node) {
            first = nextNode;
        }
        if(node.getPrev() != null) {
            node.getPrev().setNext(nextNode);
        }
        nextNode.setPrev(node.getPrev());
        node.setPrev(nextNode);
        node.setNext(nextNode.getNext());
        if(nextNode.getNext() != null) {
            nextNode.getNext().setPrev(node);
        }
        nextNode.setNext(node);
        nextNode = node.getNext();
    }
    if(node.getNext() == null) {
        last = node;
    }
}
{% endcodeblock %}


This way, there is an edge case where, after adding items that require delete from the start can cause deletion of the high frequency node.

To solve this issue, we need to add nodes pointing our first node for least frequently accessed.
{% codeblock lang:java %}
private void addNodeToFirst(CacheItem node) {
    if(first != null) {
        node.setNext(first);
        first.setPrev(node);
    }
    first = node;

    if(last == null) {
        last = node;
    }
    size++;
}
{% endcodeblock %}


#### First In First Out (FIFO)

Delete candidate is the first node.

- To add node we will use *addNodeToLast*
- Reordering is not required.
- Delete node from first

#### Last In First Out (LIFO)

Delete candidate is the last node.

- To add node we will use *addNodeToLast*
- Reordering is not required.
- Delete node from last

### Questions:

1. We can use array /ArrayList to solve the same issue with simplified coding.

Yes, it would be simpler. The aspects for not using,

- ArrayList has complexity of insertion and deletion of O(n) in worst case
- To search from there list, the worst case complexity would be O(n)

2. We could have used the LinkedList instead of ArrayList.

Once again, yes. LinkedList had the benefit for adding and removing node at O(1). The aspects for not using,

- We will still face the searching complexity of O(n)

3. For LFU, we are having a complexity of O(n) in reordering method.

Yes, unfortunately.

- I’m reordering here after each get request, which increases the complexity for each get. For post heavy system, this approach will benefit.
- It can be done at the moment of deletion. This way the complexity will increase at the time of putting data. For get heavy system, this approach will benefit more.

I have came up a solution with this so far. Please suggest any better idea.

4. We are using HashMap. It can have the issue of hash collision. Then the performance will not be O(1) as mentioned.

Absolutely.

- It’s always recommended to use better hash function that cause less collision.
- Make sure our load factor remains 0.75 or less.
- If the load factor is more than default or provided one, from Java 8 HashMap automatically increase the size and rehash the full structure.
- From Java 8, the collision also being handled by using balanced tree to obtain O(log n) performance.

I’ve written a post with implementation and explanation [HashMap Implementation for Java](https://medium.com/swlh/hashmap-implementation-for-java-90a5f58d4a5b).

5. Is this implementation thread safe?

No. To be thread safe

- We can use *ConcurrentHashMap* instead of *HashMap*
- Use synchronized block

6. How to add time based eviction strategy to auto delete items?

No. To implement this, we need to run a timer thread to define which items are eligible for eviction and perform deletion operation.
