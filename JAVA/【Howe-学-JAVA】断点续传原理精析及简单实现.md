> 今天来说说大名鼎鼎的断点续传。顾名思义，文件传输的时候收到不确定因素的影响，打断传输状态，当再次传输的时候不需要从头开始传，从断掉的地方开始，节省时间，节省资源。

## 断点续传在生活中的例子
断点续传听着很简单，但是理解的话有稍微有点不太好理解。今天举一个生活中的例子，你一看就明白了。
大家肯定都玩过一些闯关游戏.当你玩到某个关卡的时候，女朋友说想和你去运动运动，然后你二话不说保存、退出、关电脑一气呵成。
但是当你运动完事儿之后，重新打开游戏，还会从第一关开始玩吗（说超级玛丽的那位老年人请出去）？答案肯定是否定的，因为关掉游戏之前，你已经保存过了，重新打开只需要读档就行。
断点续传也是这个道理，我们在中断的时候打个标记，记下中断的点位，下一次从这里开始读不就行了。

## 从程序思维思考

经过前面的例子，估计断点续传的原理大家基本了解，但是从程序思维再看，就出现问题了：
#### 出现的问题
1. 中断时的这个点位怎么记录？
2. 读取文件时怎么从这个点位开始？

#### 解决方法
1. 关于第一个问题，当我们读取的时候，使用 byte 数组作为媒介，循环进行转存，那我们可以将中断时 byte 数组的循环次数作为一个标记存下来。这样就解决了终端点位标记的问题。

2. 关于第二个问题，怎么从 byte 数组循环某次数的这个位置重新开始读写。如果你 JAVA 基本功比较扎实的话，你就会知道 JAVA 提供的一个文件操作类 `java.io.RandomAccessFile` ，这个类的定义是  **JAVA提供的对文件内容的访问，既可以读文件，也可以写文件。支持随机访问文件，可以访问文件的任意位置。** 下面是这个类里面提供的四个主要方法：

```java
void close() //关闭此随机访问文件流并释放与该流关联的所有系统资源。
int read(byte[] b) // 将最多 b.length 个数据字节从此文件读入 byte 数组。
void seek(long pos) //设置到此文件开头测量到的文件指针偏移量，在该位置发生下一个读取或写入操作。
void write(byte[] b) // 将 b.length 个字节从指定 byte 数组写入到此文件，并从当前文件指针开始。
```

ok，现在告诉我，从中发现了什么不同。没错，就是 `seek` 这个方法，这个方法不就正是之前说的想法吗。用这个方法直接跳到我们保存的位置处，然后再开始读写，美滋滋。

## 代码实操

说完理论知识，按照惯例，让代码来告诉我们真假，万一我是骗你们的怎么整。
### 需要传输的数据文件
虽然实际中大多用于转存大文件，http(s) 等大文件请求下载。但是呢，今天我们的目的是断点续传，所以我们使用本地的两个txt 文件进行实操。这时候又有调皮的朋友要说了，http(s) 和本地文件转存是不一样的。起始，这两个东西原理上差不多，都是把一个文件转存到另外一个地方，区别在于一个是本地磁盘到本地磁盘，另一个是远程磁盘到本地磁盘，不同的只是造成中断的原因不同而已。

文件用一个简单的txt，内容简单的写0-9十个数字，大小10个字节。

[![image](https://upload-images.jianshu.io/upload_images/23246420-3ae67141608c7b3d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://neusoft.me/wp-content/uploads/2020/05/wp_editor_md_df049741d905b6ab40e23a2d156039d7.jpg)

### 正常传输
我们先使用正常传输的方式，用流来进行读写。代码如下：
```java
private static void normalTrans(String sourceFilePath, String targetFilePath) {
    File sourFile = new File(sourceFilePath);
    File targetFile = new File(targetFilePath);
    FileInputStream fis = null;
    FileOutputStream fos = null;
    byte[] buf = new byte[1];
    try {
        fis = new FileInputStream(sourFile);
        fos = new FileOutputStream(targetFile);
        while (fis.read(buf) != -1) {
            System.out.println("write dataing ...".toUpperCase());
            fos.write(buf);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (fis != null) {
                fis.close();
            }
            if (fos != null) {
                fos.close();
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```
正常传输很简单，人均会写。没有什么好说的。其中，定义了一个byte[] 数组来作为缓冲区，大小为1个字节。也就是说读取的时候是一个字节一个字节的读，读完我们的十个数字需要循环读取十遍。这是个很重要的点。
### 中断传输
我们为了达到中断传输，人为抛出异常，将传输中断，代码如下：
```java
private static int breakTrans(String sourceFilePath, String targetFilePath) {
    int position = -1;
    File sourFile = new File(sourceFilePath);
    File targetFile = new File(targetFilePath);
    FileInputStream fis = null;
    FileOutputStream fos = null;
    byte[] buf = new byte[1];
    try {
        fis = new FileInputStream(sourFile);
        fos = new FileOutputStream(targetFile);
        while (fis.read(buf) != -1) {
            System.out.println("write dataing ...".toUpperCase());
            fos.write(buf);
            if (targetFile.length() == 3) {// 当目标文件长度写到三的时候，抛出异常，终端传输
                position = 3;
                throw new Exception();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (fis != null) {
                fis.close();
            }
            if (fos != null) {
                fos.close();
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    return position;
}
```

因为我们知道了，读取全部需要循环十遍，那么我们就在他没读取完毕的时候人为干预，让他停止。这里我是当循环第三遍的时候抛出了异常来中断的。中断之后我们打开，目标文件，发现里面只有前三个数。说明中断成功了。
[![image](https://upload-images.jianshu.io/upload_images/23246420-23bd5c82ec93bab5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://neusoft.me/wp-content/uploads/2020/05/wp_editor_md_710bb5c054c9b1e7e3295d8b69f88ed0.jpg)


### 断点继续传输文件

我们从刚才中断的地方开始，继续传输文件。代码如下：
```java
private static void continueTrans(String sourceFilePath, String targetFilePath, int position) {
    File sourFile = new File(sourceFilePath);
    File targetFile = new File(targetFilePath);
    RandomAccessFile read = null;
    RandomAccessFile write = null;
    byte[] buf = new byte[1];
    try {
        read = new RandomAccessFile(sourFile, "r");
        write = new RandomAccessFile(targetFile, "rw");
        // 关键，找到位置
        read.seek(position);
        write.seek(position);
        while (read.read(buf) != -1) {
            System.out.println("write dataing ...".toUpperCase());
            write.write(buf);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (read != null) {
                read.close();
            }
            if (write != null) {
                write.close();
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

这里使用 RandomAccessFile 来传输，使用 seek 方法来定位继续的位置，然后正常读写。读写完毕之后再打开目标文件，发现十个数已经全都过来了。那么我们断点续传的目的已经达到了。

[![image](https://upload-images.jianshu.io/upload_images/23246420-5c78cd625fe74350.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://neusoft.me/wp-content/uploads/2020/05/wp_editor_md_3784b2d1d365c34f1ccb936c4c6beec5.jpg)

### 调用方法
这个方法无所谓了，既然写了就放上来吧。
```java
public static void run() {
    String sourceFilePath = "1.txt";
    String targetFilePath = "2.txt";
    int position = 0;
    Scanner scanner = new Scanner(System.in);
    myWhile: while (true) {
        System.out.println("input type:".toUpperCase());
        System.out.println("1.normalTrans");
        System.out.println("2.breakTrans");
        System.out.println("3.continueTrans");
        System.out.println("4.quit");
        int type = scanner.nextInt();
        switch (type) {
            case 1:
                normalTrans(sourceFilePath, targetFilePath);
                break;
            case 2:
                position = breakTrans(sourceFilePath, targetFilePath);
                break;
            case 3:
                continueTrans(sourceFilePath, targetFilePath, position);
                break;
            case 4:
                break myWhile;
        }
    }
```

这个方法里起始也有一个值得注意的写法。就是里面的 myWhile ，很多人看到这个很纳闷，还能这么写？说白了起始很简单，myWhile 就是给这个 while 循环体起的名字。如果嵌套循环，或者像这种while 里面套 switch case的，在switch 里面break 只会跳出switch 。如果想跳出while 循环，就需要像这样给while 起个名字，然后再break 加上这个名字，告诉break 要跳出的是哪个。

## 说在最后
这里说的只是最简单的断点续传思想，而且这个思想在单线程里面勉强还可以用，但是多线程就gg 。多线程还有另外其他比较高级的思想来进行断点续传。这个咱们今天就不深入探究了，以后有机会再说。


------------

> 废话时间，没有卵用，可以直接略过
> 这两天突然发现我的三观有一点不太对劲，不能说是三观有问题，但是我突然感觉到了它不是很健康。
> 就是那种不知道从什么时候开始，不知道哪个地方出了一点点的问题，以至于自己本体对这个问题毫无察觉，然后这个问题慢慢的堆积，到后来堆积到一定程度的时候，你突然发觉了，已经变成了一个大问题。
> 我现在自我感觉还没发展成大问题，但是已经到可以悄无声息的影响我的情绪、心情、对于事情的看法等生活细节。目前也不知道怎么取解决一下。有过同样感觉的大佬，可以私我，咱们谈谈人生呐。


------------

文中如果有不对的地方，还请提出。有则改之，无则加勉。
