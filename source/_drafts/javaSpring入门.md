---
title: javaSpring入门
categories:
 - [Java, Spring]
 - [Spring, Spring入门]
tags:
- Spring
- Java
mathjax: true
---



## Spring简介

---

{% link Spring官网, Spring.io%}

{% note quote cyan, Spring框架是Java应用最广的框架，他的成功来源于理念，而不是技术本身，她的理念包括IoC(Inversion of Control,控制反转)和AOP(Aspect Oriented Programming,面向切面编程) %}

## Spring常用术语

---

> 框架：是能完成一定功能的半成品

> 非侵入式设计

从框架的角度可以理解为：无需继承框架提供的任何类

这样我们在更换框架时，之前写过的代码几乎可以继续使用

> 轻量级与重量级

轻量级一般具有非入侵、依赖的东西更少，资源占用更少、部署简单等优点。



> JaveBean

是一种Java语言写成的可重用组件，为写成JavaBean，类必须是具体和公有的，并且有无参构造器。JavaBean通过提供符合一致性设计模式的公共方法将内部域暴露成员属性。更多的是一种规范，即包含一组get和set方法的java对象。

> Pojo

简单的Java对象，实际就是普通JavaBeans。

> entity

实体Bean，一般用于ORM对象关系映射，一个实体映射成一张表，一般无业务逻辑代码

## Spring优势

---

1. 低侵入/低耦合（降低组件之间的耦合度，	实现软件各层之间的解耦）
2. 声明式事务管理（基于切面和惯例）
3. 方便集成其它框架（如Mybatis、Hibernate
4. 降低Java开发难度
5. Sprong框架中包含了J2EE三层的每一层的解决方案（一站式）

### Spring框架

![这里写图片描述](F:\Enderblog\source\img\Spring框架.gif)

### 工厂设计模式

---

工厂模式分为简单宫村模式，工厂方法模式和抽象工厂模式，它们都属于设计模式中的创建型模式。其主要功能都是帮助我们把`对象的实例化`部分抽取出来，目的是降低系统中代码耦合度，并且增强了系统的扩展性。

#### 简单工厂设计模式

---

优点：实现对象的创建和对象的使用分离，将对象的创建交给专门的工厂类负责

缺点：工厂类不够灵活，增加新的具体产品需要修改工厂类的判断逻辑代码，而且产品较多时，工厂方法代码将会非常复杂

```java
/**
 * @author Ender-PC
 */
public interface Car {
    void run();
}
```

```java
/**
 * @author Ender-PC
 */
public class BMW implements Car{

    @Override
    public void run() {
        System.out.println("宝马启动！");
    }
}
```

```java
/**
 * @author Ender-PC
 */
public class Benz implements Car{
    @Override
    public void run() {
        System.out.println("奔驰启动！");
    }
}
```

```java
/**
 * @author Ender-PC
 */
public class Bike implements Car{
    @Override
    public void run() {
        System.out.println("没办法只能自行车了");
    }
}
```

```java
/**
 * @author Ender-PC
 */
public class CarFactory {
    /**
     * 根据对象名获取实列对象的方法
     * @param name
     * @return
     */
    public Car getCar(String name){
        if("bmw".equalsIgnoreCase(name)){
            //其中可能有很复杂的操作
            return new BMW();
        } else if("benz".equalsIgnoreCase(name)){
            //其中可能有很复杂的操作
            return new Benz();
        }
        return new Bike();
    }
}
```

```java
/**
 * @author Ender-PC
 */
public class Test {
    public static void main(String[] args) {
        CarFactory carFactory = new CarFactory();
        Car bmw = carFactory.getCar("BMW");
        Car benz = carFactory.getCar("Benz");
        Car honda = carFactory.getCar("Honda");
        bmw.run();
        benz.run();
        honda.run();
    }
}
```

#### 工厂方法模式

---

Java开发应遵循开闭原则（*对于扩展是开放的，但是对于修改是封闭的*），如果我们想要新增一种新车，那么必须修改CarFactory，就不太灵活。解决方案是使用工厂方法模式。

我们为每种车都构建一个工厂：

> 先抽象一个工厂接口

```java
/**
 * @author Ender-PC
 */
public interface Factory {
    /**
     * 统一构造方法
     * @return
     */
    Car getCar();
}
```

```java
/**
 * @author Ender-PC
 */
public class BenzFactory implements Factory{

    @Override
    public Car getCar() {
        return new Benz();
    }
}
```

```java
/**
 * @author Ender-PC 
 */
public class BMWFactory implements Factory{
    @Override
    public Car getCar() {
        return new BMW();
    }
}
```

```java
/**
 * @author Ender-PC
 */
public class Test {
    public static void main(String[] args) {
        BMWFactory bmwFactory = new BMWFactory();
        Car bmw = bmwFactory.getCar();
        bmw.run();
        BenzFactory benzFactory = new BenzFactory();
        Car benz = benzFactory.getCar();
        benz.run();
    }
}
```

##### 优点

---

此模式通过定义一个抽象的核心工厂类，并定义创建产品对象的接口，创建具体产品实列的工作延迟到其工厂子类完成。具有如下优点：

1. 核心类值关注工厂类的接口定义，而具体的产品实列交给具体的工厂子类创建
2. 当系统需要新增一个产品时，无需修改现有代码，只需要添加一个具体产品和其对应的子类，使系统的拓展性变得很好，符合面向对象编程的开闭原则

##### 缺点

---

增加了编码难度，大量增加了类的数量

## IOC容器

---

### 控制反转IOC

1. 控制反转是一种思想，就是将原本在程序种手动创建对象的控制权，交由Spring框架来管理
2. 以往的思路：若要使用某个对象，需要自己负责对象的创建
3. 反转的思想：若要使用某个对象，只需要从Spring容器中获取需要使用的对象，不关心对象的创建过程，以就是把创建对象的控制权反转给了Spring框架

{% note quote blue,好莱坞法则：Don't call me,I'll call you %}

### 举例说明

下面引用一个例子来说明什么是控制反转：

{% note quote blue, 过年了，今天楠哥（哪里都是张三）需要给家里打扫个卫生，于是想请几个钟点工来擦擦玻璃。 解决方案有几种： 

1. 自己主动的去，询问查找，看看谁认识钟点工，有了联系方式后，自己打电话邀约，谈价钱。 
2. 直接打电话给家政公司，提出要求即可。 

第一种方式，就是我们之前接触的创建对象的方式，自己主动new，而第二种就是spring给我们提供的 另外一种方式叫IOC，家政公司就像是一个大容器，能为我们提供很多的服务。 又过了几天，楠哥想清理一下油烟机，直接打电话给家政公司，师傅很快到达，帮助楠哥清理了油烟 机。 聊聊例子中的问题 一定会有人问，那家政公司从哪里来啊！ %}

#### 例子中的问题

那么就有人问了，家政公司从何而来

1. 自行构建

我们可以使用配置我呢见，或者注解的方式定义一下我们自己的容器里存放的东西

2. 用别人的

一定会有很多有钱人，成立了自己的各类公司，他们的这些服务都可以集成在我们的容器里，为我们提供很多强大的功能，比如Spring自带的就很多template模板类

#### 尝试

##### 引入依赖

加入Spring framework

{% folding cyan, 点击查看XML %}

```xml
<properties>
    <spring.version>5.2.12.RELEASE</spring.version>
    <mysql.connector.java.version>8.0.21</mysql.connector.java.version>
    <druid.version>1.1.18</druid.version>
    <aopalliance.version>1.0</aopalliance.version>
    <aopalliance.version>1.0</aopalliance.version>
    <aspectjweaver.version>1.9.2</aspectjweaver.version>
    <junit.version>4.12</junit.version>
    <lombok.version>1.16.18</lombok.version>
</properties>
<dependencies>
    <!-- Spring的核心组件 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- SpringIoC(依赖注入)的基础实现 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!--Spring提供在基础IoC功能上的扩展服务，此外还提供许多企业级服务的支持，如邮
件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- Spring表达式语言 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-expression</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!--AspectsJ：AOP-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- Spring提供对AspectJ框架的整合 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- Spring提供对持久层的末班方法 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- spring聚成测试环境 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 支持切入点表达式等等 -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>${aspectjweaver.version}</version>
    </dependency>
    <!-- 是AOP联盟的API包,里面包含了针对面向切面的接口。 通常Spring等其它具备动态
织入功能的框架依赖此包。 -->
    <dependency>
        <groupId>aopalliance</groupId>
        <artifactId>aopalliance</artifactId>
        <version>${aopalliance.version}</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid.version}</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.connector.java.version}</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source> <!-- 源代码使用的JDK版本 -->
                <target>1.8</target> <!-- 需要生成的目标class文件的编译版本
-->
                <encoding>UTF-8</encoding><!-- 字符集编码 -->
            </configuration>
        </plugin>
    </plugins>
</build>
```

{% endfolding %}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
<!--以上用于约束Spring配置文件的xml文件-->
    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

