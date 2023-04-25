# Maven

Maven的优点 ：

- **依赖管理**：Maven 能帮助我们解决软件包依赖的管理问题，不再需要提交大量的 jar 包、引入第三方库；
- **规范目录结构**：Maven 标准的目录结构有助于项目构建的标准化，通过配置 profile 还可以根据不同的环境（开发环境、测试环境，生产环境）读取不同的配置文件；
- **方便集成**：能够集成在 IDE 中更方便使用。

### 一、安装 Maven

由于 JDK 是 Maven 安装的前置条件，所以请使用 `java -version` 确认是否已经安装了 JDK：

![](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Maven/Maven001.png?raw=true)

1）**一种官网下载，手动安装**

> Maven 官网下载地址 ：https://maven.apache.org/download.cgi

![image-20230424153615717](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Maven/Maven002.png?raw=true)

很多初学者在官网下载的时候不知道选哪一个，这里做一下简单的介绍。

- bin（binary）代表由 Java 源文件编译后的二进制 class 文件，src（source）代表Java 源文件。
- 一般情况下，选择 bin 文件进行安装就 OK 了；如果你想自己编译，可选 src 版本。
- tar.gz 压缩格式适用于 Unix 操作系统，zip 适用于 Windows 操作系统；但不是绝对的。

第二步，将压缩包解压出来

![image-20230424153900356](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Maven/Maven003.png?raw=true)

- bin 目录：该包含了 Maven 运行的所有脚本，用来配置 Java 命令，准备执行环境，然后执行 Java 命令。
- boot 目录：该目录只包含了一个 plexus-classworlds-xxx-jar 文件，该文件是一个类加载器框架，相当于默认的 Java 类加载器，提供了更加丰富的语法以便配置，Maven 使用该加载器加载自己的类库。
- **conf 目录**：该目录包含了一个非常重要的文件 settings.xml。可以直接修改该文件，用来全局定制 Maven 的行为；也可以复制该文件到 `~/.m2/` 目录下（~表示用户目录），修改该文件可以在用户范围内定制 Maven 的行为。
- lib 目录：该目录包含了Maven运行时所需要的 Java 类库，包括Maven 依赖的第三方类库，比如 slf4j-api.jar。

第三步，配置环境变量

windows :

打开系统高级设置 --> 环境变量 --> path --> 将Maven的bin路径添加到path里面

Mac : 

打开终端，输入 `vim ~/.bash_profile` 命令打开 bash_profile 文件：

bash_profile 文件用于配置环境变量和启动程序，详细介绍可参照：<a href="https://www.cnblogs.com/kevingrace/p/8072860.html">环境变量</a>

在文件中添加设置环境变量的命令：

> ```text
> export JAVA_HOME=/Users/shmily/cmower/save/apache-maven-3.8.3
> export PATH=${PATH}:${JAVA_HOME}/bin
> ```

保存后退出，可以执行 `source ~/.bash_profile` 使配置生效

第四步，查看配置是否生效

重新打开 `cmd` 输入 `mvn -v` 命令 出现maven版本就说明安装成功了

### 二、配置仓库镜像和本地仓库

**1. 第三方镜像配置**

```xml
<!-- settings.xml文件里  将默认地址改为阿里云 -->
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
  </mirror>
```

**2. 本地仓库配置**

```xml
<!--将仓库(maven_repository)放入maven中  再打开settings.xml找到本地仓库配置修改为本地仓库 -->
<localRepository>D:/apache-maven-3.8.3/maven_repository</localRepository>
<!-- maven_repository 可以自己创建一个文件夹 idea集成maven会自动生成需要的文件 -->
```

**3. 仓库服务搜索**

推荐 2 个提供仓库搜索服务的网站：

> - Sonatype Nexus：[https://repository.sonatype.org/open in new window](https://repository.sonatype.org/)
> - MVNrepository：http://mvnrepository.com/

### 三、使用 Maven

**1）Maven 常见命令**

- `mvn clean`：表示运行清理操作（会默认把target文件夹中的数据清理）。
- `mvn clean compile`：表示先运行清理之后运行编译，会将代码编译到target文件夹中。
- `mvn clean test`：运行清理和测试。
- `mvn clean package`：运行清理和打包。
- `mvn clean install`：运行清理和安装，会将打好的包安装到本地仓库中，以便其他的项目可以调用。
- `mvn clean deploy`：运行清理和发布（发布到私服上面）。
- `mvn help:effective-settings`：查看 Maven 的有效配置信息。

**2）Maven 常用 POM 属性**

- `${project.build.sourceDirectory}`：项目的主源码目录，默认为`src/main/java/`
- `${project.build.testSourceDirectory}`：项目的测试源码目录，默认为 `/src/test/java/`
- `${project.build.directory}`：项目构建输出目录，默认为 `target/`
- `${project.build.outputDirectory}`：项目主代码编译输出目录，默认为 `target/classes/`
- `${project.build.testOutputDirectory}`：项目测试代码编译输出目录，默认为 `target/testclasses/`
- `${project.groupId}`：项目的 groupId.
- `${project.artifactId}`：项目的 artifactId.
- `${project.version}`：项目的 version，于 `${version}` 等价
- `${project.build.finalName}`：项目打包输出文件的名称，默认为`${project.artifactId}${project.version}`

**3）Intellij IDEA 配置 Maven**

![image-20230424160740337](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Maven/Maven004.png?raw=true)

### 四、配置文件解析

Maven 是基于 POM（Project Object Model） 进行的，项目的所有配置都会放在 pom.xml 文件中，包括项目的类型、名字，依赖关系，插件定制等等。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itwanger</groupId>
    <artifactId>MavenDemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>MavenDemo</name>
</project>
```

- 第一行是XML头，指定了该xml文档的版本和编码方式。
- project 是根元素，声明了一些POM相关的命名空间及xsd元素。
- modelVersion指定了当前POM的版本，对于Maven 3来说，值只能是4.0.0。
- groupId定义了项目属于哪个组织，通常是组织域名的倒序。
- artifactId定义了项目在组织中的唯一ID。
- version指定了项目当前的版本，SNAPSHOT意为快照，说明该项目还处于开发中。
- name 声明了一个对于用户更为友好的项目名称。

groupId、artifactId和version这三个元素定义了一个项目的基本坐标，在Maven的世界里，任何的jar和pom都是以基于这些坐标进行区分的。

```text
<dependencies>
    <dependency>
        <groupId>实际项目</groupId>
　　　　 <artifactId>模块</artifactId>
　　　　 <version>版本</version>
　　　　 <type>依赖类型</type>
　　　　 <scope>依赖范围</scope>
　　　　 <optional>依赖是否可选</optional>
　　　　 <!--主要用于排除传递性依赖-->
　　　　 <exclusions>
　　　　     <exclusion>
　　　　　　　    <groupId>…</groupId>
　　　　　　　　　 <artifactId>…</artifactId>
　　　　　　　</exclusion>
　　　　 </exclusions>
　　</dependency>
<dependencies>
```

- dependencies 可以包含一个或者多个dependency元素，以声明一个或者多个项目依赖。
- grounpId、artifactId和version 组成了依赖的基本坐标。
- type 指定了依赖的类型，默认为 jar。
- scope 指定了依赖的范围（详情见下面**依赖范围**部分）。
- optional 标记了依赖是否是可选的（详情见下面**依赖可选**部分）。
- exclusions 用来排除传递性依赖（详情见下面**依赖排除**部分）。

**依赖范围**有以下几种：

- compile，默认的依赖范围，表示依赖需要参与当前项目的编译，后续的测试、运行周期也参与其中，是比较强的依赖。
- test，表示依赖仅仅参与测试相关的工作，包括测试代码的编译和运行。比较典型的如 junit。
- runntime，表示依赖无需参与到项目的编译，不过后期的测试和运行需要其参与其中。
- provided，表示打包的时候可以不用包进去，别的容器会提供。和 compile 相当，但是在打包阶段做了排除的动作。
- system，从参与程度上来说，和 provided 类似，但不通过 Maven 仓库解析，可能会造成构建的不可移植，要谨慎使用。

![image-20230424161327352](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Maven/Maven005.png?raw=true)

关于**传递性依赖**：

比如一个account-email项目为例，account-email有一个compile范围的spring-code依赖，spring-code有一个compile范围的commons-logging依赖，那么commons-logging就会成为account-email的compile的范围依赖，commons-logging是account-email的一个传递性依赖：

![img](Maven-img/maven-10.png)

有了传递性依赖机制，在使用Spring Framework的时候就不用去考虑它依赖了什么，也不用担心引入多余的依赖。Maven会解析各个直接依赖的POM，将那些必要的间接依赖，以传递性依赖的形式引入到当前的项目中。

关于**依赖排除**：

有时候你引入的依赖中包含你不想要的依赖包，你想引入自己想要的，这时候就要用到排除依赖了，比如下图中spring-boot-starter-web自带了logback这个日志包，我想引入log4j2的，所以我先排除掉logback的依赖包，再引入想要的包就行了

```text
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<version>2.5.6</version>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!-- 使用 log4j2 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-log4j2</artifactId>
	<version>2.5.6</version>
</dependency>
```

声明exclustion的时候只需要groupId和artifactId，不需要version元素，因为groupId和artifactId就能唯一定位某个依赖。