> 网络爬虫（又称为网页蜘蛛，网络机器人，在FOAF社区中间，更经常的称为网页追逐者），是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本。另外一些不常使用的名字还有蚂蚁、自动索引、模拟程序或者蠕虫。

## 如何开始
做一件事情，难得不是做什么，难得是怎么做，怎么开始。良好的开端成功的一半，下面让我们一起头脑风暴一下，这件事情应该怎么做。
![image](https://upload-images.jianshu.io/upload_images/23246420-1c7e3ea969d03365.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


总共有31个省，每个省有很多市，每个市有很多县，每个县有很多乡镇，每个乡镇有很多居委会。既然这样，第一反应肯定是循环，遍历。那么接下来，应该怎么循环。先将这个页面的静态页面get到，然后将东西保存并获取链接，再然后循环爬取。

想到了就干，但是写完之后碰到了很多问题。
1. 文件怎么保存，总共有6级嵌套。
2. 循环结束才可以保存文件，如果循环中抛异常会导致文件损坏。

关于这两个问题，我的解决方法是：
1. 文件夹下保存一个省汇总的xls，将省的信息保存下来，包括名称、代号、链接。然后创建每个省的文件夹，在这个文件夹中保存一个市汇总的xls，保存市的名称、代号、链接。以此类推。
2. 先将省列表爬完，期间不去爬市的信息，得到省的链接。然后再用保存下来的链接去获取每个省的市的信息，期间不去管下一级的信息。一次类推，很大程度上杜绝了文件损坏的发生。

## 代码分析
在详细研究了各个层次的请求参数之后，其中市县乡的爬取可以提取一个方法。我将请求提取了一个方法。

### Get 请求方法

代码：
```java
public String GetMethod(URL url, String encode) {
    BufferedReader in = null;
    String ret = null;
    HttpURLConnection conn = null;
    try {
        conn = (HttpURLConnection) url.openConnection();
		
        if (conn != null) {
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setRequestMethod("GET");
            conn.connect();
        }
        in = new BufferedReader(new InputStreamReader(conn.getInputStream(), encode));
        String line = null;
        while ((line = in.readLine()) != null) {
            ret += line;
        }
        return ret;
    } catch (Exception e) {
        e.printStackTrace();
        return "Error :" + e.getMessage();
    } finally {
        try {
            if (in != null) {
                in.close();
            }
            conn.disconnect();
        } catch (final IOException ex) {
            ex.printStackTrace();
        }
    }
}
```
这个方法很简单的 get 请求。但是细心的同学可能注意到了，我给这个方法传了一个 `String encode`的参数。这就是我遇到的第一个坑，因为网页编码格式是 GB2312 ，而JAVA默认格式是UTF-8，所以没有传这个参数的时候默认获取UTF-8的编码信息，存下来之后发现全部汉字变成了乱码。


### 获取省份信息
代码：
```java
private static String getProvince() {
    WebUtils webUtils = new WebUtils();
    WritableWorkbook book = null;
    try {
        File file = new File(parentPath + File.separator + "汇总.xls");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        book = Workbook.createWorkbook(file);
        WritableSheet sheet = book.createSheet("省份", 0);
        WritableFont font = new WritableFont(WritableFont.ARIAL, 12);
        WritableCellFormat cellFormat = new WritableCellFormat(font);
        sheet.setColumnView(0, 30);
        sheet.setColumnView(1, 10);
        sheet.setColumnView(2, 90);
        sheet.addCell(new Label(0, 0, "名称", cellFormat));
        sheet.addCell(new Label(1, 0, "代码", cellFormat));
        sheet.addCell(new Label(2, 0, "链接", cellFormat));
        String ret = webUtils.GetMethod(new URL(baseUrl + "index.html"), "GB2312");
        Document doc = Jsoup.parse(ret);
        Elements provincetrEles = doc.getElementsByClass("provincetr");
        int i = 1;
        for (Element element : provincetrEles) {
            Elements tdEles = element.getElementsByTag("td");
            for (Element td : tdEles) {
                String href = td.select("a").attr("href");
                String name = td.select("a").text().trim();
                String areaCode = "";
                try {
                    areaCode = href.substring(0, href.indexOf("."));
                } catch (Exception e) {
                    // TODO: handle exception
                }
                if (href != "" && areaCode != null && name != "") {
                    sheet.addCell(new Label(0, i, name, cellFormat));
                    sheet.addCell(new Label(1, i, areaCode, cellFormat));
                    sheet.addCell(new Label(2, i, baseUrl + href, cellFormat));
                    System.out.println("-[Province]  " + i + "  SAVED:" + name + " - " + href);
                    i++;
                }
            }
        }
        return file.getAbsolutePath();
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (RowsExceededException e) {
        e.printStackTrace();
    } catch (WriteException e) {
        e.printStackTrace();
    } finally {
        if (book != null) {
            try {
                book.write();
                book.close();
                System.out.println("Complete".toUpperCase());
            } catch (WriteException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    return null;
}
```
这里面包含了自己写的GetMethod 方法，用jxl 来操作xls 表格，用Jsoup来解析get下来的静态页面。值得注意的是，一定要记得释放资源。

### 获取市县乡

这三个获取大同小异，可以提取一个公共方法。
#### 公共方法
```java
private static void creatXls(String path, String name, String url, String className) {
    WritableWorkbook book = null;
    try {
        File file = new File(path + File.separator + "汇总.xls");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        book = Workbook.createWorkbook(file);
        WritableSheet sheet = book.createSheet(name, 0);
        WritableFont font = new WritableFont(WritableFont.ARIAL, 12);
        WritableCellFormat cellFormat = new WritableCellFormat(font);
        sheet.setColumnView(0, 30);
        sheet.setColumnView(1, 30);
        sheet.setColumnView(2, 90);
        sheet.addCell(new Label(0, 0, "名称", cellFormat));
        sheet.addCell(new Label(1, 0, "统计用区划代码", cellFormat));
        sheet.addCell(new Label(2, 0, "链接", cellFormat));
        WebUtils webUtils = new WebUtils();
        String ret = webUtils.GetMethod(new URL(url), "GB2312");
        Document doc = Jsoup.parse(ret);
        Elements trElements = doc.getElementsByClass(className);
        int i = 1;
        for (Element tr : trElements) {
            Elements tdElements = tr.getElementsByTag("td");
            if (tdElements.size() != 0) {
                String href = tdElements.first().select("a").attr("href");
                String areaCode = tdElements.first().select("a").text().trim();
                String cityName = tdElements.last().select("a").text().trim();
                if (href != "" && areaCode != null && cityName != "") {
                    sheet.addCell(new Label(0, i, cityName, cellFormat));
                    sheet.addCell(new Label(1, i, areaCode, cellFormat));
                    sheet.addCell(new Label(2, i, url.substring(0, url.lastIndexOf("/") + 1) + href, 
cellFormat));
                    System.out.println("  --[" + className + "]  " + i + "  SAVED:" + cityName + " - " + 
areaCode
                            + " - " + href);
                    i++;
                }
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (book != null) {
            try {
                book.write();
                book.close();
            } catch (WriteException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}
```
没什么特别的，跟获取省的差不多。其中传入参数参见下表
- String path			上级爬取之后创建的新文件夹路径
- String name			这个传不传无所谓，是即将爬的单位名
- String url			即将爬取的链接
- String className		静态页面中的class名

#### 获取市
```java
private static String getCity(File file, String code) throws Exception {
    if (!file.exists()) {
        throw new Exception("【Error】文件不存在：" + file.getName());
    }
    try {
        String[] nameList = readXLS(file, 0, 1);
        String[] codeList = readXLS(file, 1, 1);
        String[] urlList = readXLS(file, 2, 1);
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        if (code != null) {
            // TODO 单独获取
            int index = Arrays.binarySearch(codeList, code);
            if (index < 0) {
                throw new Exception("省份代码不存在");
            }
            String path = file.getParent() + File.separator + nameList[index];
            creatXls(path, nameList[index], urlList[index], "citytr");
            return path + File.separator + "汇总.xls";
        } else {
            String ret = "";
            for (int i = 0; i < codeList.length; i++) {
                if (nameList[i] != null && nameList[i] != "" && nameList[i] != "null") {
                    String path = file.getParent() + File.separator + nameList[i];
                    creatXls(path, nameList[i], urlList[i], "citytr");
                    ret = ret + path + File.separator + "汇总.xls" + SPLITE_SYMBOL;
                }
            }
            return ret;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

#### 获取县
```java
private static String getCountry(File file, String code) throws Exception {
    if (!file.exists()) {
        throw new Exception("【Error】文件不存在：" + file.getAbsolutePath());
    }
    try {
        String[] nameList = readXLS(file, 0, 1);
        String[] codeList = readXLS(file, 1, 1);
        String[] urlList = readXLS(file, 2, 1);
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        if (code != null) {
            // TODO 单独获取
            int index = Arrays.binarySearch(codeList, code);
            if (index < 0) {
                throw new Exception("市代码不存在");
            }
            String path = file.getParent() + File.separator + nameList[index];
            creatXls(path, nameList[index], urlList[index], "countytr");
            return path + File.separator + "汇总.xls";
        } else {
            String ret = "";
            for (int i = 0; i < codeList.length; i++) {
                if (nameList[i] != null && nameList[i] != "" && nameList[i] != "null") {
                    String path = file.getParent() + File.separator + nameList[i];
                    creatXls(path, nameList[i], urlList[i], "countytr");
                    ret = ret + path + File.separator + "汇总.xls" + SPLITE_SYMBOL;
                }
            }
            return ret;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

#### 获取乡镇
```java
private static String getTown(File file, String code) throws Exception {
    if (!file.exists()) {
        throw new Exception("【Error】文件不存在：" + file.getName());
    }
    if (!file.getParentFile().exists()) {
        file.getParentFile().mkdirs();
    }
    try {
        String[] nameList = readXLS(file, 0, 1);
        String[] codeList = readXLS(file, 1, 1);
        String[] urlList = readXLS(file, 2, 1);
        if (code != null) {
            // TODO 单独获取
            int index = Arrays.binarySearch(codeList, code);
            if (index < 0) {
                throw new Exception("县代码不存在");
            }
            String path = file.getParent() + File.separator + nameList[index];
            creatXls(path, nameList[index], urlList[index], "towntr");
            return path + File.separator + "汇总.xls";
        } else {
            String ret = "";
            for (int i = 0; i < codeList.length; i++) {
                if (nameList[i] != null && nameList[i] != "" && nameList[i] != "null") {
                    String path = file.getParent() + File.separator + nameList[i];
                    creatXls(path, nameList[i], urlList[i], "towntr");
                    ret = ret + path + File.separator + "汇总.xls" + SPLITE_SYMBOL;
                }
            }
            return ret;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```
### 获取居委会
居委会内容比其他的多点，其他的大差不差，但是还是不能跟上面的提取公共方法，起码我不会。但是为了让单独获取和批量获取复用，我还是提取了一个方法。
#### 公共方法
```java
private static void createVillageXLS(File file, String url) {
    WritableWorkbook book = null;
    try {
        String ret = null;
        book = Workbook.createWorkbook(file);
        WritableSheet sheet = book.createSheet(file.getName().replace(".xls", ""), 0);
        WebUtils webUtils = new WebUtils();
        ret = webUtils.GetMethod(new URL(url), "GB2312");
        WritableFont font = new WritableFont(WritableFont.ARIAL, 12);
        WritableCellFormat cellFormat = new WritableCellFormat(font);
        sheet.setColumnView(0, 30);
        sheet.setColumnView(1, 30);
        sheet.setColumnView(2, 30);
        sheet.addCell(new Label(0, 0, "名称", cellFormat));
        sheet.addCell(new Label(1, 0, "统计用区划代码", cellFormat));
        sheet.addCell(new Label(2, 0, "城乡分类代码", cellFormat));
        Document doc = Jsoup.parse(ret);
        Elements trElements = doc.getElementsByClass("villagetr");
        int i = 1;
        for (Element tr : trElements) {
            Elements tdElements = tr.getElementsByTag("td");
            if (tdElements.size() != 0) {
                String[] strs = tdElements.text().split(" ");
                String areaCode = strs[0];
                String classifiCode = strs[1];
                String villageName = strs[2];
                sheet.addCell(new Label(0, i, villageName, cellFormat));
                sheet.addCell(new Label(1, i, areaCode, cellFormat));
                sheet.addCell(new Label(2, i, classifiCode, cellFormat));
                System.out.println(
                        " --[Village] " + i + " SAVED:" + villageName + " - " + areaCode + " - " + 
classifiCode);
                i++;
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (book != null) {
            try {
                book.write();
                book.close();
            } catch (WriteException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}
```

#### 获取居委会
```java
private static void getVillage(File file, String code) throws Exception {
    if (!file.exists()) {
        throw new Exception("【Error】文件不存在：" + file.getName());
    }
    try {
        String[] nameList = readXLS(file, 0, 1);
        String[] codeList = readXLS(file, 1, 1);
        String[] urlList = readXLS(file, 2, 1);
        String path = null;
        if (code != null) {
            int index = Arrays.binarySearch(codeList, code);
            if (index < 0) {
                throw new Exception("县代码不存在");
            }
            path = file.getParent() + File.separator + nameList[index];
            file = new File(path, "村委会汇总.xls");
            if (!file.getParentFile().exists()) {
                file.getParentFile().mkdirs();
            }
            createVillageXLS(file, urlList[index]);
        } else {
            for (int i = 0; i < codeList.length; i++) {
                if (nameList[i] != null && nameList[i] != "" && nameList[i] != "null") {
                    path = file.getParent() + File.separator + nameList[i];
                    file = new File(path, "村委会汇总.xls");
                    if (!file.getParentFile().exists()) {
                        file.getParentFile().mkdirs();
                    }
                    createVillageXLS(file, urlList[i]);
                }
            }
        }
        Thread.sleep(DELAY_TIME);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

### 其他方法
#### 静态changliang

```java
    private static String baseUrl = "http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2019/";
    private static final String parentPath = System.getProperty("user.dir") + File.separator + "NBS";
    private static final int DELAY_TIME = 100;
```

#### 程序入口
```java
public static void run() {
    Scanner scanner = new Scanner(System.in);
    System.out.println("欢迎使用");
    String code = null;
    while (true) {
        System.out.println("请选择爬取方式：");
        System.out.println("手动选择爬取  --  1");
        System.out.println("自动全部爬取  --  2");
        System.out.println("请输入编码：(只输入数字)");
        code = scanner.nextLine().trim();
        if (code.matches("\\d")) {
            if (code.equals("1") || code.equals("2")) {
                break;
            } else {
                System.out.println("只能输入1或者2，请重新输入");
                continue;
            }
        } else {
            System.out.println("输入错误，请重新输入");
        }
    }
    chooseType(code, scanner);
    System.out.println("获取完毕，按Enter关闭此软件");
    scanner.nextLine();
    scanner.close();
}
```
#### 方式选择
```java
private static void chooseType(String i, Scanner scanner) {
    switch (i) {
        case "1":
            try {
                // System.out.println("请按 Enter 继续...");
                // scanner.nextLine();
                // 获取省
                String provinceSum = getProvince();
                if (provinceSum == null || provinceSum == "") {
                    System.out.println("加载失败");
                    return;
                }
                // 获取市
                File file = new File(provinceSum);
                String code = topLine(file, scanner);
                String citySum = getCity(file, code);
                if (citySum == null || citySum == "") {
                    System.out.println("加载失败");
                    return;
                }
                // 获取县
                file = new File(citySum);
                code = topLine(file, scanner);
                String countrySum = getCountry(file, code);
                if (countrySum == null || countrySum == "") {
                    System.out.println("加载失败");
                    return;
                }
                // 获取乡镇
                file = new File(countrySum);
                code = topLine(file, scanner);
                String townSum = getTown(file, code);
                if (townSum == null || townSum == "") {
                    System.out.println("加载失败");
                    return;
                }
                // 获取村委会
                file = new File(townSum);
                code = topLine(file, scanner);
                getVillage(file, code);
            } catch (Exception e) {
                e.printStackTrace();
            }
            break;
        case "2":
            try {
                for (int j = 0; j < 3; j++) {
                    if (j > 0) {
                        System.out.println("请再次确认" + (3 - j) + "遍");
                    }
                    System.out.println("****************************");
                    System.out.println("***********请注意************");
                    System.out.println("*******将会全部批量爬取*******");
                    System.out.println("********会消耗大量时间********");
                    System.out.println("****爬取过程中会有可能封ip*****");
                    System.out.println("******所以有可能中途失败*******");
                    System.out.println("*****如果中途停止，请等待******");
                    System.out.println("****************************");
                    System.out.println("*****如果你已了解相关危险******");
                    System.out.println("********请按Enter继续********");
                    System.out.println("*****************************");
                    scanner.nextLine();
                }
                // 获取省
                String provinceSum = getProvince();
                if (provinceSum == null || provinceSum == "") {
                    System.out.println("加载失败");
                    return;
                }
                File file = new File(provinceSum);
                String[] citySums = getCity(file, null).split(SPLITE_SYMBOL);
                for (String city : citySums) {
                    System.out.println("************CITY***************");
                    file = new File(city);
                    String[] countrySums = getCountry(file, null).split(SPLITE_SYMBOL);
                    for (String country : countrySums) {
                        System.out.println("************COUNTRY***************");
                        file = new File(country);
                        String[] townSums = getTown(file, null).split(SPLITE_SYMBOL);
                        for (String town : townSums) {
                            System.out.println("************TOWN***************");
                            file = new File(town);
                            getVillage(file, null);
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            break;
    }
}
```
#### 选择头
```java
private static String topLine(File file, Scanner scanner) throws Exception {
    String[] nameList = readXLS(file, 0, 1);
    String[] codeList = readXLS(file, 1, 1);
    String code = null;
    while (true) {
        for (int i = 0; i < codeList.length; i++) {
            System.out.println(nameList[i] + "  --  " + i);
        }
        System.out.println("请输入编码：(只输入数字)");
        code = scanner.nextLine();
        if (code.trim().matches("\\d+")) {
            return codeList[Integer.parseInt(code)];
        } else {
            System.out.println("输入错误，请重新输入：");
        }
    }
}
```

#### 读取xls 文件
```java
private static String[] readXLS(File file, int lieStart, int hangStart) throws Exception {
    String[] result;
    try {
        FileInputStream fis = new FileInputStream(file);
        Workbook wb = Workbook.getWorkbook(fis);
        Sheet sheet = wb.getSheet(0);
        int hangEnd = sheet.getRows();
        result = new String[hangEnd - hangStart + 1];
        for (int i = hangStart; i < hangEnd; i++) {
            result[i] = sheet.getCell(lieStart, i).getContents();
        }
        fis.close();
        wb.close();
        return result;
    } catch (Exception e) {
        e.printStackTrace();
        throw new Exception("【异常】" + e.getMessage());
    }
}
```

## 如何结束

本来到这里已经结束了，但是我这个人呢，就是想搞一些有意思的。如果我想在一台没装JDK的windows电脑上运行，可不可行？经过度娘畅游之后，还是被我发现了一些方法。这里只是顺带提一下，有机会以后再讲，或者各位看官也可以自己去找一下。下面我粗略的说一下方法：

这些代码我是用Vscode写的（别问为什么不用那么多的IDEA，因为我主力是安卓并且不喜欢装软件），Maven项目。但是呢，我又不会用这个导出为可执行jar文件。所以搬出了老大哥Eclipse，用Eclipse导出为可执行Jar文件。

但是如果电脑没有装JDK，那不是费了吗。jar文件执行不了。所有我又找到了一个软件 exe4j 可以将Jar文件转为exe文件。这样就完美了，转exe的时候将JRE放进去。这样就可以满足要求了。


------------

> 废话时间
> 这玩意儿虽说简单，我再下班时间写，还写了两天。学艺不精啊，还需要多磨练。
> 加油吧，少年


------------

需要源码可以私信联系我
