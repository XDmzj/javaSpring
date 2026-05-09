# Spring框架


首先一定要明确，使用Spring首要目的是为了使得软件项目进行解耦，而不是为了去简化代码！通过它，就可以更好的对我们的Bean进行管理，这一部分我们来体验一下Spring的基本使用。

Spring并不是一个独立的框架，它实际上包含了很多的模块：
![[Pasted image 20260509131445.png]]


而我们首先要去学习的就是Core Container，也就是核心容器模块，只有了解了Spring的核心技术，我们才能真正认识这个框架为我们带来的便捷之处。


# 使用


## IoC的配置文件设置

这里我们就来尝试编写一个最简的Spring项目，我们在前面已经讲过了，Spring会给我们提供IoC容器用于管理Bean，但是我们得先为这个容器编写一个配置文件，我们可以通过配置文件告诉容器需要管理哪些Bean以及Bean的属性、依赖关系等等。

首先我们需要在resource中创建一个Spring配置文件（在resource中创建的文件，会在编译时被一起放到类路径下），命名为test.xml，直接右键点击即可创建：

xml复制代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```



## IoC使用举例

Spring为我们提供了一个IoC容器，用于去存放我们需要使用的对象，我们可以将对象交给IoC容器进行管理，当我们需要使用对象时，就可以向IoC容器去索要，并由它来决定给我们哪一个对象。


### 创建上下文

而我们如果需要使用Spring为我们提供的IoC容器，那么就需要创建一个应用程序上下文，它代表的就是IoC容器，它会负责实例化、配置和组装Bean：

```java
public static void main(String[] args) {
  	ApplicationContext是应用程序上下文的顶层接口，它有很多种实现，这里我们先介绍第一种
  	因为这里使用的是XML配置文件，所以说我们就使用 ClassPathXmlApplicationContext 这个实现类
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");  
    这里写上刚刚配置文件的名字
}
```



### 定义类

比如现在我们要让IoC容器帮助我们管理一个Student对象（Bean），当我们需要这个对象时再申请，那么就需要这样，首先先将Student类定义出来：

```java
package com.test.bean;

public class Student {
    public void hello(){
        System.out.println("Hello World!");
    }
}
```


### 注册bean

既然现在要让别人帮忙管理对象，那么就不能再由我们自己去new这个对象了，而是编写对应的配置，我们打开刚刚创建的`test.xml`文件进行编辑，添加：

```java
<bean name="student" class="com.test.bean.Student"/>
```

这里我们就在配置文件中编写好了对应Bean的信息，之后容器就会根据这里的配置进行处理了。


### 从IoC申请对象

现在，这个对象不需要我们再去创建了，而是由IoC容器自动进行创建并提供，我们可以直接从上下文中获取到它为我们创建的对象：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
    Student student = (Student) context.getBean("student");   
    使用getBean方法来获取对应的对象（Bean）
    student.hello();
}
```

实际上，这里得到的Student对象是由Spring通过反射机制帮助我们创建的，

