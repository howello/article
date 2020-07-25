### 引言
为了安全的进行数据传输，就需要对数据进行加密与解密操作，Base64就是JAVA提供的加解密处理工具。
### 背景知识
Base64是一种利用64个可打印字符来表示二进制数据的算法，也是在网络传输中较为常见的一种加密算法。从JDK1.8版本开始，在java.util中提供了Base64的工具类，其中有两个内部类实现数据加密和解密操作。

 - **【数据加密】**`java.util.Base64.Encoder`
 对象获取方法：`public static Base64.Encoder.getEncoder()`
 数据加密处理：`public byte[] encoder(byte[] src)`
 - **【数据解密】** `java.util.Base64.Decoder`
 对象获取方法：`public static Base64.Decoder.getDecoder()`
 数据加密处理：`public byte[] decoder(byte[] src)`
### 最简单的加密解密操作
代码块：

```java
 private static final String str = "abcdefghigklmnopqrstuvwxyz1234567890";

    /**
     * @name: 最简单的加密解密
     * @msg:
     * @param {type}
     * @return:
     */
    public static void simpleRun() {
        String encodiString = new String(Base64.getEncoder().encode(str.getBytes()));
        System.out.println(encodiString);
        encodiString = Base64.getEncoder().encodeToString(str.getBytes());
        System.out.println(encodiString);
        String oldString = String.valueOf(Base64.getDecoder().decode(encodiString));
        System.out.println(oldString);
        oldString = new String(Base64.getDecoder().decode(encodiString));
        System.out.println(oldString);
    }

```
输出：

```java
//加密数据
YWJjZGVmZ2hpZ2tsbW5vcHFyc3R1dnd4eXoxMjM0NTY3ODkw
YWJjZGVmZ2hpZ2tsbW5vcHFyc3R1dnd4eXoxMjM0NTY3ODkw
//解密数据
[B@70dea4e
abcdefghigklmnopqrstuvwxyz1234567890
```
由此看出，加密操作有两种写法，而解密的时候`String.valueOf`取不到值。
### 使用“盐”值进行多次加密解密
代码块：
```java
 private static final String str = "abcdefghigklmnopqrstuvwxyz1234567890";

 /**
     * @name: 使用“盐”值进行多次加密解密
     * @msg:
     * @param {type}
     * @return:
     */
    public static void mRun() {
        String eString = encode(str);
        System.out.println(eString);
        String dString = decode(eString);
        System.out.println(dString);
    }

    private static final String SALT = "neusoft";
    private static final int REPEAT = 5;

    private static String encode(String string) {
        String temp = string + "{" + SALT + "}";
        byte[] data = temp.getBytes();
        for (int i = 0; i < REPEAT; i++) {
            data = Base64.getEncoder().encode(data);
        }
        return new String(data);
    }

    private static String decode(String string) {
        byte[] data = string.getBytes();
        for (int i = 0; i < REPEAT; i++) {
            data = Base64.getDecoder().decode(data);
        }
        return new String(data).replaceAll("\\{\\w+\\}", "");
    }
}
```
输出：

```java
//利用“盐”值加密5遍数据
VmpGYWExTXlSbk5qUldoWFlsUkdhRlJYTVc5a01XUnhVMnBDYWsxcmNGbFViR2hoWVd4T1JsZHFWbHBsYXpWVVZGWmtVMlJXY0VWVmJYaFlVbnByTUZaR1pIZFVhekZHVGxaV1dGWkZOVkZWYTJRd1RURndWVk5VUm1sU01VcGFWa2MxWVZsWFNuTlRWRlU5
//解密数据
abcdefghigklmnopqrstuvwxyz1234567890
```
本程序基于Base64的功能实现了一个自定义加密解密程序，使用“盐”值多次加密后确保了密文数据的可靠性。在实际开发中如果不对外公布“盐”值内容和加密次数，就可以在较为安全的环境下进行数据传输。
>所学所得，如有不正确之处还请留言指正。
>个人博客永久链接： [https://neusoft.me/ziyuan/java/2020/05/03/54/](https://neusoft.me/ziyuan/java/2020/05/03/54/ "https://neusoft.me/ziyuan/java/2020/05/03/54/")
