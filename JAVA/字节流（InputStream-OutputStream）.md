> 流是I/O中的基本操作单元，在流设计中都会提供有输入域输出两方面支持。下面是流操作的基本步骤：
> 1. 如果要操作的是个文件，需要使用File先找到一个要操作的文件路径。
> 2. 通过字节流的子类为字节流对象实例化（向上转型）。
> 3. 执行读写操作。
> 4. 关闭操作资源，不管随后代码是啥，都要先关闭流，用 `close()` 方法。

## 字节输出流

字节（Byte）是进行 I/O 操作的基本数据单位，在程序进行字节数据输出时就可以使用字节输出流（OutputStream）完成。
代码如下：
```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";
public static void writeOutput() {
    try {
        File file = new File(filePath);
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        OutputStream out = new FileOutputStream(file);// 子类实例化输出流对象
        String content = "I AM IRON MAN";
        out.write(content.getBytes());// 字符串转化为字节数组
        System.out.println("OUTPUT SUCCESS");
        out.close();// 关闭资源
    } catch (IOException e) {
        // TODO: handle exception
    }
}
```

执行之后文件内容为：
```
I AM IRON MAN
```
值得注意的是，输出时必须保证父目录的存在，但是不需要保证文件的存在，在输出时会自动创建文件。
但是在使用流操作文件时，往往会忘记关闭，可以使用 AutoCloseable 来自动关闭。
代码如下：

```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";

public static void appendOutput() {
    File file = new File(filePath);
    try (OutputStream out = new FileOutputStream(file, true)) {//这里 true 指的是是否追加，选false时不会追加，只会覆盖。
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        if (!file.exists()) {
            file.createNewFile();
        }
        String content = "\r\nBUT I AM THANOS";
        out.write(content.getBytes());
        System.out.println("OUTPUT SUCCESS");
    } catch (IOException e) {
    }
}
```
执行之后文件变为：
```
I AM IRON MAN
BUT I AM THANOS
```

## 字节输入流

字节输入流（InputStream）属于抽象类，可以使用 FileInputStream 子类来实现实例化。
代码如下：
```java 
public static final String filePath = System.getProperty("user.dir") + File.separat
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.se
public static void simpleRun() {
    File file = new File(filePath);
    if (file.exists()) {
        try {
            InputStream in = new FileInputStream(file);// 文件输入流实例化
            byte[] data = new byte[1024];// 数据读取缓存区
            // 读取数据，将数据存储到缓存区中，并同时返回读取的字节个数
            int len = in.read(data);
            System.out.println(new String(data, 0, len));// 字节转为字符串
            in.close();// 关闭资源
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```
执行之后的结果为
```
I AM IRON MAN
BUT I AM THANOS
```
但是这样的方法有一点繁琐，而且浪费空间。在JDK1.9之后提供了返回全部输入流内容的方法 readAllBytes()。

代码如下
```java
public static final String filePath = System.getProperty("user.dir") + File.separator + "mp3" + File.separator
        + "1.txt";
public static final String directoryPath = System.getProperty("user.dir") + File.separator + "mp3";

public static void readAllContent() {
    File file = new File(directoryPath + File.separator + "1.java");
    if (file.exists()) {
        InputStream in = new FileInputStream(file);
        byte[] data = in.readAllBytes();// jdk1.9之后才有
        System.out.println(new String(data));
        in.close();
    }
}
```
执行之后，输出为：
```
I AM IRON MAN
BUT I AM THANOS
```

------------

> 最后再说一些废话吧
> 我知道，我现在是在白忙活，发了也不会有人看。但是，从小到大，我都没有从一而终的坚持完成某一件事情，这一次我想要挑战一下自己，坚持写下去。尽量做到日更，最大限度两天一更。
> 加油吧！

所学所得，如有不正确之处，还请指出提点，多谢！

永久链接： https://neusoft.me/java/2020/05/06/509/
