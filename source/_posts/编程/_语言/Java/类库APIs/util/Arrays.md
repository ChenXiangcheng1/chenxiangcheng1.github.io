# Arrays

该类包含各种操作数组的方法(排序sort和搜索binarySearch)



## sort

### 引用类型

`Arrays.sort(T[],int,int,Comparator<? super T>):void` 对对象数组的 [fromIndex,toIndex) 进行稳定排序，所有元素通过 Comparator 进行比较 ，排序算法改编自TimSort

示例1：

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

示例2：对 int[] 数组进行降序排序(因为Java的基本类型不能应用泛型T搞得很麻烦)

(最佳实践：直接正序排序，从 length-1 to 0 操作)

```java
// 不会装箱，原地排序
int[] arr = ...;
Arrays.sort(arr);
IntStream.range(0, arr.length/2)
    		.forEach(i -> {
            	int tmp = arr[i];
                arr[i] = arr[arr.length-i-1];
                arr[arr.length-i-1] = tmp;
            });
```

```java
// 不会装箱，非原地
int[] arr = ...;
Arrays.sort(arr);
int[] reversed = IntStream.range(0, arr.length)
                          .map(i -> arr[arr.length-i-1])
                          .toArray();
```

```java
// 会装箱，非原地
int[] iarray = {1,3,2};
int[] desiarray = Arrays.stream(iarray)
                        .boxed()
                        .sorted(Comparator.reverseOrder())
                        .mapToInt(Integer::intValue)
                        .toArray();
```

示例3：对 int\[]\[] 数组进行排序(按照第一列的元素大小进行升序排序)

```java
int[][] intervals = {{1,3},{3,4},{2,3}};
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
```

示例3：对 int\[]\[] 数组进行排序(按照第一列的元素大小进行升序排序，如果第一列相同，则比较第二列)

```java
int[][] intervals = {{1,3},{3,4},{2,3}};
Arrays.sort(intervals, (o1, o2) -> {
    if (o1[0] == o2[0])
        return Integer.compare(o1[1], o2[1]);
    return Integer.compare(o1[0], o2[0]);
});
```



### 基础类型

基础类型支持 int、long
**基础类型不支持指定 Comparator 比较器**

`Arrays.sort(int[],int,int):void` 对该数组的 [fromIndex,toIndex) 进行升序排序，排序算法为 Dual Pivot Quicksort(改编自TimSort)



`Arrays.parallelSort(int[],int,int):void` 对该数组的 [fromIndex,toIndex) 进行升序排序，排序算法为 Dual Pivot Quicksort(改编自TimSort)

`Arrays.parallelSort(T[],int,int,Comarator<? super T>):void`



`Arrays.parallelPrefix(T[],int,int,BinaryOperator<T>):void` 对数组的 [fromIndex,toIndex) 执行并行前缀计算



## binarySearch

`Arrays.binarySearch(int[],int,int,int):int` 对数组的 [fromIndex,toIndex) 进行二分搜索，在进行调用之前，必需对该范围进行排序

`Arrays.binarySearch(T[],int,int,T,Comparator<? super T>):int` 对数组的 [fromIndex,toIndex) 进行二分搜索，在进行调用之前，必需对该范围进行排序



`Arrays.equals(int[],int,int,int[],int,int):boolean` 

`Arrays.euqals(T[],int,int,T[],int,int,Comparator<? superT>):boolean`



`Arrays.compare(int[],int,int,int[],int,int):int` 小于返回负数，等于返回0，大于返回正数

`Arrays.compare(T[],int,int,T[],int,int,Comparator<? super T>):int` 小于返回负数，等于返回0，大于返回正数



`Arrays.setAll(int[],IntUnaryOperator):void` 用generator函数计算每个元素

`Arrays.setAll(T[],IntFunction<? extends T>):void` 用generator函数计算每个元素



`Arrays.parallelSetAll(int[],IntUnaryOperator):void` 用generator函数计算每个元素

`Arrays.parallelSetAll(T[],IntUnaryOperator):void` 用generator函数计算每个元素



## copyOf

`Arrays.copyOf(int[],int):int[]` 底层调用System.arraycopy()

`Arrays.copyOf(T[],int):T[]` 复制指定的数组，截断或用nulls填充(如果有必要)以使副本具有指定长度。int表示返回的副本的长度
底层调用System.arraycopy()

`Arrays.copyOfRange(int[],int,int):int[]` 底层调用System.arraycopy()

`Arrays.copyOfRange(T[],int,int):T[]` 底层调用System.arraycopy()



## fill

`Arrays.fill(int[],int,int,int):void` 将指定的int值分配给一维数组 [fromIndex,toIndex) 的每个元素，填充数组

`Arrays.fill(Object[],int,int,Object):void` 将指定的对象引用(Object reference)分配给一维数组 [fromIndex,toIndex) 的每个元素，填充数组



## toString() 打印数组

`Arrays.toString(int[]):String` 返回由数组内容构成的字符串，例如 "[1,null,3]"

`Arrays.deepToString(Object[]):String` 返回表示指定数组的深层内容的字符串。将多维数组转换为字符串，自引用为"[...]"，null数组为 "null"



## asList

`Arrays.asList(T...):List<T>` 返回固定大小的ArrayList由指定数组支持。对数组的改变在返回的List中可见，对List的改变在数组中可见



## Stream

[返回以**指定数组** (int[]、T[]) 作为源的顺序流 (IntStream、Stream<T>)](./Stream.md#Arrays)

