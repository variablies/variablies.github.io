# Hutool（简化每一行代码）

### 01、引入 Hutool

Maven 项目只需要在 pom.xml 文件中添加以下依赖即可。

```text
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.3</version>
</dependency>
```

Hutool 的设计思想是尽量减少重复的定义，让项目中的 util 包尽量少。一个好的轮子可以在很大程度上避免“复制粘贴”，从而节省我们开发人员对项目中公用类库和公用工具方法的封装时间。同时呢，成熟的开源库也可以最大限度的避免封装不完善带来的 bug。

就像作者在官网上说的那样：

- 以前，我们打开搜索引擎 -> 搜“Java MD5 加密” -> 打开某篇博客 -> 复制粘贴 -> 改改，变得好用些

> 有了 Hutool 以后呢，引入 Hutool -> 直接 SecureUtil.md5()

Hutool 对不仅对 JDK 底层的文件、流、加密解密、转码、正则、线程、XML等做了封装，还提供了以下这些组件：

![image-20230425113853070](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/class/hutool.png?raw=true)

### 02、类型转换

类型转换在 Java 开发中很常见，尤其是从 HttpRequest 中获取参数的时候，前端传递的是整形，但后端只能先获取到字符串，然后再调用 parseXXX() 方法进行转换，还要加上判空，很繁琐。

Hutool 的 Convert 类可以简化这个操作，可以将任意可能的类型转换为指定类型，同时第二个参数 defaultValue 可用于在转换失败时返回一个默认值。

##### 字符串转整形

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void contextLoads() {

        // 将接受的字符串转换为整形
        String param = "10";
        int paramInt = Convert.toInt(param);
        int paramIntDefault = Convert.toInt(param, 0); // 这里第二个参数可在转换失败后返回一个默认值

        System.out.println(paramInt);
    }
}
```

##### 字符串转换成日期

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void stringToDate() {

        String dateStr = "2020年09月29日";
        Date date = Convert.toDate(dateStr);

        System.out.println(date);
    }
}

```

##### 字符串转成 Unicode

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void stringToDate() {

        String unicodeStr = "你好";
        String unicode = Convert.strToUnicode(unicodeStr);

        System.out.println(unicode);
    }
}
```

### 03、日期时间

JDK 自带的 Date 和 Calendar 不太好用，Hutool 封装的 DateUtil 用起来就舒服多了！

##### 获取当前日期

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void stringToDate() {

        Date date = DateUtil.date();
        System.out.println(date);
    }
}
```

DateUtil.date() 返回的其实是 DateTime，它继承自 Date 对象，重写了 toString() 方法，返回 yyyy-MM-dd HH:mm:ss 格式的字符串。

##### 字符串转日期

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void stringToDateTest() {

        String dateStr = "2020-09-29";
        Date date = DateUtil.parse(dateStr);

        System.out.println(date);
    }
}
```

DateUtil.parse() 会自动识别一些常用的格式，比如说：

- yyyy-MM-dd HH:mm:ss
- yyyy-MM-dd
- HH:mm:ss
- yyyy-MM-dd HH:mm
- yyyy-MM-dd HH:mm:ss.SSS

还可以识别带中文的：

- 年月日时分秒

##### 格式化时间差

```java
@SpringBootTest
public class Day06SpringBootHutoolApplicationTests {

    @Test
    void stringToDateTest() {

        String dateStr1 = "2020-09-29 22:33:23";
        Date date1 = DateUtil.parse(dateStr1);

        String dateStr2 = "2020-10-01 23:34:27";
        Date date2 = DateUtil.parse(dateStr2);

        long betweenDay = DateUtil.between(date1, date2, DateUnit.MS);

        // 输出：2天1小时1分4秒
        String formatBetween = DateUtil.formatBetween(betweenDay, BetweenFormater.Level.SECOND);

        System.out.println(formatBetween);
    }
}
```

##### 星座和属相

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void test() {
        // 射手座
        String zodiac = DateUtil.getZodiac(Month.DECEMBER.getValue(), 10);
        // 蛇
        String chineseZodiac = DateUtil.getChineseZodiac(1989);

        System.out.println(chineseZodiac + " " + zodiac);
    }
}
```

### 04、IO 流相关

IO 操作包括读和写，应用的场景主要包括网络操作和文件操作，原生的 Java 类库区分字符流和字节流，字节流 InputStream 和 OutputStream 就有很多很多种，使用起来让人头皮发麻。

Hutool 封装了流操作工具类 IoUtil、文件读写操作工具类 FileUtil、文件类型判断工具类 FileTypeUtil 等等。

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void Io() {
        BufferedInputStream in = FileUtil.getInputStream("hutool/origin.txt");
        BufferedOutputStream out = FileUtil.getOutputStream("hutool/to.txt");
        long copySize = IoUtil.copy(in, out, IoUtil.DEFAULT_BUFFER_SIZE);

        System.out.println(copySize);
    }
}
```

在 IO 操作中，文件的操作相对来说是比较复杂的，但使用频率也很高，几乎所有的项目中都躺着一个叫 FileUtil 或者 FileUtils 的工具类。Hutool 的 FileUtil 类包含以下几类操作：

- 文件操作：包括文件目录的新建、删除、复制、移动、改名等
- 文件判断：判断文件或目录是否非空，是否为目录，是否为文件等等
- 绝对路径：针对 ClassPath 中的文件转换为绝对路径文件
- 文件名：主文件名，扩展名的获取
- 读操作：包括 getReader、readXXX 操作
- 写操作：包括 getWriter、writeXXX 操作

在实际编码当中，我们通常需要从某些文件里面读取一些数据，比如配置文件、文本文件、图片等等，那这些文件通常放在什么位置呢？

![image-20230425131144888](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/class/hutool002.png?raw=true)

放在项目结构图中的 resources 目录下，当项目编译后，会出现在 classes 目录下。

当我们要读取文件的时候，我是不建议使用绝对路径的，因为操作系统不一样的话，文件的路径标识符也是不一样的。最好使用相对路径。

假设在 src/resources 下放了一个文件 origin.txt，文件的路径参数如下所示：

```text
FileUtil.getInputStream("origin.txt")
```

### 05、字符串工具 

Hutool 封装的字符串工具类 StrUtil 和 Apache Commons Lang 包中的 StringUtils 类似，作者认为优势在于 Str 比 String 短

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {	
	@Test
    void stringTool() {
        String template = "{}，是一个好用的{}";
        String str = StrUtil.format(template, "Hutool", "工具包。");

        System.out.println(str);
    }
}
```

### 06、反射工具

反射机制可以让 Java 变得更加灵活，因此在某些情况下，反射可以做到事半功倍的效果。Hutool 封装的反射工具 ReflectUtil 包括：

- 获取构造方法
- 获取字段
- 获取字段值
- 获取方法
- 执行方法（对象方法和静态方法）

```java
public class ReflectDemo  {

    private int id;

    public ReflectDemo() {
        System.out.println("构造方法");
    }

    public void print() {
        System.out.println("你好， Hutool！");
    }

    public static void main(String[] args) throws IllegalAccessException {
        // 构建对象
        ReflectDemo reflectDemo = ReflectUtil.newInstance(ReflectDemo.class);

        // 获取构造方法
        Constructor[] constructors = ReflectUtil.getConstructors(ReflectDemo.class);
        for (Constructor constructor : constructors) {
            System.out.println(constructor.getName());
        }

        // 获取字段
        Field field = ReflectUtil.getField(ReflectDemo.class, "id");
        field.setInt(reflectDemo, 10);
        // 获取字段值
        System.out.println(ReflectUtil.getFieldValue(reflectDemo, field));

        // 获取所有方法
        Method[] methods = ReflectUtil.getMethods(ReflectDemo.class);
        for (Method m : methods) {
            System.out.println(m.getName());
        }

        // 获取指定方法
        Method method = ReflectUtil.getMethod(ReflectDemo.class, "print");
        System.out.println(method.getName());


        // 执行方法
        ReflectUtil.invoke(reflectDemo, "print");
    }
}
```

### 07、压缩工具

在 Java 中，对文件、文件夹打包压缩是一件很繁琐的事情，Hutool 封装的 ZipUtil 针对 java.util.zip 包做了优化，可以使用一个方法搞定压缩和解压，并且自动处理文件和目录的问题，不再需要用户判断，大大简化的压缩解压的复杂度。

```text
ZipUtil.zip("hutool", "hutool.zip");
File unzip = ZipUtil.unzip("hutool.zip", "hutoolzip");
```

### 08、身份证工具

Hutool 封装的 IdcardUtil 可以用来对身份证进行验证，支持大陆 15 位、18 位身份证，港澳台 10 位身份证。

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void idCard() {
        String ID_18 = "321083197812162119";
        String ID_15 = "150102880730303";

        boolean valid = IdcardUtil.isValidCard(ID_18);
        boolean valid15 = IdcardUtil.isValidCard(ID_15);

        System.out.println(valid);
        System.out.println(valid15);
    }
}

```

### 09、扩展 HashMap

Java 中的 HashMap 是强类型的，而 Hutool 封装的 Dict 对键的类型要求没那么严格。

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void huToolHashMap() {
        Dict dict = Dict.create()
                .set("age", 18)
                .set("name", "shmily")
                .set("birthday", DateTime.now());

        int age = dict.getInt("age");
        String name = dict.getStr("name");
        Date birthday = dict.getDate("birthday");

        System.out.println(age + name + birthday);
    }
}
```

### 10、控制台打印

本地编码的过程中，经常需要使用 System.out 打印结果，但是往往一些复杂的对象不支持直接打印，比如说数组，需要调用 Arrays.toString。Hutool 封装的 Console 类借鉴了 JavaScript 中的 console.log()，使得打印变成了一个非常便捷的方式。

```java 
public class ConsoleDemo {
    public static void main(String[] args) {
        // 打印字符串
        Console.log("沉默王二，一枚有趣的程序员");

        // 打印字符串模板
        Console.log("洛阳是{}朝古都",13);

        int [] ints = {1,2,3,4};
        // 打印数组
        Console.log(ints);
    }
}
```

### 11、字段验证器

做 Web 开发的时候，后端通常需要对表单提交过来的数据进行验证。Hutool 封装的 Validator 可以进行很多有效的条件验证：

- 是不是邮箱
- 是不是 IP V4、V6
- 是不是电话号码
- 等等

```java
@SpringBootTest
class Day06SpringBootHutoolApplicationTests {

    @Test
    void hutool() {
        boolean shmily = Validator.isEmail("shmily");
        boolean mobile = Validator.isMobile("itwanger.com");

        System.out.println(shmily);
        System.out.println(mobile);
    }
}
```

### 12、加密解密

加密分为三种：

- 对称加密（symmetric），例如：AES、DES 等
- 非对称加密（asymmetric），例如：RSA、DSA 等
- 摘要加密（digest），例如：MD5、SHA-1、SHA-256、HMAC 等

Hutool 针对这三种情况都做了封装：

- 对称加密 SymmetricCrypto
- 非对称加密 AsymmetricCrypto
- 摘要加密 Digester

快速加密工具类 SecureUtil 有以下这些方法：

1）对称加密

- SecureUtil.aes
- SecureUtil.des

2）非对称加密

- SecureUtil.rsa
- SecureUtil.dsa

3）摘要加密

- SecureUtil.md5
- SecureUtil.sha1
- SecureUtil.hmac
- SecureUtil.hmacMd5
- SecureUtil.hmacSha1

例子 :

```java
public class SecureUtilDemo {
    static AES aes = SecureUtil.aes();
    public static void main(String[] args) {

        String encry = aes.encryptHex("shmily");
        System.out.println(encry);
        String oo = aes.decryptStr(encry);
        System.out.println(oo);
    }
}
```

### 13、其他类库

Hutool 中的类库还有很多，尤其是一些对第三方类库的进一步封装，比如邮件工具 MailUtil，二维码工具 QrCodeUtil，Emoji 工具 EmojiUtil，小伙伴们可以参考 Hutool 的官方文档：https://www.hutool.cn/

项目源码地址：https://github.com/looly/hutool