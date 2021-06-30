---
title: HashMap Implementation for Java
date: 2020-08-18 13:18:27
tags:
  - java
  - hashmap

---
# HashMap Implementation for Java

HashMap is a dictionary data structure provided by java. It’s a Map-based collection class that is used to store data in Key & Value pairs. In this article, we’ll be creating our own hashmap implementation.

The benefit of using this data structure is faster data retrieval. It has data access complexity of O(1) in the best case.

![](https://cdn-images-1.medium.com/max/2000/1*X8Wju0TZ-wcqBUucVyGj_w.png)

In layman’s terms, a for each key we get the associated value.

To implement this structure,

1. We need a list to store all the keys

1. Key — Value relationship to get value based on key

We can have a list containing all the key, values and to access we need to search all of it.

But the main point of hash map is to access the keys faster in 0(1) access time.

Here, hashing comes into play. We can hash the key and relate it with the index to retrieve data faster.

Hash comes with a problem too, collision. It is always recommended to use a better hash function that can reduce chances of collision.

![](https://cdn-images-1.medium.com/max/2000/1*KKeNedzjxdkNwljIhTkkuA.png)

Multiple hash can have same hash key. For that reason, there is a bucket or container for each key where all the values are store if collision occurs.

Let’s dive into a basic implementation of our hashmap.

Firstly, we need an array to store all the keys, a bucket model to store all the entry and a wrapper for our key, value pair.
```
    public class MyKeyValueEntry<K, V> {
        private K key;
        private V value;
    
        public MyKeyValueEntry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        // getters & setters
        // hashCode & equals
    }
```
Bucket to store all the key values
```
    public class MyMapBucket {
        private List<MyKeyValueEntry> entries;
    
        public MyMapBucket() {
            if(entries == null) {
                entries = new LinkedList<>();
            }
        }
    
        public List<MyKeyValueEntry> getEntries() {
            return entries;
        }
    
        public void addEntry(MyKeyValueEntry entry) {
            this.entries.add(entry);
        }
    
        public void removeEntry(MyKeyValueEntry entry) {
            this.entries.remove(entry);
        }
    }
```
Lastly, implementation of our hashmap
```
    public class MyHashMap<K, V> {
        private int CAPACITY = 10;
        private MyMapBucket[] bucket;
        private int size = 0;
    
        public MyHashMap() {
            this.bucket = new MyMapBucket[CAPACITY];
        }
        private int getHash(K key) {
            return (key.hashCode() & 0xfffffff) % CAPACITY;
        }
    
        private MyKeyValueEntry getEntry(K key) {
            int hash = getHash(key);
            for (int i = 0; i < bucket[hash].getEntries().size(); i++) {
                MyKeyValueEntry myKeyValueEntry = bucket[hash].getEntries().get(i);
                if(myKeyValueEntry.getKey().equals(key)) {
                    return myKeyValueEntry;
                }
            }
            return null;
        }

        public void put(K key, V value) {
            if(containsKey(key)) {
                MyKeyValueEntry entry = getEntry(key);
                entry.setValue(value);
            } else {
                int hash = getHash(key);
                if(bucket[hash] == null) {
                    bucket[hash] = new MyMapBucket();
                }
                bucket[hash].addEntry(new MyKeyValueEntry<>(key, value));
                size++;
            }
        }
    
        public V get(K key) {
            return containsKey(key) ? (V) getEntry(key).getValue() : null;
        }
    
        public boolean containsKey(K key) {
            int hash = getHash(key);
            return !(Objects.*isNull*(bucket[hash]) || Objects.*isNull*(getEntry(key)));
        }
    
        public void delete(K key) {
            if(containsKey(key)) {
                int hash = getHash(key);
                bucket[hash].removeEntry(getEntry(key));
                size--;
            }
        }
        public int size() {
            return size;
        }
    }
```
**Put into map:**

1. If key already exists, then update value of that key.

1. Otherwise, add an entry to the bucket.

**Get from map:**

1. Check if the key exists, and return data.

**Contains:**

1. Check if the bucket is null

1. If not, then the bucket contains the key.

**Performance:**

It has the performance of O(1) in best case and O(n) in worst case.

**Java Improvement:**

1. From Java version 8, All of the hashing based Map implementations: HashMap, Hashtable, LinkedHashMap, WeakHashMap and ConcurrentHashMap are modified to use an enhanced hashing algorithm for string keys when the capacity of the hash table has ever grown beyond 512 entries. The enhanced hashing implementation uses the murmur3 hashing algorithm along with random hash seeds and index masks. These enhancements mitigate cases where colliding String hash values could result in a performance bottleneck. [Alternative String hashing implementation](https://hg.openjdk.java.net/jdk8/jdk8/jdk/rev/43bd5ee0205e)

1. From Java version 8, once the number of items in a hash bucket grows beyond a certain threshold, that bucket will switch from using a linked list of entries to a balanced tree. In the case of high hash collisions, this will improve worst-case performance from O(n) to O(log n). [Handle Frequent HashMap Collisions with Balanced Trees](https://openjdk.java.net/jeps/180).

**Java Hashmap features:**

* The default initial capacity is 16

    static final int *DEFAULT_INITIAL_CAPACITY *= 1 << 4;

* The load factor used when none specified in constructor.

    static final float *DEFAULT_LOAD_FACTOR *= 0.75f;

* The bin count threshold for using a tree rather than list for a bin. Bins are converted to trees when adding an element to a bin with at least this many nodes.

    static final int *TREEIFY_THRESHOLD *= 8;

* The bin count threshold for untreeifying a (split) bin during a resize operation.

    static final int *UNTREEIFY_THRESHOLD *= 6;

* An instance of HashMap has two parameters that affect its performance: *initial capacity* and *load factor*. 
* The *capacity* is the number of buckets in the hash table, and 
* the initial capacity is simply the capacity at the time the hash table is created. 
The *load factor*s a measure of how full the hash table is allowed to get before its capacity is automatically increased. 
When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is *rehashed* (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.

* As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. 
* Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). 
The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. 
* If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.
