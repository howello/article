> Collection 接口中的 `toArray()` 方法可以将集合保存的数据转为对象数组返回，用户可以利用数据循环的方式获取内容。但是此类方式由于性能不高并不是集合输出的首选方案。在类集框架中对于集合的输出提供了 4 种方式： Iterator、 ListIterator、 Enumeration、 foreach。

## Iterator 迭代输出
在 Iterator 接口中提供有 `remove()` 方法，Collection 中也提供了 `remove()` 方法。但是在使用中尽量不要使用 `Collection.remove()` 方法，如果使用之后，会出现异常信息。
代码如下：
```java
public static void runIterator() {
    Set<String> set = new HashSet<>();
    set.add("ONE");
    set.add("TWO");
    set.add("THREE");
    set.add("FOUR");
    set.add("FIVE");
    set.add("SIX");
    Iterator<String> iter = set.iterator();
    while (iter.hasNext()) {
        String str = iter.next();
        if (("ONE".equals(str))) {
            iter.remove();
        } else {
            System.out.print(str + " - ");
        }
    }
}
```
输出如下：
```java
FIVE - SIX - FOUR - TWO - THREE -
```

## ListIterator 双向迭代输出
ListIterator 比 Iterator 多一点，就是可以从后往前迭代，也可以从前往后迭代。
代码如下：
```java
public static void runListIterator() {
    List<String> list = new ArrayList<>();
    list.add("ONE");
    list.add("TWO");
    list.add("THREE");
    ListIterator<String> lIter = list.listIterator();
    System.out.println("From front to back:".toUpperCase());
    System.out.print("\t");
    while (lIter.hasNext()) {
        System.out.print(lIter.next() + " - ");
    }
    
    System.out.println("\r\nBack to front:".toUpperCase());
    System.out.print("\t");
    while (lIter.hasPrevious()) {
        System.out.print(lIter.previous() + " - ");
    }
}
```
输出结果：
```java
FROM FRONT TO BACK:
        ONE - TWO - THREE -
BACK TO FRONT:
        THREE - TWO - ONE -
```

## Enumeration 枚举输出
Enumeration 设置的主要目的是输出 Vector 集合数据。

代码如下：
```java
public static void runEnumeration() {
    Vector<String> vector = new Vector<>();
    vector.add("ONE");
    vector.add("TWO");
    vector.add("THREE");
    Enumeration<String> enumeration = vector.elements();
    while (enumeration.hasMoreElements()) {
        System.out.println(enumeration.nextElement());
    }
}
```

输出：
```java
ONE
TWO
THREE
```

## foreach 遍历输出

foreach 除了可以实现数组输出外，还支持集合的输出操作。

代码如下：
```java
public static void runForeach() {
    Set<String> set = new HashSet<>();
    set.add("ONE");
    set.add("TWO");
    set.add("THREE");
    for (String string : set) {
        System.out.println(string);
    }
}
```

输出：
```java
ONE
TWO
THREE
```
其他操作与数组输出完全相同。


------------

> 相关推荐
> [【Howe 学 JAVA】Java 类集框架1——List集合](https://neusoft.me/ziyuan/java/2020/05/11/517/ "【Howe 学 JAVA】Java 类集框架1——List集合")
> [Howe 学 JAVA】Java 类集框架2——Set 集合](https://neusoft.me/ziyuan/java/2020/05/11/520/ "Howe 学 JAVA】Java 类集框架2——Set 集合")


------------

本文永久链接：[https://neusoft.me/ziyuan/java/2020/05/11/523/](https://neusoft.me/ziyuan/java/2020/05/11/523/ "https://neusoft.me/ziyuan/java/2020/05/11/523/")
