# Comparator

`java.util.Comparator` 是一个[函数式接口](../Lambda.md#函数式接口)



## compare

`compare(T, T):int` 第一个参数小于第二个参数返回负整数、等于返回0，大于返回正整数

示例：

```java
String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
Arrays.sort(array, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
```

```java
String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));
```



`Comparator.naturalOrder():Comparator<T>` 返回一个按自然顺序比较的 Comparator，比较 null 时抛出 NullPointerException 
`Comparator.reverseOrder():Comparator<T>` 返回一个 Comparator，其与自然顺序相反



## null支持

`nullsFirst(Comparator<? super T>):Comparator<T>` 返回一个 null 友好的 Comparator，其认为 null 小于非null。如果指定的Comparator 为空，则返回的 Comparator，其认为所有非null相等
`nullsLast(Comparator<? super T>):Comparator<T>` 返回一个 null 友好的 Comparator，其认为 null 大于非null。如果指定的Comparator 为空，则返回的 Comparator，其认为所有非null相等



## 提取比较键

`comparing(Function<? super T,? extends U>,Comparator<? super U>):Comparator<T>` Function函数从类型T中提取排序键(sort key)，并返回使用指定Comparator按该排序键排序的Comparator<T> 

`comparingInt(ToIntFunction<? super T>):Comparator<T>` ToIntFunction函数从类型T中提取int排序key，并返回按该int排序的Comparator<T>

示例：

```java
int[][] intervals = {{1,3},{3,4},{2,3}};
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
```



`thenComparing(Function<? super T,? extends U>, Comparator<? super U>):Comparator<T>` Function函数从类型T中提取排序键(sort key)，并返回使用指定Comparator按该排序键排序的Comparator<T> 

`thenComparingInt(ToIntFunction<? super T>):Comparator<T>` ToIntFunction函数从类型T中提取int排序key，并返回按字典序排序的Comparator<T>



# Comparable\<T>

`java.lang.Comparable`

该排序也被称为类的**自然排序**，compareTo也被称为类的自然排序方法
实现该接口的对象，才可以通过 `Collections.sort` 、 `Arrays.sort` 排序
实现该接口的对象，不需要 Comparator 就可以作为 sorted map、sorted set 中的元素

强烈建议自然排序与 equals 一致

`compareTo(T):int` 大于返回正整数，等于返回0，小于返回负整数

示例：

```java
@Override
public int compareTo(WeightEdge another) {    
    return Integer.compare(x, y);  // better
    // return weight - another.weight;
}
```





