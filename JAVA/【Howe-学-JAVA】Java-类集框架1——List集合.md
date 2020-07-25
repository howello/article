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

## List 集合
在使用 List 开发时，基本都是使用其子类进行实例化，常用的有 ArrayList、 Vector、 LinkedList 等。

### ArrayList 子类
平时使用中ArrayList 子类使用的是最频繁的，使用代码如下：
```java
public static void runArrayList() {
    List<String> list = new ArrayList<>();
    System.out.println("list.isEmpty():" + list.isEmpty());
    System.out.println("list.size():" + list.size());
    list.add("ONE");
    list.add("TWO");
    list.add("THREE");
    list.add("ONE");
    list.add("TWO");
    list.add("THREE");
    list.add("FOUR");
    System.out.println(list);
    System.out.println("list.isEmpty():" + list.isEmpty());
    System.out.println("list.size():" + list.size());
    System.out.println("list.contains(\"FOUR\"):" + list.contains("FOUR"));
    System.out.println("list.get(1):" + list.get(1));
    System.out.println("list.remove(\"TWO\"):" + list.remove("TWO"));
    System.out.println("list.isEmpty():" + list.isEmpty());
    System.out.println("list.size():" + list.size());
    list.forEach(System.out::print);
}
```
> 代码没有什么难点，一看就懂，我说一下最后一句。
> 第一印象， 哇， 好高大上的写法， 那么这究竟是怎样的一种语法呢。
> `system.out::print` 这段代码其实就是`Consumer<T>`接口的一个实现方式啊。
> 就是把你遍历出来的每一个对象都用来去调用System.out（也就是PrintStream类的一个实例）的print方法。
> 这一段摘自（https://blog.csdn.net/qq_36929361/article/details/84926277）

输出结果：
```java
list.isEmpty():true
list.size():0
[ONE, TWO, THREE, ONE, TWO, THREE, FOUR]
list.isEmpty():false
list.size():7
list.contains("FOUR"):true
list.get(1):TWO
list.remove("TWO"):true
list.isEmpty():false
list.size():6
ONETHREEONETWOTHREEFOUR
```

### LinkedList 子类

LinkedList 说白了就是基于链表数据结构实现的 List 集合标准，代码如下：
```java
public static void runLinkedList() {
    List<String> list = new LinkedList<>();
    list.add("ONE");
    list.add("TWO");
    list.add("THREE");
    list.add("TWO");
    list.add("THREE");
    System.out.println(list);
    list.forEach(System.out::println);
}
```
输出：
```java
[ONE, TWO, THREE, TWO, THREE]
ONE
TWO
THREE
TWO
THREE
```

### Vector 子类

Vector 与 ArrayList 的区别就在于 Vector 的操作方法都是 synchronized 同步处理的，而ArrayList 并没有进行同步处理。所以 Vector 类中的方法在多线程访问的时候属于线程安全的，但是性能没有 ArrayList 高，所以在考虑线程高并发的情况下才会去使用 Vector 子类。

使用代码：
```java
public static void runVector() {
    List<String> list = new Vector<>();
    list.add("ONE");
    list.add("TWO");
    list.add("THREE");
    list.add("TWO");
    list.add("THREE");
    System.out.println(list);
}
```
输出：
```java
[ONE, TWO, THREE, TWO, THREE]
```

本文永久链接： https://neusoft.me/ziyuan/java/2020/05/11/517/


<!--more-->
> 废话时间
> 刚说完要坚持，结果就鸽了两天，真想抽自己两巴掌。
> 不更是有一定的原因的，因为服务器挂了，收拾了两天服务器。
> 不过这都是借口，就是懒了。
> 继续加油吧！
