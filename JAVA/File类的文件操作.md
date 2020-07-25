> 引言
I/O （ Input/Output ，输入、输出）可以实现数据的存取与写入操作。其中读取文件最简单的就是File类。


## 简介

`java.io.File` 类是一个与文件本身操作有关的类，此类可以实现文件创建、删除、重命名、取的文件基本信息（大小、修改日期）等常见的系统文件操作。

## File 类的基础操作

代码如下：

```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";
/**
 * @name: File类的文件基础操作
 * @msg:
 * @param {type}
 * @return:
 */
public static void fileCreatAndDelete() {
    File file = new File(filePath);// 构造File对象
    if (!file.getParentFile().exists()) {// 判断文件父目录是否存在
        file.getParentFile().mkdirs();// 不存在就创建
    }
    if (!file.exists()) {// 判断文件是否岑仔
        try {
            System.out.println("createNewFile:\t" + file.createNewFile());// 不存在就创建，创建成功返回True，不成功会抛出IOException异常
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    // 获取文件基本信息
    System.out.println("canRead:\t" + file.canRead());// 文件是否能读
    System.out.println("canWrite:\t" + file.canWrite());// 文件是否能写
    System.out.println("canExecute:\t" + file.canExecute());// 文件是否能执行
    System.out.println("length:\t" + file.length());// 获取文件大小，返回字节长度
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");
    System.out.println("lastModified:\t" + sdf.format(file.lastModified()));// 获取文件最后一次修改时间，返回时间戳，这里使用 SimpleDateFormat 转成了常用格式
    System.out.println("isDirectory:\t" + file.isDirectory());//是否为目录
    System.out.println("isFile:\t" + file.isFile());//是否为文件
    System.out.println("isHidden:\t" + file.isHidden());//是否隐藏
    if (file.getParentFile().isDirectory()) {
        File[] files = file.getParentFile().listFiles();//列出目录中的全部文件，包括文件夹
        for (File fi : files) {
            System.out.println(fi.getAbsolutePath());
        }
    }
}
```
代码执行之后，输出结果为
```java
createNewFile:  true
canRead:        true
canWrite:       true
canExecute:     true
length: 0
lastModified:   2020-05-04 16:32:05:634
isDirectory:    false
isFile: true
isHidden:       false
E:\005-AndroidHelloWorld\myMaven\mp3\1
E:\005-AndroidHelloWorld\myMaven\mp3\1.java
E:\005-AndroidHelloWorld\myMaven\mp3\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\2
E:\005-AndroidHelloWorld\myMaven\mp3\2.java
E:\005-AndroidHelloWorld\myMaven\mp3\3
```

## 递归列出文件夹及其子文件夹中的文件
从上面的结果不难看出，当文件夹中有子文件夹中的时候不会列出子文件夹中的东西。下面我们就用递归的方式来列出嵌套文件夹中的所有文件。
代码如下：
```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";

/**
 * @name: 递归列出文件夹下的所有文件及文件夹
 * @msg:
 * @param {type}
 * @return:
 */
public static void ListDirectory(File file) {
    if (file.isDirectory()) {
        File[] results = file.listFiles();
        if (results != null && results.length != 0) {
            for (int i = 0; i < results.length; i++) {
                ListDirectory(results[i]);
            }
        }
    }
    System.out.println(file);
}
```
执行之后输出内容如下：
```java
E:\005-AndroidHelloWorld\myMaven\mp3\1\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\1\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\1
E:\005-AndroidHelloWorld\myMaven\mp3\1.java
E:\005-AndroidHelloWorld\myMaven\mp3\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\2\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\2\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\2\4
E:\005-AndroidHelloWorld\myMaven\mp3\2
E:\005-AndroidHelloWorld\myMaven\mp3\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6\7\1.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6\7\2.txt
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6\7
E:\005-AndroidHelloWorld\myMaven\mp3\3\5\6
E:\005-AndroidHelloWorld\myMaven\mp3\3\5
E:\005-AndroidHelloWorld\myMaven\mp3\3
E:\005-AndroidHelloWorld\myMaven\mp3
```

## 文件批量改名

如果有些不太聪明的小朋友将文件全部 `.java` 文件错误的创建成了 `.txt` ，那么怎么才能批量给他改过来呢？下面我们就来看看文件批量重命名。
代码如下：
```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";

/**
 * @name: 递归改名
 * @msg:
 * @param {type}
 * @return:
 */
public static void renameFile(File file) {
    if (file.isDirectory()) {
        File[] results = file.listFiles();
        for (int i = 0; i < results.length; i++) {
            renameFile(results[i]);
        }
    } else {
        String newFileName = file.getName().substring(0, file.getName().lastIndexOf('.')) + ".java";
        if (file.isFile() && file.getName().substring(file.getName().lastIndexOf("."), file.getName().length())
                .equals(".txt")) {
            System.out.println("RENAMEING:" + file.getName() + "\t-->\t" + newFileName);
            file.renameTo(new File(file.getParentFile(), newFileName));
        }
    }
}
```
输出结果如下：
```java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
RENAMEING:1.txt -->     1.java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
RENAMEING:2.txt -->     2.java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
RENAMEING:1.txt -->     1.java
RENAMEING:2.txt -->     2.java
```
可以看到，所有的`.txt`都已经被改成了`.java`。当然，我这里只写了更改后缀的实现，喜欢思考的小朋友一定已经知道了，还可以写更多的。比如，重命名为有序的文件、名字里面添加或者删除字符等等。


------------

今天的分享就到这里，最后加上一句今天看到的鸡汤：
> 一有想法马上着手实施，边做边想，考虑多了往往事儿就黄了。

<!--more-->
---所学所得，如有不正确之处，还请指出提点，多谢！

博客永久链接：https://neusoft.me/java/2020/05/04/504/

