---
title: javaMaven
categories:
  - - Java
    - Maven
tags:
  - Maven
  - Java
date: 2021-01-31 15:56:12
---

java工具——包管理工具Maven

<!-- more -->

## Maven

### 优点

1. 统一管理jar包，自动导入jar及其以来依赖
2. 项目移植之后甚至不需要安装并发工具，只需要maven加命令就能跑，降低学习成本
3. 使得项目流水线成为可能，因为使用简单的命令我们就能完成项目的编译，打包，发布等工作，就让程序操作程序成为了可能，大名鼎鼎的jekins也能做到这一点

### Maven下载安装

---

{% note download, [Maven官网](http://maven.apache.org/) %}

下载binary文件

#### Maven安装

---

1. 解压
2. 配置MAVEN_HOME，为maven的解压
3. 配置path，%MAVEN_HOME%\bin
4. 使用如下命令验证是否配置成功

```shell
mvn -v
```

#### Maven核心全局配置文件

---

```java
\apache-maven-3.6.3\conf\settings.xml
```

##### 配置路径

---

选择一个位置新建文件夹repository，setting.xml中添加如下标记

```xml
<localRepository>D:\program\Maven\repository</localRepository>
```

##### 配置国内镜像

---

```xml
<mirror>
	<id>alimaven</id>
	<name>aliyun maven</name>
  	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	<mirrorOf>central</mirrorOf>
</mirror>
```

##### 配置全局编译jdk版本

---

当计算机中有多个版本的jdk时，以下配置是必须的

```xml
<profile>
    <id>jdk-11.0.9</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>11</jdk>
    </activation>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
         <maven.compiler.target>11</maven.compiler.target>
        <maven.compiler.compilerVersion>11</maven.compiler.compilerVersion>
    </properties>
</profile>
```



#### Maven体验

---

#### Maven标准目录

---

```
src
	|--main
		|--java			源代码目录
		|--resources	资源目录
 	|--test
		|--java			测试代码目录
		|--resources	测试资源目录
 	|--target
		|--ckasses		编译后的class文件目录
		|--test-classes	编译后的测试class文件目录
pom.xml			Maven工程配置文件
```

#### pom.xml基本内容

---

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <!-- 模型版本 -->
    <modelVersion>4.0.0</modelVersion>
    <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.companyname.project-group，maven会将该项目打成的jar包放本地路径：/com/companyname/project-group -->
    <groupId>com.companyname.project-group</groupId>
 
    <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
    <artifactId>project</artifactId>
 
    <!-- 版本号 -->
    <version>1.0</version>
</project>
```

在创建好上述项目结构并配置好maven以及项目pom后，在项目根目录使用如下命令就能利用maven，进行自动的依赖下载，以及项目编译:

```shell
mvn compile
```

![maven编译项目.png](https://i.loli.net/2021/01/31/KOq2WEoYfb6kAig.png)

### Maven生命周期

---

maven生命周期描述了一个项目从源代码到部署的整个周期

Maven有三个内置的生命周期：

1. 清理（clean）：为执行一下工作做必要的清理，如删除target文件夹
2. 默认（default）：真正进行项目编译打包工作的阶段
3. 站点（site）：生成项目报告，站点，发布站点

默认生命周期包括以下阶段（该阶段经过简化，实际上更加复杂）：

1. 验证（validate）：验证项目是否正确，所有必要信息是否可用
2. **编译**（compile）：编译项目的源代码
3. 测试（test）：使用合适的单元测试框架测试编译的源代码。这些测试不应该要求代码被打包或部署
4. **打包**（package）：采用编译的代码，并以其可分配格式（如JAR）进行打包
5. 验证（verify）：对集成测试的结果执行任何检查，以确保满足质量标砖
6. **安装**（install）：将软件包安装到本地存储中，用作本地其他项目的依赖项
7. 部署（deploy）：在构建环境中完成，将最终的包复制到远程存储以与其他开发人员和项目共享（私服）

### Maven的版本规范

---

所有软件都有版本

Maven使用如下几个要素来定位一个项目，因此它们又称为项目的坐标。

1. `groupId`：团体、组织的标识符。团体标识的约定量，它以创建这个项目的组织名称的{% wavy,逆向域 %}名开头。一般对应着JAVA的包的结构，例如org.apache。
2. `artifactId`：单独项目的唯一标识符。比如tomcat，commons等。不要再其中使用{% kbd . %}
3. `version`：项目版本
4. `packaging`：项目的类型，默认是jar，描述了项目打包后的输出，类型为jar的项目产生一个JAR文件，类型为war的项目产生一个web应用。

Maven在版本管理时可以使用几个特殊的字符串 `SNAPSHOT`，`LARESR`，`RELEASE`。比如“1.0-SNAPSHOT”。各个部分的含义和处理逻辑如下说明：

- `SNAPSHOT` 这个版本一般用于开发过程中，表示不稳定版
- `LARESR` 指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个snapshot版，具体看那个时间最后
- `RELEASE` 指最后一个发布版

### Idea配置Maven

---

#### 为当前项目配置

---

File $\to$ Settings $\to$ 搜索Maven $\to$ `Maven home path`、`User settings file`、`Local repository`

三项分别设置为：

1. Maven安装路径
2. conf/settings.xml的路径
3. repository文件夹的路径

#### 为未来项目配置

---

New Project Settings $\to$ Settings for New Projects... $\to$ 搜索Maven $\to$ `Maven home path`、`User settings file`、`Local repository`

配置同上

### Maven依赖

---

> Maven管理依赖也就是jar包不用我们自己下载，会从一些地方自动下载

{% link maven远程仓库, https://mvnrepository.com/ %}

{% link maven远程仓库, https://maven.aliyun.com/mvn/search %}

> Maven 工程中我们依赖在pom.xml文件进行配置完成jar包管理工作（依赖）

在工程中引入某个jar包，只需要在pom.xml中引入jar包的坐标，比如引入log4j的依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.7</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

`Maven`通过`groupId`、`artifactId`与`version`三个向量来定位Maven仓库其jar包所在的位置，并把对应的jar包引入到工程中来

#### 依赖范围

---

##### classpath

---

编译好的`class`文件所在的路径

事实上，类加载器（`classloader`）就是去对应的`classpath`中加载`class`二进制文件

##### maven项目

---

maven工程会将`src/main/java`和`src/main/recources`文件夹下的文件全部打包在classpath中。运行时它们两个文件夹下的文件会被放在一个文件夹下。

maven项目不同的阶段引入到classpath中的依赖是不同的例如

1. `编译时`，maven会将与编译相关的依赖引入classpath中
2. `测试时`，maven会将与测试相关的依赖引入classpath中
3. `运行时`，maven会将与运行相关的依赖引入classpath中

而依赖范围就是用来控制依赖于这三种classpath的关系

##### scop标签

---

scop标签就是依赖范围的配置

该项默认配置为compile，可选配置还有test、provide、runtime、system、import

其中compile、test和provided使用较多

部分jar包指在某一特定时候需要被加载，例如：

1. `servlet-api`，运行时其实是不需要的，因为tomcat里有，但编译时需要，因为编译时没有tomcat环境
2. `junit`，只有在测试的时候才能用到，运行时不需要
3. `JDBC`，测试时必须要有，编译时不需要，编译时用的都是jdk中的接口，运行时我们才使用反射注册驱动

###### 编译依赖范围（compile）

---

该范围是默认范围，次依赖范围对于编译、测试、运行三种`classpath`都有效如

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.68</version>
</dependency>
```

###### 测试依赖范围（test）

---

指对测试classpath有效，编译运行时都无法使用依赖，如junit

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.7</version>
    <scop>test</scop>
</dependency>
```

###### 已提供依赖范围（provided）

---

支队编译和测试的classpath有效，对运行的classpath无效，如`servlet-api`，如果不设置依赖范围，当容器依赖的版本和maven依赖的版本不一致时会引起冲突

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scop>provided</scop>
</dependency>
```

###### 运行时依赖范围（runtime）

---

只对测试和运行的classpath有效，对编译的classpath无效，如`JDBC`驱动

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connection-java</artifactId>
    <version>5.1.25</version>
    <scop>runtime</scop>
</dependency>
```

#### 依赖传递

---

jar包也是别人写的工程项目，她们也会依赖其他的jar包，传递性让我们可以不用关心我们所依赖的jar包依赖了哪些jar，只要我们添加了依赖，他会自动将所依赖的jar统统依赖进来

##### 依赖传递原则

---

- **最短路径优先原则**：如果A依赖了B，B依赖了C，在B和C	中同时依赖了log4j的依赖，并且这两个版本不一致，那么A会根据最短路径原则，在A中会传递过来B的log4j版本

![最短路依赖.png](https://i.loli.net/2021/01/31/guFL2vx59BshnpQ.png)

- **路径相同先声明原则**：如果我们的工程同时依赖于B和A，B和C没有依赖关系，并且都有D的依赖，且版本不一致，那么会引入在pom.xml中先声明依赖的log4j版本。

![路径相同先声明.png](https://i.loli.net/2021/01/31/uwW7VnLve8J3ZlI.png)

```xml
<dependency>
    <groupId>com.ender</groupId>
    <artifactId>B</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>com.ender</groupId>
    <artifactId>A</artifactId>
    <version>1.5</version>
</dependency>
```

因为D的1.2.3版本的依赖关系优先得到确定，所以依赖D的1.2.3，因此如果A包中使用了D1.3.2的某些新特性，可能造成A包无法使用的问题，于是就需要把低版本排除，一般高版本会兼容低版本

#### 依赖的排除

---

对于上例中，我们如果想把低版本的D包排除，就可以做如下设置

```xml
<dependency>
    <groupId>com.ender</groupId>
    <artifactId>B</artifactId>
    <version>1.2</version>
    
    
    <exclusions>
        <exclusion>
            <artifactId>com.ender</artifactId>
            <groupId>D</groupId>
        </exclusion>
    </exclusions>
    
    
</dependency>
<dependency>
    <groupId>com.ender</groupId>
    <artifactId>A</artifactId>
    <version>1.5</version>
</dependency>
```

### 聚合和继承

---

分布式项目必须使用到该功能

**聚合模块（父模块）**的打包方式必须时pom，否则无法完成构建

在聚合多个项目时，如果这些聚合的项目中需要引入相同的jar，那么可以将这些jar写入父pom中，各个子项目集成该pom即可。父模块的打包方式必须为pom，否则无法构建项目

通过修改pom.xml来表明继承关系：

父模块pom：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>maven-test</artifactId>
    <version>1.0-SNAPSHOT</version>
    
<!--    子模块-->
    <modules>
        <module>child-one</module>
    </modules>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
<!--    打包方式-->
    <packaging>pom</packaging>

<!--    依赖-->
    <dependencies>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.1</version>
        </dependency>
    </dependencies>
</project>
```

子模块POM：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<!--    依赖关系-->
    <parent>
        <artifactId>maven-test</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>child-one</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

</project>
```

可被继承的POM元素如下：

1. groupId:项目子ID，项目坐标的核心元素
2. version:项目版本，项目坐标的核心元素
3. properties：自定义的Maven属性，一般用于统一制定各个依赖的版本好
4. dependencies
5. dependencyManagement：项目依赖管理配置
6. repositories
7. build：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等

#### 实现父子工程统一版本管理

---

对于一个父子嵌套的工程，当我们要对所有子工程进行统一管理时，常常需要所有子工程使用统一的插件版本，我们可能会使用

```xml
<dependencies>
    <dependency>
        <groupId>commons-lang</groupId>
        <artifactId>commons-lang</artifactId>
        <version>2.1</version>
    </dependency>
</dependencies>
```

这样的方式在父工程的pom中配置，但是这样在打包时，子工程无论用没用到这个插件，都会将其打包进该工程中，因此我们需要使用别的方法进行版本管理：

首先在父工程的pom中配置：

```xml
<!-- 定义常量进行统一版本管理 -->
<properties>
    <fastjson-version>1.2.68</fastjson-version>
</properties>

<!-- 此处引用的插件在子工程中服务立马生效，只有当子工程声明使用该插件时才能生效 -->
<dependenciesManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson-version}</version>
        </dependency>
    </dependencies>
</dependenciesManagement>
```

子工程pom中配置：

```xml
<!-- 子工程中声明时不需要声明版本号，将自动从父工程中的插件里得到版本号 -->
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>
</dependencies>
```



### POM文件

---

#### 基础配置

---

一个典型的pom.xml文件配置如下：

{% folding, 点击查看配置%}

```xml
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <!-- 模型版本 -->
    <modelVersion>4.0.0</modelVersion>
    <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.companyname.project-group，maven会将该项目打成的jar包放本地路径：/com/companyname/project-group -->
    <groupId>com.companyname.project-group</groupId>
 
    <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
    <artifactId>project</artifactId>
 
    <!-- 版本号 -->
    <version>1.0.0-SNAPSHOT</version>
    <!-- 打包机制,如pom、jar、war，默认为jar-->
    <packaging>jar</packaging>
    
    <!-- 为pom定义一些商量，在pom中的其他地方可以直接引用 使用方式如下：${file.encoding} -->
    <!-- 常用来整体控制一些依赖的版本号 -->
    <properties>
        <file.encoding>UTF-8</file.encoding>
        <java.source.version>1.8</java.source.version>
        <java.target.version>1.8</java.target.version>
    </properties>
    
    <!-- 定义本项目的依赖关系，就是依赖的jar包 -->
    <dependencies>
        <!-- 每个dependency都对应一个jar包 -->
        <dependency>
            <!-- 一般情况下，maven是通过groupId、artifactId、version三个元素指（俗称坐标）来检索该构件，然后引入你的工程。如果别人想引用你现在开发的这个目录（前提是已开发完毕并发布到了远程仓库） -->
            <!-- 就需要在她的pom文件中新建一个dependency节点，将本项目的groupId、artifactId、version写入，maven就会把你上传的jar包下载到她的本地 -->
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            
            <!-- 依赖范围 -->
            <scop>compile</scop>
            <!-- 设置 依赖是否可选，默认为false，即子项目默认都继承，如果为true，则子项目必须显式的引入 -->
            <optional>false</optional>
            <!-- 排除依赖 -->
            <exclusions>
                <exclusion>
                	<groupId>commons-lang</groupId>
            		<artifactId>commons-lang</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</project>
```

{% endfolding %}

一般来说，上面的几个配置项对任何项目都是必不可少的，定义了项目的基本属性

#### 构建配置

---

{% folding, 点击查看配置%}

```xml
<build>
    <!-- 产生的构建的文件名，默认值为${artifactId}-${version} -->
    <finalName>myProjectName</finalName>
    <!-- 构建产生的所有文件存放发目录，默认为${basedir}/target,即项目根目录下的target -->
    <directory>${basedir}/target</directory>
    <!-- 项目相关的所有资源路径列表，例如和项目相关的配置文件、属性文件，这些资源被包含在最终的打包文件里 -->
    <!-- 项目源码目录，当构建项目的时候，构建系统会编译目录里的源码，该路径是相对于pom.xml的相对路径 -->
    <sourceDirectory>${basedir}\src\main\java</sourceDirectory>
    <!-- 项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码，该路径是相对于pom.xml的相对路径 -->
    <testSourceDirectory>${basedir}\src\test\java</testSourceDirectory>
    <!-- 被编译过的应用程序class文件存放的目录 -->
    <outputDirectory>${basedir}\target\classes</outputDirectory>
    
    <!-- 被编译过的测试class文件存放的目录 -->
    <testOutputDirectory>${basedir}\target\test-classes</testOutputDirectory>
    <!-- 以上配置遵循约定大于配置原则，一般使用默认配置 -->
    
    <!-- 自行定义资源目录 -->
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <inlcudes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </inlcudes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <inlcudes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </inlcudes>
            <filtering>false</filtering>
        </resource>
    </resources>
    
    <!-- 单元测试相关的所有资源路径，配置方法于resources类似 -->
    <testResources>
        <testResource>
            <targetPath />
            <filtering />
            <directory />
            <includes />
            <excludes />
        </testResource>
    </testResources>
    
    <!-- 使用的插件列表 -->
    <plugins>
        <plugin>
            ...具体在插件使用中了解
        </plugin>
    </plugins>
    
    <!-- 主要定义插件的共同元素、拓展元素集合，类似于dependencyManagement -->
    <!-- 所有继承于次项目的子项目都能使用。该插件配置项直到被引用时才会被解析或绑定到生命周期。 -->
    <!-- 给定插件的任何本地配置都会覆盖这里的配置 -->
    <pluginManagement>
        <plugins>...</plugins>
    </pluginManagement>
</build>
```

{% endfolding %}

###### 常用的几个配置

---

处理资源被过滤的问题

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <inlcudes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </inlcudes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <inlcudes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </inlcudes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

添加本地jar包

```xml
<!-- geelynote maven的核心插件-compiler插件默认只支持编译Java1.4，因此需要加上支持高版本jre的配置，在 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
        <compilerArguments>
            <!-- 本地jar，支付宝jar包放到 src/main/webapp/WEB-INF/lib 文件夹下，如果没有配置，本地不会有问题，但线上会找不到sdk类，为什么要引入，因为支付宝jar包在中央仓库中没有，再比如oracle连接驱动的jar -->
            <extdirs>${project.basedir}/src/main/webapp/WEB-INF/lib</extdirs>
        </compilerArguments>
    </configuration>
</plugin>
```

#### 仓库配置

---

```xml
<repositories>
    <repossitory>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snaphosts>
            <enabled>false</enabled>
        </snaphosts>
    </repossitory>
</repositories>
```

pom.xml中的仓库和setting.xml里的仓库功能一样，区别在于pom里配置的仓库时个性化的，比如，公司里的settings文件时公用的，若有项目都用一个settings文件，但各个子项目却会引用不同的第三方库，所以需要正在pom.xml里设置自己需要的仓库地址。

#### 项目信息配置（了解）

---

{% folding, 点击查看配置%}

```xml
<!--项目的名称, Maven产生的文档用 -->
<name>banseon-maven</name>
<!--项目主页的URL, Maven产生的文档用 -->
<url>http://www.baidu.com/banseon</url>
<!-- 项目的详细描述, Maven 产生的文档用。 当这个元素能够用HTML格式描述时（例如，CDATA中的文本会被解析器忽略，就可以包含HTML标 
        签）， 不鼓励使用纯文本描述。如果你需要修改产生的web站点的索引页面，你应该修改你自己的索引页文件，而不是调整这里的文档。 -->
<description>A maven project to study maven.</description>
<!--描述了这个项目构建环境中的前提条件。 -->
<prerequisites>
    <!--构建该项目或使用该插件所需要的Maven的最低版本 -->
    <maven />
</prerequisites>
<!--项目的问题管理系统(Bugzilla, Jira, Scarab,或任何你喜欢的问题管理系统)的名称和URL，本例为 jira -->
<issueManagement>
    <!--问题管理系统（例如jira）的名字， -->
    <system>jira</system>
    <!--该项目使用的问题管理系统的URL -->
    <url>http://jira.baidu.com/banseon</url>
</issueManagement>
<!--项目持续集成信息 -->
<ciManagement>
    <!--持续集成系统的名字，例如continuum -->
    <system />
    <!--该项目使用的持续集成系统的URL（如果持续集成系统有web接口的话）。 -->
    <url />
    <!--构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告） -->
    <notifiers>
        <!--配置一种方式，当构建中断时，以该方式通知用户/开发者 -->
        <notifier>
            <!--传送通知的途径 -->
            <type />
            <!--发生错误时是否通知 -->
            <sendOnError />
            <!--构建失败时是否通知 -->
            <sendOnFailure />
            <!--构建成功时是否通知 -->
            <sendOnSuccess />
            <!--发生警告时是否通知 -->
            <sendOnWarning />
            <!--不赞成使用。通知发送到哪里 -->
            <address />
            <!--扩展配置项 -->
            <configuration />
        </notifier>
    </notifiers>
</ciManagement>
<!--项目创建年份，4位数字。当产生版权信息时需要使用这个值。 -->
<inceptionYear />
<!--项目相关邮件列表信息 -->
<mailingLists>
    <!--该元素描述了项目相关的所有邮件列表。自动产生的网站引用这些信息。 -->
    <mailingList>
        <!--邮件的名称 -->
        <name>Demo</name>
        <!--发送邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <post>banseon@126.com</post>
        <!--订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <subscribe>banseon@126.com</subscribe>
        <!--取消订阅邮件的地址或链接，如果是邮件地址，创建文档时，mailto: 链接会被自动创建 -->
        <unsubscribe>banseon@126.com</unsubscribe>
        <!--你可以浏览邮件信息的URL -->
        <archive>http:/hi.baidu.com/banseon/demo/dev/</archive>
    </mailingList>
</mailingLists>
<!--项目开发者列表 -->
<developers>
    <!--某个项目开发者的信息 -->
    <developer>
        <!--SCM里项目开发者的唯一标识符 -->
        <id>HELLO WORLD</id>
        <!--项目开发者的全名 -->
        <name>banseon</name>
        <!--项目开发者的email -->
        <email>banseon@126.com</email>
        <!--项目开发者的主页的URL -->
        <url />
        <!--项目开发者在项目中扮演的角色，角色元素描述了各种角色 -->
        <roles>
            <role>Project Manager</role>
            <role>Architect</role>
        </roles>
        <!--项目开发者所属组织 -->
        <organization>demo</organization>
        <!--项目开发者所属组织的URL -->
        <organizationUrl>http://hi.baidu.com/banseon</organizationUrl>
        <!--项目开发者属性，如即时消息如何处理等 -->
        <properties>
            <dept>No</dept>
        </properties>
        <!--项目开发者所在时区， -11到12范围内的整数。 -->
        <timezone>-5</timezone>
    </developer>
</developers>
<!--项目的其他贡献者列表 -->
<contributors>
    <!--项目的其他贡献者。参见developers/developer元素 -->
    <contributor>
        <name />
        <email />
        <url />
        <organization />
        <organizationUrl />
        <roles />
        <timezone />
        <properties />
    </contributor>
</contributors>
<!--该元素描述了项目所有License列表。 应该只列出该项目的license列表，不要列出依赖项目的 license列表。如果列出多个license，用户可以选择它们中的一个而不是接受所有license。 -->
<licenses>
    <!--描述了项目的license，用于生成项目的web站点的license页面，其他一些报表和validation也会用到该元素。 -->
    <license>
        <!--license用于法律上的名称 -->
        <name>Apache 2</name>
        <!--官方的license正文页面的URL -->
        <url>http://www.baidu.com/banseon/LICENSE-2.0.txt</url>
        <!--项目分发的主要方式： repo，可以从Maven库下载 manual， 用户必须手动下载和安装依赖 -->
        <distribution>repo</distribution>
        <!--关于license的补充信息 -->
        <comments>A business-friendly OSS license</comments>
    </license>
</licenses>
<!--SCM(Source Control Management)标签允许你配置你的代码库，供Maven web站点和其它插件使用。 -->
<scm>
    <!--SCM的URL,该URL描述了版本库和如何连接到版本库。欲知详情，请看SCMs提供的URL格式和列表。该连接只读。 -->
    <connection>
        scm:svn:http://svn.baidu.com/banseon/maven/banseon/banseon-maven2-trunk(dao-trunk)
    </connection>
    <!--给开发者使用的，类似connection元素。即该连接不仅仅只读 -->
    <developerConnection>
        scm:svn:http://svn.baidu.com/banseon/maven/banseon/dao-trunk
    </developerConnection>
    <!--当前代码的标签，在开发阶段默认为HEAD -->
    <tag />
    <!--指向项目的可浏览SCM库（例如ViewVC或者Fisheye）的URL。 -->
    <url>http://svn.baidu.com/banseon</url>
</scm>
<!--描述项目所属组织的各种属性。Maven产生的文档用 -->
<organization>
    <!--组织的全名 -->
    <name>demo</name>
    <!--组织主页的URL -->
    <url>http://www.baidu.com/banseon</url>
</organization>
<!--构建项目需要的信息 -->
```

{% endfolding %}

其他配置信息参照：

{% link Maven POM教程, https://www.runoob.com/maven/maven-pom.html %}

### Maven仓库

---

任何一个构件都有唯一坐标，Maven根据这个坐标定位了构件在仓库中的唯一存储路径

Maven仓库分类两类：

1. 本地仓库
2. 远程仓库，远程仓库分为三种：
   1. 中央仓库
   2. 私服
   3. 其他公共仓库 

#### 本地仓库

---

本地仓库时Maven在本地存储构建的地方，在安装Maven时不会被创建，在第一次执行Maven命令时才被创建

Maven本地仓库的默认位置：用户目录下的.m2/repository/

还可在conf/settings.xml中修改本地库的位置

```xml
<!-- 本地仓库的路径，默认值为 -->
<localRepository>D:/myworkspace/Maven/repository</localRepository>
```



#### 远程仓库

---

用于获取其他人的Maven构件

##### 中央仓库

---

默认的远程仓库，Maven在安装时，自带的就是中央仓库的配置

所有的Maven都议会继承超级POM，超级POM中包含如下配置：

```xml
<repositories>
    <repository>
        <id>central</id>
        <name>Cntral Repository</name>
        <url>http://repo.maven.apache.org</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

中央仓库包含了绝大多数流行的开源Java构件，以及源码，作者信息，SCM，信息，许可信息等

还可以在里面配置优先使用的镜像，比如在国内直接连接中央仓库较慢，一般使用阿里的镜像

```xml
<!-- 镜像列表 -->
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <!-- 被镜像的服务器ID -->
        <mirrorOf>central</mirrorOf>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
</mirrors>
```

##### 私服

---

私服时一种特殊的远程仓库，它时架设在局域网的仓库服务，私服代理广域网上的远程仓库，供局域网的Maven用户使用。当Maven需要下载构件的时候，它从私服请求，如果私服上不存在该构件，则从外部的远程仓库中下载，缓存在私服上之后，再为Maven的下载请求提供服务

###### 优点

---

1. 加速构建
2. 节约带宽
3. 节约中央Maven仓库的带宽
4. 稳定（应对一旦中央服务器出问题的情况）
5. 可以建立本地内部仓库
6. 可以建立公共仓库



如果没有特殊需求，一般只需要将私服地址配置为镜像，同时配置其代理所有的仓库就可以实现通过私服下载依赖的功能。镜像配置如下：

```xml
<mirror>
    <id>Nexus Mirror</id>
    <name>Nexus Mirror</name>
    <url>http://localhost:8081/nexus/content/groups/public</url>
    <mirrorOf>*</mirrorOf>
</mirror>
```

可以使用Docker搭建Nexus私服，私服可以有用户名和密码，可以在settings.xml中配置

### Maven项目模板（了解）

---

Archetype时一个Maven插件，其任务是按照其模板来创建一个项目结构。

执行如下命令即可创建Maven项目模板

```shell
mvn archetype:generate
```

常用archetype有两种：

1. maven-archetype-quickstart默认的Archetype
2. maven-archetype-webapp

创建webapp可以使用如下命令

```shell
mvn archetype:generate -DgroupId=com.ender -DartifactId=seckill -DarchetypeArtifactId=maven-archetype-webapp
```

