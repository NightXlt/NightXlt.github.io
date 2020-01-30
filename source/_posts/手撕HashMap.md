title: 手撕HashMap
date: 2019-1-29
tags: [Java]
categories: JavaAPI
description: 真香
---
## 数据结构
### 位桶 + 链表 + 红黑树

```java
//Node是单向链表，它实现了Map.Entry接口
static class Node<k,v> implements Map.Entry<k,v> {
    final int hash;
    final K key;
    V value;
    Node<k,v> next;
    //构造函数Hash值 键 值 下一个节点
    Node(int hash, K key, V value, Node<k,v> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
 
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + = + value; }
 
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
 
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //判断两个node是否相等,若key和value都相等，返回true。可以与自身比较为true
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<!--?,?--> e = (Map.Entry<!--?,?-->)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

```java
transient Node<k,v>[] table;//存储位桶的数组。
```

```java
//红黑树
static final class TreeNode<k,v> extends LinkedHashMap.LinkedHashMapEntry<K,V> {//LinkedHashMap.LinkedHashMapEntry<K,V>  继承 Node<K,V>
    TreeNode<k,v> parent;  // 父节点
    TreeNode<k,v> left; //左子树
    TreeNode<k,v> right;//右子树
    TreeNode<k,v> prev;    // needed to unlink next upon deletion
    boolean red;    //颜色属性
    TreeNode(int hash, K key, V val, Node<k,v> next) {
        super(hash, key, val, next);
    }
 
    //返回当前节点的根节点
    final TreeNode<k,v> root() {
        for (TreeNode<k,v> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
```
## 工作机制
![hashMap](/images/HashMap.png)
Bucket为Hash值相同的元素集合。通过Hash函数计算出元素放置位置。若无冲突直接插入。有冲突则当桶的上的结点数大于TREEIFY_THRESHOLD时且table中结点数目大于MIN_TREEIFY_CAPACITY会转成红黑树。当桶上的结点数小于UNTREEIFY_THRESHOLD时树转链表。再插入结点。 数据结构转换时，是以table中hash函数所在的第一个节点开始转换。
## 成员变量
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 填充因子
    final float loadFactor;
}
```
## 构造函数
```java
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不能小于0，否则报错
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    // 初始容量不能大于最大值，否则为最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 填充因子不能小于或等于0，不能为非数字
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    // 初始化填充因子                                        
    this.loadFactor = loadFactor;
    // 初始化threshold大小
    this.threshold = tableSizeFor(initialCapacity);    
}
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
 
//构造函数3
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
 
//构造函数4用m的元素初始化散列映射
public HashMap(Map<!--? extends K, ? extends V--> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

```

tableSizeFor(initialCapacity)返回大于等于initialCapacity的最小的二次幂数值。如capacity为9.返回16.
```java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;//>>> 操作符表示无符号右移，高位取0。
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
 ## 重要成员函数
 ```java
 static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
//获取key对应所在tab的位置
i = (n - 1) & hash（key）
 ```

```java
public V get(Object key) {
    Node<k,v> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
 
 
final Node<k,v> getNode(int hash, Object key) {
    Node<k,v>[] tab; Node<k,v> first, e; int n; K k;
    //hash & (length-1)得到对象的保存位
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            //如果第一个节点是TreeNode,说明采用的是数组+红黑树结构处理冲突
            //遍历红黑树，得到节点值
            if (first instanceof TreeNode)
                return ((TreeNode<k,v>)first).getTreeNode(hash, key);
            //链表结构处理
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
```java
   public V put(K key, V value) {
       return putVal(hash(key), key, value, false, true);
   }
 /* putVal
 1判断键值对数组tab[]是否为空或为null，否则resize()；

2根据键值key计算hash值得到插入的数组索引i，如果tab[i]==null，直接新建节点添加，否则转入3

3判断当前数组中处理hash冲突的方式为链表还是红黑树(check第一个节点类型即可),分别处理。
 */
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
       Node<k,v>[] tab; Node<k,v> p; int n, i;
       //如果tab为空或长度为0，则分配内存resize()
       if ((tab = table) == null || (n = tab.length) == 0)
           n = (tab = resize()).length;
       //(n - 1) & hash找到put位置，如果为空,则直接put
       if ((p = tab[i = (n - 1) & hash]) == null)
           tab[i] = newNode(hash, key, value, null);
       else {
           Node<k,v> e; K k;
           //第一节节点hash值同，且key值与插入key相同
           if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
               e = p;
           else if (p instanceof TreeNode)//属于红黑树处理冲突
               e = ((TreeNode<k,v>)p).putTreeVal(this, tab, hash, key, value);
           else {
               //链表处理冲突
               for (int binCount = 0; ; ++binCount) {
                   //p第一次指向表头,以后依次后移
                   if ((e = p.next) == null) {
                       //e为空，表示已到表尾也没有找到key值相同节点，则新建节点
                       p.next = newNode(hash, key, value, null);
                       //新增节点后如果节点个数到达阈值，则将链表转换为红黑树
                       if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                           treeifyBin(tab, hash);
                       break;
                   }
                   //容许null==null
                   if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                       break;
                   p = e;//更新p指向下一个节点
               }
           }
           //更新hash值和key值均相同的节点Value值
           if (e != null) { // existing mapping for key
               V oldValue = e.value;
               if (!onlyIfAbsent || oldValue == null)
                   e.value = value;
               afterNodeAccess(e);
               return oldValue;
           }
       }
       ++modCount;
       if (++size > threshold)
           resize();
       afterNodeInsertion(evict);
       return null;
   }
```
resize分为两步:(oldCap：原数组长度， newCap：新数组长度)
1. 将newTable扩充为两倍的table
2. 重新计算index，把节点再放到新的bucket中
之所以重新计算index，是因为数组长度变化可能导致元素的所在index变化
若仅有一个节点，则index = e.hash & (newCap - 1);
若多于一个节点，以链表为例，
获取每个bucket中的节点e, 判断（e.hash & oldCap) == 0，是的话则放置在原索引位置 j ，否则放置到 j + oldCap;

```java
final Node<K,V>[] resize() {
    // 当前table保存
    Node<K,V>[] oldTab = table;
    // 保存table大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 保存当前阈值 
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 之前table大小大于0
    if (oldCap > 0) {
        // 之前table大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 阈值为最大整形
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量翻倍，使用左移，效率更高
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
            oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值翻倍
            newThr = oldThr << 1; // double threshold
    }
    // 之前阈值大于0
    else if (oldThr > 0)
        newCap = oldThr;
    // oldCap = 0并且oldThr = 0，使用缺省值（如使用HashMap()构造函数，之后再插入一个元素会调用resize函数，会进入这一步）
    else {           
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 新阈值为0
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 初始化table
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 之前的table已经初始化过
    if (oldTab != null) {
        // 复制元素，重新进行hash
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 将同一桶中的元素根据(e.hash & oldCap)是否为0进行分割，分成两个不同的链表，完成rehash
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```
