> 有时候需要为apk添加系统级签名，起始很简单的几步就能够生成系统签名，然后倒入IDE或者手动签名即可。
此方法来自：http://curlog.com/2016/08/30/android-pk2debug-keystore/

###### 使用到系统级Api的，或者Androidmanifest.Xml文件中声明了

`android:sharedUserId=“android.uid.system”`

###### 那么没有系统签名，直接debug签名运行是不行的

#### 1、准备

- platform.pk8
- platform.x509.pem

#### 2、动手前的知识理解

- 为什么需要这两个文件？需要这两个文件干什么？

 因为需要生成系统级的签名，必要依靠系统里面的文件platform.pk8和platform.x509.pem转换成签名文件，然后才能签名apk进行debug调试或者直接安装。

在不知道系统签名可以转换成debug签名前，老实说我一直都是用Log的方式调试，太特么痛苦了。现在知道后整个人都懵逼了。我们都希望可以像调试普通app那样调试系统app.
	
- 从哪里能够取得这两个文件？
	如果你调试机是原生手机，可以直接从源码网站下载源码，在源码目录build\target\product\security中，直接下载就可以。
	如果你调试机的系统时定制系统的话，就需要找底层工程师要这两个文件，或者自己解包找这两个文件。

#### 3、开始动手
最好是在Linux环境下进行操作，当然Windows下和Mac下也是可以的。首先需要安装openssl工具，Mac自带openssl，Windows和Linux需要自己安装。这里给出Centos7的安装方法。
```bash
yum install openssl
yum install openssl-devel
```
##### 第一步：生成platform.priv.pem第一步：生成platform.priv.pem
```bash
openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out platform.priv.pem -nocrypt
```
##### 第二步：生成platform.pk12
```bash
openssl pkcs12 -export -in platform.x509.pem -inkey platform.priv.pem -out platform.pk12 -name androiddebugkey
```
这一步中，`-name`后跟的是签名文件中的key名，执行过程中会提示输入密码，输入自己签名密码即可。
##### 第三步：生成签名文件
```bash
keytool -importkeystore -deststorepass android -destkeypass android -destkeystore debug.keystore -srckeystore platform.pk12 -srcstoretype PKCS12 -srcstorepass android -alias androiddebugkey
```
这一步中，`-deststorepass` 后面跟的是签名文件的密码，保持和key的密码相同，否则会报错。`-destkeypass` 后面跟的是key的密码，保持和签名文件密码相同。`-destkeystore` 后面跟签名文件名称。`-srcstorepass` 后面跟的是keystore密码，保持和其他相同。`-alias` 后面跟key的名字，和上一步保持相同。
##### 到此就能得到签名文件。到此就能得到签名文件。

------------


> **为小白写的文章，大牛请路过就好，谢谢。**

>原文地址：[https://neusoft.me/ziyuan/java/2020/05/02/40/](https://neusoft.me/ziyuan/java/2020/05/02/40/)
