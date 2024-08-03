---
title: Java集合框架
date: 2023-07-13
update: 2023-11-16
tags:
categories:
  - Java
  - 类库
keywords: Java,Collection,Map,Java集合体系
description: 关于Java集合框架的Collection、Map、J.U.C包。
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_07_18_52.png
---



# Java Collections Framework

## 层次结构

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_07_18_52.png" style="zoom: 50%;" />



<table>
    <tr>    	
        <td><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_10_21_48" alt="集合的类继承树" style="zoom:80%;" /></td>
        <td><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_09_21_07.png" alt="image-20230709210741031" style="zoom:70%;" /></td>
    </tr>
</table>

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_09_11_16.png" alt="Snipaste_2023-07-09_11-15-28" style="zoom:60%;" />



## Collection 接口

* java.util.Colection

* View Collections(视图集合)
  本身不存储元素，底层依赖支持集合(backing collection)来存储实际元素
  对支持集合的任何改变在view collection中可见，对view collection的任何改变会写入支持集合
  `Collections.checkedCollection()`
  `Collections.synchronizedCollection()`
  `Collections.unmodifiableCollection()`
  `List.subList()`
  `Map.entrySet()`
  `Map.keySet()`
  `Map.values()`

* Unmodifiable Collections(不可修改的集合)
  调用可变(mutator)方法会抛出UnsupportedOperationException异常
  不可修改集合不一定是不可变的(例如包含可变元素的不可修改集合)
  不允许null元素
  value-based(equal实例可互换、不应该用于同步)
  `Collections.unmodifiableCollection()`
  `Collections.unmodifiableList()`
  `Collections.unmodifiableSet()`
  `Collections.unmodifiableMap()`
  应用场景：数据集较少且不变



### List 接口

* List是一种有序集合(也被称为序列(sequence))
  可以通过整数**索引访问**元素

* 创建Unmodifiable Lists(不可修改的列表)的静态工厂方法
  `List.of(E...):List<E>`
  `List.copyOf(Collection<? extends E>):List<E>`

* 特点：**元素有序、可重复**

| List接口的实现       | 底层结构                                                     | 同步策略(线程安全=sync)                                      | null元素 | 评价                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| ~~List~~             |                                                              |                                                              |          | com.sun.tools.javac.util.List，不算支持的API                 |
| Vector               | List接口的动态数组(可调整大小)实现                           | sync                                                         | Y        | 如果不需要线程安全的实现，建议使用 ArrayList 替代 Vector     |
| ArrayList            | List接口的动态数组实现<br />resize为均摊常量时间复杂度       | unsync<br />`List list = Collections.synchronizedList(new ArrayList(...));` | Y        | 此类大约等价于Vector，除了此类是不同步的<br />增删操作性能还是不差的，调用数组拷贝复制方法(navite方法) |
| LinkedList           | List和Deque接口的双向链表实现                                | unsync<br />`List list = Collections.synchronizedList(new LinkedList(...));` | Y        |                                                              |
| ~~Stack~~            | 继承Vector                                                   | sync                                                         | Y        | 更完整且一致的 LIFO 栈操作被Deque接口及其实现提供，应优先使用Deque接口<br />` Deque<Integer> stack = new ArrayDeque<Integer>();` |
| CopyOnWriteArrayList | ArrayList的线程安全的变体，所有变更(mutative)操作通过生产(making)底层数组的新副本实现 | sync(不是严格的线程安全，弱一致性)                           | Y        | 缺点：CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性(存在一小段修改操作未完成的时间窗口，其他线程可能会看到旧的数据)<br/>CopyOnWriteArrayList 适用于读多写少的场景，Vector会造成线程阻塞 |



### Set 接口

* Set是一种包含不重复元素的集合
* 创建Unmodifiable Set(不可修改的Set)的静态工厂方法
  `Set.of(E...):Set<E>` 
  `Set.copyOf(Collection<? extends E>):Set<E>`

* 特点：**元素无序、不可重复**

| Set接口的实现 | 底层结构                                                     | 线程是否安全 | 元素null | 评价 |
| ------------- | ------------------------------------------------------------ | ------------ | -------- | ---- |
| TreeSet       | 底层实际上是一个 TreeMap 实例 (红黑树）<br/>可以实现排序的功能 | 非同步       | N        |      |
| HashSet       | 底层数据结构是哈希表 (是一个元素为链表的数组) + 红黑树<br/>实际上就是封装了 HashMap | 非同步       | Y        |      |
| LinkedHashSet | 底层数据结构由哈希表 (是一个元素为链表的数组) 和双向链表组成。<br/>父类是 HashSet<br/>实际上就是 LinkHashMap | 非同步       | Y        |      |



### Queue 接口

* Queue是一种在处理元素之前保存元素的集合
  方法(insertion、extraction、inspection操作)存在两种形式：一种操作失败抛出异常、另一种操作失败返回特殊值(null或false)

  |                                | 抛出异常  | 返回特殊值(推荐)                                             | 释义                                                   |
  | ------------------------------ | --------- | ------------------------------------------------------------ | ------------------------------------------------------ |
  | Insert                         | add(e)    | offer(e)<br/>被设计用于插入失败是正常的在固定容量(有界)的队列 | 如果可能插入一个元素，否则返回false或抛出unchecked异常 |
  | Remove(仅在队列为空时行为不同) | remove()  | poll()                                                       | 删除并返回队列的头节点                                 |
  | Examine                        | element() | peek()                                                       | 返回但不删除队列的头节点                               |

* Queue接口没有定义常见于并发编程中的阻塞队列方法(等待元素出现或者空间变得可用)，这些方法定义在了java.util.concurrent.BlockingQueue接口

* 通常不允许null元素的插入。即使在允许的实现中，null也不应该被插入队列，因为null也被用作poll方法的一个特殊的返回值，表示队列不包含任何元素

| Queue接口的实现 | 底层结构                      | 线程是否安全                                                 | 元素null | 评价                                                         |
| --------------- | ----------------------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| PriorityQueue   | 基于优先堆的无界优先队列      | unsync<br />`java.util.concurrent.PriorityBlockingQueue `    | N        |                                                              |
| LinkedList      | List和Deque接口的双向链表实现 | unsync<br />`List list = Collections.synchronizedList(new LinkedList(...));` | Y        |                                                              |
| ArrayDeque      | Deque接口的动态数组实现       | unsync                                                       | N        | 当**用作stack**时，该类可能比Stack更快<br />当**用作queue**时，该类可能比LinkList更快 |



#### Deque 接口

* Queue是一种支持在两端插入和删除元素的线性集合，deque是double ended queue的缩写(发音为"deck")
  方法(insertion、extraction、inspection操作)存在两种形式：一种操作失败抛出异常、另一种操作失败返回特殊值(null或false)

|                                | 第一个元素(头部) |                                                              | 最后一个元素(尾部) |                                                              |
| ------------------------------ | ---------------- | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
|                                | 抛出异常         | 返回特殊值(推荐)                                             | 抛出异常           | 返回特殊值(推荐)                                             |
| Insert                         | addFirst(e)      | offerFirst(e)<br />被设计用于插入失败是正常的在固定容量(有界)的队列 | addLast(e)         | offerLast(e)<br />被设计用于插入失败是正常的在固定容量(有界)的队列 |
| Remove(仅在队列为空时行为不同) | removeFirst()    | pollFirst()                                                  | removeLast()       | pollLast()                                                   |
| Examine                        | getFirst()       | peekFirst()                                                  | getLast()          | peekLast()                                                   |

* deque被用作queue

| Queue方法 | 等价的Deque方法 |
| --------- | --------------- |
| add(e)    | addLast(e)      |
| offer(e)  | offerLast(e)    |
| remove()  | removeFirst()   |
| poll()    | pollFirst()     |
| element() | getFirst()      |
| peek()    | peekFirst()     |

* deque被用作stack。应优先使用deque接口而不是旧版的Stack类

| Stack方法 | 等价的Deque方法 |
| --------- | --------------- |
| push(e)   | addFirst(e)     |
| pop()     | removeFirst()   |
| peek()    | getFirst()      |

* 通常没有容量限制
* 通常不允许null元素的插入。即使在允许的实现中，null也不应该被插入双端队列，因为null也被用作各种方法的一个特殊的返回值，表示双端队列不包含任何元素

| Deque接口的实现 | 底层结构                      | 线程是否安全                                                 | 元素null | 评价                                                         |
| --------------- | ----------------------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| LinkedList      | List和Deque接口的双向链表实现 | unsync<br />`List list = Collections.synchronizedList(new LinkedList(...));` | Y        |                                                              |
| ArrayDeque      | Deque接口的动态数组实现       | unsync                                                       | N        | 当**用作stack**时，该类可能比Stack更快<br />当**用作queue**时，该类可能比LinkList更快(因为数组相比链表没有频繁的内存申请与释放操作) |



## Map 接口

* java.util.Map
  Map对象用于映射keys到values。不能包含重复的keys；每个key映射到至少一个value
* collection view (集合视图)
  本身不存储元素，底层依赖支持集合(backing collection)来存储实际元素
  对支持集合的任何改变在view collection中可见，对view collection的任何改变会写入支持集合
  `Collections.checkedCollection()`
  `Collections.synchronizedCollection()`
  `Collections.unmodifiableCollection()`
  `List的subList()`1
  `SortedMap的subMap()`
  `SortedMap的headMap()`
  `SortedMap的tailMap()`
  `Map.entrySet()`
  `Map.keySet()`
  `Map.values()`
* Unmodifiable Map(不可修改的Map)
  调用可变(mutator)方法会抛出UnsupportedOperationException异常
  不可修改Map不一定是不可变的(例如包含可变元素的不可修改Map)
  不允许null键和值
  value-based(equal实例可互换、不应该用于同步)
  `Collections.unmodifiableCollection()`
  `Collections.unmodifiableList()`
  `Collections.unmodifiableSet()`
  `Collections.unmodifiableMap()`
  `Map.of()`
  `Map.ofEntries()`
  `Map.copyOf()`
  应用场景：数据集较少且不变

| Map接口的实现   | 底层结构                                                     | 同步策略(线程安全=sync)                                      | 键null元素 | 值null元素 | 评价                                                       |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------- | ---------- | ---------------------------------------------------------- |
| ~~Hashtable~~   | 继承Dictionary类，实现Map接口                                | sync<br />如果需要线程安全且高并发的实现，建议使用**java.util.concurrent.ConcurrentHashMap**代替Hashtable | N          | N          | 如果不需要线程安全的实现，建议使用**HashMap**代替Hashtable |
| Properties      |                                                              |                                                              |            |            |                                                            |
| HashMap         | 基于哈希表的Map接口的实现                                    | unsync<br />`Map m = Collections.synchronizedMap(new HashMap(...));` | Y          | Y          |                                                            |
| LinkedHashMap   | Map接口的哈希表和链表实现，具有可预测的迭代顺序<br />与HashMap不同的是LinkedHashMap维护了贯穿所有键值对的**双向链表**(doublely-linked list)，该链表定义了**迭代顺序**<br /> | unsync<br />Map m = Collections.synchronizedMap(new LinkedHashMap(...)); | Y          | Y          |                                                            |
| WeakHashMap     |                                                              |                                                              |            |            | 应用场景少                                                 |
| IdentityHashMap |                                                              |                                                              |            |            | 应用场景少                                                 |
| TreeMap         | 基于红黑树的NavigableMap接口的实现                           | unsync<br />`SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));` | N          | Y          |                                                            |



## Iterable 接口

java.lang.Iterable 使一个对象成为增强for循环的目标

`iterator():Iterator<T>`

`forEach(Consumer<? super T):void`

示例：

```java
mp.forEach(System.out::println);
```

`spliterator():Spliterator<T>`



## Iterator 接口

java.util.Iterator 集合上的迭代器，Iterator 取代了 Enumeration

`hasNext():boolean`

`next():E`

`remove():void`

`forEachRemaining(Consumer<? super E>):void`



### ListIterator 接口



## Spliterator 接口

主要用于支持Java中的并行流和并行集合操作



# API(不重要)

## Collection 接口

`size():int`

`contains(Object):boolean`

`toArray():Object[]`
`toArray(T[]):T[]`

`add(E):boolean`

`remove(Object):boolean`

`containsAll(Collection<?>):boolean`

`addAll(Collection<? extends E>):boolean`

`removeAll(Collection<?>):boolean`

`removeIf(Predicate<? super E>):boolean`

`retainAll(Collection<?>):boolean`

`clear():void`

`stream():Stream<E>`

`parallelStream():Stream<E>`



### List 接口

`List.of():List<E>` 返回包含0个元素的不可修改List

`List.of(E...):List<E>` 返回包含任意数量元素的不可修改List

`List.copyOf(Collection<? extends E>):List<E>`

`addAll(int,Collection<? extends E>):boolean`

`replaceAll(UnaryOperator<E>):void`

`sort(Comparator<? super E>):void`

`get(int):E`

`set(int, E):E`

`add(int, E):void`

`remove(int):E`

`indexOf(Object):int`

`lastIndexOf(Object):int`

`subList(int,int):List<E>`



#### Vector

`Vector(int,int)` 构造一个空的Vector，伴随指定的初始容量(initialCapacity)、容量增量(capacityIncrement)
`Vector()` 初始容量=10，容量增量0
`Vector(Collection<? extends E>)` 

```java
//增加容量以确保它至少可以容纳最小容量参数指定的元素数量。
//形参：minCapacity – 所需的最小容量
//throws：OutOfMemoryError – 如果 minCapacity 小于零
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = ArraysSupport.newLength(oldCapacity,
            minCapacity - oldCapacity, /* minimum growth */
            capacityIncrement > 0 ? capacityIncrement : oldCapacity /* preferred growth */);  // 每次增长1倍
    return elementData = Arrays.copyOf(elementData, newCapacity);
}
```



##### Stack

`Stack()`

`push(E):E`

`pop():E`

`peek():E`

`empty():boolean`

`search(Object):int`



#### ArrayList

默认容量(DEFAULT_CAPACITY)=10

不同步，如需同步使用 `List list = Collections.synchronizedList(new ArrayList(...));`

`ArrayList(int)` 构造一个空的List，伴随指定的初始容量(initialCapacity)
`ArrayList()`

`trimToSize():void` 修剪此ArrayList实例的容量为list的当前size，用于最小化ArrayList实例的存储空间

`ensureCapacity(int):void` 增加此ArrayList实例的容量，确保至少能容纳的元素数量为参数int minCapacity
用于使容量安全，减少递增重分配的次数

```java
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);  // 每次增长0.5倍(>>1等价与地板除)
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```



#### LinkedList

`LinkedList()`

`LinkedList(Collection<? extends E>)`



#### CopyOnWriteArrayList







HashSet

TreeSet

LinkedHashSet

EnumSet





ConcurrentSkipListSet



### Set 接口

`Set.of():Set<E>` 返回包含0个元素的不可修改Set

`Set.of(E...):Set<E>` 返回包含任意数量元素的不可修改Set

`Set.copyOf(Collection<? extends E>):Set<E>`



#### SortedSet 接口



##### NavigableSet 接口



###### TreeSet



#### HashSet



##### LinkedHashSet



### Queue 接口

`offer(E):boolean`

`remove():E`

`poll():E`

`element():E`

`peek():E`



#### PriorityQueue

`PriorityQueue()` 创建一个根据自然顺序排序元素的PriorityQueue，伴随默认初始容量(11)
[自然排序](./util/Comparator和Comparable.md/#Comparable\<T>)

`PriorityQueue(int)`

`PriorityQueue(Comparator<? super E>)`

`PriorityQueue(int,Comparator<? super E>)`

`PriorityQueue(Collection<? extends E>)`

`PriorityQueue(PriorityQueue<? extends E>)`

`PriorityQueue(SortedSet<? extends E>)`

`comparator():Comparator<? super E>`



#### Deque 接口

`removeFirstOccurrence(Object):boolean` 从双端队列中删除第一次出现的指定元素。如果此调用造成元素被删除，则返回true

`removeLastOccurrence(Object):boolean` 从双端队列中删除最后一次出现的指定元素。如果此调用造成元素被删除，则返回true



##### ArrayDeque

`ArrayDeque()`

`ArrayDeque(int)`

`ArrayDeque(Collection<? extends E>)`



## Map 接口

`size():int`

`containsKey(Object):boolean`

`containsValue(Object):boolean`

`get(Object):V`

`put(K,V):V` 将特定值与特定键相关联在Map中，如果Map已经包含该键的映射，则将旧值替换为指定值

`remove(Object):V`

`putAll(Map<? extends K,? extends V>):void`

`clear():void`

`keySet():Set<K>`

`values():Collection<V>`

`entrySet():Set<Entry<K,V>>`

示例：遍历Map

```java
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {  // 遍历Map
    map2.put(entry.getKey(), entry.getValue());
}
```

`getOrDefault(Object,V):V`

`forEach(BiConsumer<? super K,? super V>):void`

`replaceAll(BiFunction<? super K,? super V,? extends V>):void`

`putIfAbsent(K,V):V`

`remove(Object,Object):boolean`

`replace(K,V,V):boolean`

`replace(K,V):V`

`computeIfAbsent(K,Function<? super K,? extends V>:V`

`computeIfPresent(K,BiFunction<? super K,? super V,? extends V>:V`

`compute(K,BiFunction<? super K,? super V,? extends V>):V`

`merge(K, V,BiFunction<? super V,? super V,? extends V>):V`

`Map.of():Map<K,V>`

`Map.of(K,V):Map<K,V>`

`Map.of(...K10,...V10):Map<K,V>`

`Map.ofEntries(Entry<? extends K, ? extends V>...):Map<K,V>`

`Map.entry(K,V):Entry<K,V>`

`Map.copyOf(Map<? extends K, ? extends V>):Map<K, V>`



### Hashtable

该类实现了一个哈希表(映射键到值)
**为了成功地从哈希表中存储和找回对象，用作键的对象必需实现`hashcode()`和`equals()`方法**
注意：哈希表是开放的，在发生哈希冲突的情况下，单个桶存储多个键值对(entry)，必需按顺序搜索这些键值对

初始容量(initial capacity)，容量是哈希表中桶(buckets)的数量。

负载因子(load factor)，是在容量自动增加之前哈希表允许多么满的指标，通常默认负载因子为0.75。



#### Properties



### HashMap

初始容量(initial capacity)，容量是哈希表中桶(buckets)的数量。
设置：太大则浪费空间，太小则rehash耗时。
如果迭代性能很重要，不要把初始容量设置得太高

负载因子(load factor)，是在容量自动增加之前哈希表允许多么满的指标，通常默认负载因子为0.75。
当哈希表中 $键值对数>(负载因子*当前容量)$ 时，哈希表将被重哈希(重建内部树结构)，使得哈希表的桶数变为大约两倍
设置：太大则减空间成本增时间成本
尽量减少重哈希次数

`HashMap(int,float)` 构造一个空HashMap，伴随指定的初始容量和负载因子

`HashMap(int)` 构造一个空HashMap，伴随指定的初始容量和默认负载因子(0.75)

`HashMap()` 构造一个空HashMap，伴随默认初始容量(16)和默认负载因子(0.75)

`HashMap(Map<? extends K,? extends V)` 构造一个新HashMap伴随与指定Map相同的映射，该HashMap被创建伴随着默认负载因子(0.75)和足够容纳指定Map中的映射的初始容量



#### LinkedHashMap

Map接口的哈希表和链表实现，具有可预测的迭代顺序

与HashMap不同的是LinkedHashMap维护了贯穿所有键值对的双向链表(doublely-linked list)，该链表定义了**迭代顺序**
通常是插入顺序(注意如果键被重新插入Map，插入顺序不受影响)
访问顺序(从最近最少访问到最近访问(east-recently accessed to most-recently))

设置初始容量和负载因子需要注意：与hashMap相比，LinkedHashMap为初始容量选择过高值所带来的惩罚没有HashMap严重，因为**LinkedHashMap的遍历迭代时间不受容量的影响**

`LinkedHashMap(int,float,boolean)` 构造一个空LinkedHashMap实例，伴随指定的初始容量、负载因子和排序模式。 true表示按访问顺序， false表示按插入顺序

`LinkedHashMap(int,float)`

`LinkedHashMap(int)`

`LinkedHashMap()`

`LinkedHashMap(Map<? extends K,? extends V>)`

`LinkedHashMap(int,float,boolean)`



### WeakHashMap



### IdentityHashMap



### SortedMap 接口

SortedMap是提供在其键上进行总排序的一个Map，Map被排序根据键的**自然顺序(实现Comparable接口或传入Comparator)**。

`comparator():Comparator<? super K>`

`subMap(K, K):SortedMap<K,V>` [)

`headMap(K):SortedMap<K,V>`

`tailMap(K):SortedMao<K,V>`

`firstKey():K`

`lastKey():K`



#### NavigableMap 接口

NavigableMap继承SortedMap接口，扩展了导航方法(返回给定搜索目标的最近匹配)，这些方法被设计为了定位(locating)键值对

`lowerEntry(K):Entry<K,V>` 小于给定键的键关联的Map.Entry对象，如果不存在这样的键则返回null

`lowerKey(K):K` 小于给定键的键关联的键，如果不存在这样的键则返回null

`floorEntry(K):Entry<K,V>` 小于或等于给定键的键关联的Map.Entry对象，如果不存在这样的键则返回null

`floorKey(K):K` 小于或等于给定键的键关联的键，如果不存在这样的键则返回null

`ceilingEntry(K):Entry<K,V>` 大于或等于给定键的键关联的Map.Entry对象，如果不存在这样的键则返回null

`ceilingKey(K):K` 大于或等于给定键的键关联的键，如果不存在这样的键则返回null

`higherEntry(K):Entry<K,V>` 大于给定键的键关联的Map.Entry对象，如果不存在这样的键则返回null

`higherKey(K):K` 大于给定键的键关联的键，如果不存在这样的键则返回null

`firstEntry():Entry<K,V>` 返回最小Map(如果存在)，否则返回null

`lastEntry():Entry<K,V>` 返回最大Map(如果存在)，否则返回null

`pollFirstEntry():Entry<K,V>` 删除最小Map(如果存在)，否则返回null

`pollLastEntry():Entry<K,V>` 删除最大Map(如果存在)，否则返回null

`descendingMap():NavigableMap<K,V>` 返回Map视图，反向

`navigableKeySet():NavigableSet<K>`

`descendingKeySet():NavigableSet<K>`

`subMap(K,boolean,K,boolean):NavigableMap<K,V>`

`headMap(K,boolean):NavigableMap<K,V>`

`tailMap(K,boolean):NavigableMap<K,V>`



##### TreeMap

基于红黑树的NavigableMap实现，Map被排序根据其键的自然顺序

###### `get(Object):V` 源码分析

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // 为了性能卸载基于comparator的版本(说明comparable比comparator性能更好)
    if (comparator != null)
        return getEntryUsingComparator(key);  // 优先指定比较器类对象的自定义排序
    Objects.requireNonNull(key);
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);  // 自然排序
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
```



# HashMap源码分析

[数据结构-线性结构-哈希表](../../数据结构/线性结构/哈希表)

TODO: 有空去读HashMap源码

## 成员变量

关键点：Node结构、**默认初始容量为16、负载因子为0.75、树化阈值为8、最小树化容量为64**
树化：哈希表容量大于 64且链表长度大于 8

```java
transient Node<K,V>[] table;
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认初始容量为16
static final float DEFAULT_LOAD_FACTOR = 0.75f;  // 默认的装载因子为0.75
static final int TREEIFY_THRESHOLD = 8;  // 树化阈值，当链表长度超过Treeify_Threshold后链表变红黑树，最坏时间复杂度从O(n)提高到O(logn)
static final int UNTREEIFY_THRESHOLD = 6;  // 链化阈值
static final int MIN_TREEIFY_CAPACITY = 64;  // 可以树化的最小哈希表容量，(应至少为4*TREEIFY_THRESHOLD，以避免扩容和树化之间的冲突)
```



## 构造函数

关键点：未初始化Node节点(延迟创建)

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // 未初始化Node节点(延迟创建)
}

static final int tableSizeFor(int cap) {  // hashmap容量总为2^n幂，为了使hash函数中的异或操作(^)能代替取模
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```



## `put(K,V):V`方法

**put() 方法的逻辑：**

1. 如果 HashMap 未被初始化过，则初始化
2. 对 Key 求 Hash 值，然后再计算下标
3. 如果没有碰撞，直接放入桶中
4. 如果碰撞了 (hash存在且键不存在) ，以链表的方式链接到后面
5. 如果链表长度超过阀值，就把链表转成红黑树
6. 如果链表长度低于6，就把红黑树转回链表
7. 如果节点已经存在 (hash存在且键存在) 就替换旧值
8. **如果桶满了 (哈希表元素总数 > 容量*装载因子) ，就需要扩容2倍后重排**

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;  // resize() 初始化
    if ((p = tab[i = (n - 1) & hash]) == null)  
        tab[i] = newNode(hash, key, value, null);// 当哈希不存在，则new一个该键值对的Node
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;  // 当hash和键一致，则替换
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  // 当hash存在且键不存在且是树化后的节点，则尝试存储键值对
        else {  
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);  // 当hash存在且键不存在且是未树化的链表节点，则链表尾插该键值对，同时若链表插入后超过树化阈值则树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key  // 当hash存在且键存在
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();  // resize() 扩容
    afterNodeInsertion(evict);
    return null;
}
```



## `get(Object):V`方法

```java
final Node<K,V> getNode(Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n, hash; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & (hash = hash(key))]) != null) {  // 通过键对象的哈希函数找到bucket的位置
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
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



## 哈希函数

关键点：调用键的哈希函数且与高位异或

哈希函数的设计：均匀性，减少碰撞是提升性能的关键

```java
static final int hash(Object key) {  
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
// key.hashCode() 返回int散列值。key.hashCode()范围太大，内存数组不够，所以需要进一步计算  // 哈希算法一直在迭代
// 异或操作：加大地位的随机性且地位参杂高位特征，在数组较小时高低位都能参与哈希。需要table容量始终2^n幂
```

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_09_22_26.png" alt="image-20230709222615003" style="zoom:50%;" />

[数据结构-线性结构-哈希表#设计哈希函数](../../../数据结构/线性结构/哈希表#设计哈希函数)

哈希函数有效减少碰撞的方法：**使用 final 对象作为键，并采用合适的 equals() 和 hashCode() 方法**
**为哈希函数添加扰动函数**：促使元素位置分布均匀，减少碰撞机率
不可变性使得能够缓存不同键的 hashcode，能提高获取对象的速度
final 还能防止键改变
所以使用 String常量、Integer常量 (String是final的且重写了equals和hashcode的方法)



## rehash(扩容)

`resize():Node<K,V>` 将哈希表初始化或**变为两倍容量**

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
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

HashMap 扩容的问题：

* 多线程环境下，调整大小会存在条件竞争，容易造成死锁
* rehashing (新的大数组代替小数组) 是一个比较耗时的过程



# 未整理



同步的线程安全的 HashMap

* 可以用`Collections.synchronizedMap(Map<K,V>): Map<K,V>` 方法将 map 包装为线程安全的 Hashmap 实例对象。但是多线程下串行执行，效率较低。
* 使用 ConcurrentHashMap 



1. ConcurrentHashMap
   * get() 方法是非阻塞，无锁的。重写 Node 类，通过 volatile 修饰 next 来实现每次获取都是最新设置的值
   * 在高并发环境下，统计数据 (计算 size..等等) 其实是无意义的，因为在下一时刻 size 值就变化了。





Hashtable：锁住整个 Hashtable 对象，数组 + 链表

ConccurentHashMap：CAS + 部分操作上同步锁、key 和 value 都不能为 null





## Collection

#### Set 接口实现

##### TreeSet (优化遍历选择)

基于红黑树的有序集合 (平衡二叉树的实现)的有序集合

使用：
默认使用**自然排序**(泛型实现的 Comparable 接口的 `compareTo()`方法)
也可以使用**自定义排序**(比较器类对象实现 Comparator<T> 接口的 `caompare()` 方法)

###### 实例方法

`TreeSet(Comparator<? super K>)` 构造函数，使用自定义排序

```java
Set<Customer> mp = new TreeSet<Customer>(new Comparator<Customer>() {
    public int compare(Customer o1, Customer o2) {        
        return 0;
}	}
```

`TreeSet(SortedMap<K, ? extends V>)` 构造函数

`forEach(Consumer): void` 遍历(数据结构的中序遍历)的最好方式，其他方式有`foreach(:)`和 iterator<T>

```java
st.forEach(System.out::println);
```





## ConcurrentHashMap

**线程安全的 HashMap 有哪些？**

* synchronizedMap 在 Colections 工具类中静态包装方法，串行执行
* ConcurrentHashMap 在 J.U.C (java.util.concurrent) 包下，
  * 不允许键值的键 或 值为 Null
  * 支持线程的同步 (线程安全)
  * 比 HashMap 慢



**如何优化 Hashtable？**

* 通过锁细粒度化，将整锁拆解成多个锁进行优化

早期的 ConcurrentHashMap: 通过分段锁 Segment 来实现，即将 Hashtable 逻辑上拆分成多个子数组

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_13_29.png" alt="image-20230710132909685" style="zoom: 50%;" />

当前的 ConcurrentHashMap: CAS + synchronized 使锁更细化

* 首先使用无锁操作 CAS 插入头节点，失败则循环重试
* 若头节点已存在，则尝试获取头节点的同步锁，再进行操作

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_13_31.png" alt="Snipaste_2023-07-10_13-30-46" style="zoom:67%;" />



###### 特有的成员变量

```java
private transient volatile int sizeCtl;  // -1表示正在初始化、-n表示有n-1个活动线程正在调整容量大小、正数或零表示还未被初始化数值为下次初始化的大小。因为有volatile修饰符所以sizeCtl是多线程可见的，可以起到控制的作用。
```



###### put() 方法的逻辑

1. 判断 Node 数组是否初始化，没有则进行初始化操作
2. 通过 hash 定位数组的索引坐标，是否有 Node 节点，如果没有则使用 CAS 进行添加(链表的头节点)，添加失败则进入下次循环。
3. 检查到内部正在扩容，就帮助它一块扩容。
4. 如果 f!=null (头节点不为空)，则使用 synchronized 锁住 f 元素 (链表/红黑二叉树的头节点)
       4.1 如果是 Node (链表结构)则执行链表的添加操作(尾插)。
       4.2如果是 TreeNode (树型结构)则执行树添加操作。
5. 判断链表长度是否超过临界值 (默认为8)，当超过这个值就需要把链表转换为树结构。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();  // ConcurrentHashMap 不允许插入Null键
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {  // 不断的失败重试
        Node<K,V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();  // 若为table空，则初始化
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  // f表示链表后红黑二叉树的头节点
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)  // 正在移动元素
            tab = helpTransfer(tab, f);  // helpTransfer 感兴趣可以去看下
        else if (onlyIfAbsent // check first node without acquiring lock
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;
        else {  // 发生了哈希碰撞
            V oldVal = null;
            synchronized (f) {  // 锁头
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {  // 如果fh是链表头节点
                        binCount = 1;  // 初始化链表计数器
                        for (Node<K,V> e = f;; ++binCount) {  // 遍历
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;  // 如果节点存在，则更新
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);  // 如果不存在，尾插
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {  // 如果头节点是红黑树节点
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                    else if (f instanceof ReservationNode)  // 如果头节点是保留节点，则抛出异常
                        throw new IllegalStateException("Recursive update");
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);  // 链表转化为树结构
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```



###### 问题

`size()` 方法和 `mappingCount()` 方法的异同，多线程下是否能获得 ConcurrentHashMap 准确的大小？

* 通过遍历获取 ConcurrentHashMap 大小，多线程下不准确
* `mappingCount()` 在多线程下能获取准确的map大小



多线程环境下 ConcurrentHashMap 如何进行扩容？

简而言之：使用分段锁的扩容策略





