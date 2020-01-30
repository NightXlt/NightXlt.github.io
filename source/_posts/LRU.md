title: LRU
date: 2019-1-7
tags: [Leetcode,字节跳动]
categories: algorithm
description: Design and implement a data structure for Least Recently Used (LRU) cache.
---
## Problem description
  ### Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and put.get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.put(key, value) - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.Could you do both operations in O(1) time complexity?
 ## Examples
``` java
Example 1:
LRUCache cache = new LRUCache( 2 /* capacity */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2
cache.get(2);       // returns -1 (not found)
cache.put(4, 4);    // evicts key 1
cache.get(1);       // returns -1 (not found)
cache.get(3);       // returns 3
cache.get(4);       // returns 4
```

## Solution
　　一开始选取HashMap，但因为HashMap无序只能放弃。最终选取了LinkedHashMap,但在使用过程中，无法remove指定index元素。差点手写。幸而搜到了LInkedHashMap的一个构造函数可以实现LRU。还需要重写removeEldestEntry方法，LInkedHashMap在添加元素后会调用removeEldestEntry方法检查是否需要移出最久未使用元素。
 ```java
 public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {//accessOrder：true采取LRU淘汰算法
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
 ```

## Code

```javaclass LRUCache {

     LinkedHashMap mCacheMap;
    int mCapacity;
    public LRUCache(int capacity) {
        mCapacity = capacity;
        mCacheMap = new LinkedHashMap(capacity, 0.75F, true){
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                if (mCapacity + 1 == mCacheMap.size()) {
                    return true;
                }
                return false;
            }
        };
    }

    public int get(int key) {
        if (mCacheMap.containsKey(key)) {
            return (int) mCacheMap.get(key);
        }
        return -1;
    }

    public void put(int key, int value) {
        mCacheMap.put(key, value);
    }
}
```