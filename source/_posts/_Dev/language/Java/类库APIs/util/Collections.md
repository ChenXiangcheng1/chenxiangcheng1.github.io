# Collections

java.util.Collections 此类仅由对集合进行操作或返回集合的静态方法组成。


`sort(List<T>,Comparator<? super T>):void`

`binarySearch(List<? extends T>,T,Comparator<? super T>):int`

`reverse(List<?>):void`

`shuffle(List<?>,Random):void`

`fill(List<? super T>, T):void`

`copy(List<? super T>,List<? extends T>):void`

`min(Collection<? extends T>, Comparator<? super T>):T`

`max(Collection<? extends T>, Comparator<? super T>):T`

`rotate(List<?>,int):void`

`replaceAll(List<T>, T,T):boolean`

`indexOfSubList(List<?>, List<?>):int`

`lastIndexOfSubList(List<?>,List<?>):int`

`unmodifiableCollection(Collection<? extends T>):Collection<T>`

`synchronizedCollection(Collection<T>):Collection<T>`



`synchronizedMap(Map<K,V>): Map<K,V>` 包装器，返回包装后的 map 为线程安全的 Hashmap 实例对象。多线程下串行执行，效率较低，更推荐使用 ConcurrentHashMap。

 `Collections.emptyList(); `表示返回空 List