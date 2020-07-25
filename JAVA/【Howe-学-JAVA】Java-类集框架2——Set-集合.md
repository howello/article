> 类集是Java中的一个重要特性，是Java针对常用数据结构的官方实现，在实际开发中广泛使用。在JDK1.5 之后，为了使类集操作更加安全，对类集框架进行了修改，加入了泛型操作。

## Collection 集合接口
`java.util.Collection` 是单值集合操作的最大的父接口，在该接口中定义了所有的单值数据的处理操作。如下所示

```java
    @Override
    // 获取数据长度
    public int size() {
        return 0;
    }
    @Override
    // 是否为空
    public boolean isEmpty() {
        return false;
    }
    @Override
    // 是否包含
    public boolean contains(Object o) {
        return false;
    }
    @Override
    // 添加
    public boolean add(Object e) {
        return false;
    }
    @Override
    // 移除
    public boolean remove(Object o) {
        return false;
    }
    @Override
    // 清空
    public void clear() {
    }
```
平时使用中基本不会直接使用 Collection 接口，往往会使用其两个子接口 List 和 Set 接口。List 允许重复，而 Set 不允许重复。

## Set 集合
为了与 List 集合有所区分，在进行 Set 接口的设计时要求其内部不允许保存重复元素。

### HashSet 子类
HashSet 是Set 接口常见的一个子类，最大的特点是不允许保存重复元素，并且所有的内容都是采用散列（无序）的方式进行储存。

代码如下：
```java
public static void runHashSet() {
    Set<String> set = new HashSet<>();
    set.add("Four");
    set.add("Five");
    set.add("ONE");
    set.add("TWO");
    set.add("THREE");
    set.add("Four");
    set.add("Five");
    System.out.println(set);
}
```
调用 `add` 方法时，按照1,2,3,4,5 存进去的，那么接下来看输出：
```java
[Five, ONE, Four, TWO, THREE]
```
首先，重复的 4 和 5 只有一个，印证了不保存重复元素。其次，输出不是按照存的时候的顺序来的，那么是存的时候无序存的还是，取得时候无序取得，或者都有？那么我们再多跑几遍试试：
```java
[Five, ONE, Four, TWO, THREE]
[Five, ONE, Four, TWO, THREE]
[Five, ONE, Four, TWO, THREE]
[Five, ONE, Four, TWO, THREE]
```
没错，不管几遍，输出是一样的。所以是存的时候按照无序存进去，取得时候按照顺序取出来的。

### TreeSet 子类
TreeSet 可以针对设置进行排序保存。所有保存的字符串会按照字母大小顺序排列（数据会按照由小到大排列）。
代码如下：
```java
public static void runTreeSet() {
    Set<String> set = new TreeSet<>();
    set.add("D");
    set.add("B");
    set.add("A");
    set.add("C");
    System.out.println(set);
    Set<Integer> setInt = new TreeSet<>();
    setInt.add(1);
    setInt.add(2);
    setInt.add(3);
    setInt.add(4);
    setInt.add(5);
    System.out.println(setInt);
}
```
输出结果为：
```java
[A, B, C, D]
[1, 2, 3, 4, 5]
```

### LinkedHashSet 子类
如果需要 Set 的不重复特性，又不想使用无序排列，那么还有一个子类，那就是 LinkedHashSet 可以很好的做到这两点。

代码：
```java
public static void runLinkedSet() {
    Set<String> set = new LinkedHashSet<>();
    set.add("A");
    set.add("D");
    set.add("C");
    set.add("B");
    System.out.println(set);
}
```
输出结果：
```java
[A, D, C, B]
```


------------

> 相关推荐
>  [【Howe 学 JAVA】Java 类集框架1——List集合](https://neusoft.me/ziyuan/java/2020/05/11/517/ "【Howe 学 JAVA】Java 类集框架1——List集合")


------------
本文永久链接：https://neusoft.me/ziyuan/java/2020/05/11/520/
