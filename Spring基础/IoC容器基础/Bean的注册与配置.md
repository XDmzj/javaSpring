# Bean注册与属性配置

前面我们通过一个简单例子体验了一下如何使用Spring来管理我们的对象，并向IoC容器索要被管理的对象。这节课我们就来详细了解一下如何向Spring注册Bean以及Bean的相关配置。

## 多个配置文件
实际上我们的配置文件可以有很多个，并且这些配置文件是可以相互导入的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>
    <import resource="test.xml"/>
</beans>
```


## 配置一个bean

但是为了简单起见，我们还是从单配置文件开始讲起，首先我们需要知道如何配置Bean并注册。

要配置一个Bean，只需要添加：

xml复制代码

```xml
<bean/>
```

但是这样写的话，Spring无法得知我们要配置的Bean到底是哪一个类，所以说我们还得指定对应的类才可以：

xml复制代码

```xml
<bean class="com.test.bean.Student"/>
```

![image-20221122164149924](https://oss.itbaima.cn/internal/markdown/2022/11/22/SRV3znQJH4A7vDl.png)

可以看到类的旁边出现了Bean的图标，表示我们的Bean已经注册成功了，这样，我们就可以根据类型向容器索要Bean实例对象了：

java复制代码

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
  	//getBean有多种形式，其中第一种就是根据类型获取对应的Bean
  	//容器中只要注册了对应类的Bean或是对应类型子类的Bean，都可以获取到
    Student student = context.getBean(Student.class);
    student.hello();
}
```



## bean歧义问题

不过在有些时候，Bean的获取可能会出现歧义，我们可以来分别注册两个子类的Bean：

```java
public class ArtStudent extends Student{
  	public void art(){
        System.out.println("我爱画画");
    }
}
```

```java
public class SportStudent extends Student{
		public void sport(){
        System.out.println("我爱运动");
    }
}
```

注册两个bean：
```xml
<bean class="com.test.bean.ArtStudent"/>
<bean class="com.test.bean.SportStudent"/>
```

但是此时我们在获取Bean时却是索要的它们的父类：

```java
Student student = context.getBean(Student.class);
student.hello();
```

运行时得到如下报错：

![image-20221122164100561](https://oss.itbaima.cn/internal/markdown/2022/11/22/vliWJ4SZdx5E6yX.png)

这里出现了一个Bean定义不唯一异常，很明显，因为我们需要的类型是Student，但是此时有两个Bean定义都满足这个类型，它们都是Student的子类，此时IoC容器不知道给我们返回哪一个Bean，所以就只能抛出异常了。


**结果**：
因此，如果我们需要一个Bean并且使用类型进行获取，那么必须要指明类型并且不能出现歧义：

```java
ArtStudent student = context.getBean(ArtStudent.class);
student.art();
```


## bean重复问题

那要是两个Bean的类型都是一样的呢？

```java
<bean class="com.test.bean.Student"/>
<bean class="com.test.bean.Student"/>
```

这种情况下，就无法使用Class来进行区分了，除了为Bean指定对应类型之外，我们也可以为Bean指定一个名称用于区分：

```xml
<bean name="art" class="com.test.bean.ArtStudent"/>
<bean name="sport" class="com.test.bean.SportStudent"/>
```

name属性就是为这个Bean设定一个独一无二的名称（id属性也可以，跟name功能相同，但是会检查命名是否规范，否则会显示黄标），不同的Bean名字不能相同，否则报错：

```xml
<bean name="a" class="com.test.bean.Student"/>
<bean name="b" class="com.test.bean.Student"/>
```

这样，这两个Bean我们就可以区分出来了：

```java
Student student = (Student) context.getBean("a");
student.hello();
```

虽然目前这两Bean定义都是一模一样的，也没什么区别，但是这确实是两个不同的Bean，只是类型一样而已，之后我们还可以为这两个Bean分别设置不同的其他属性。


## bean别名- alias属性

我们可以给Bean起名字，也可以起别名，就行我们除了有一个名字之外，可能在家里还有自己的小名：

```xml
<bean name="a" class="com.test.bean.Student"/>
<alias name="a" alias="test"/>
```

这样，我们使用别名也是可以拿到对应的Bean的：

```java
Student student = (Student) context.getBean("test");
student.hello();
```


## bean的单例模式和原型模式

那么现在又有新的问题了，IoC容器创建的Bean是只有一个还是每次索要的时候都会给我们一个新的对象？我们现在在主方法中连续获取两次Bean对象：

```java
Student student1 = context.getBean(Student.class);
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);   //默认为单例模式，对象始终为同一个
```

我们发现，最后得到的结果为true，那么说明每次从IoC容器获取到的对象，始终都是同一个，默认情况下，通过IoC容器进行管理的Bean都是单例模式的，这个对象只会被创建一次。

如果我们希望每次拿到的对象都是一个新的，我们也可以将其作用域进行修改：

![image-20221122175719997](https://oss.itbaima.cn/internal/markdown/2022/11/22/hDGo7m9uBlgVn5A.png)

这里一共有两种作用域，第一种是`singleton`，默认情况下就是这一种，

当然还有`prototype`，表示为原型模式（为了方便叫多例模式也行）这种模式每次得到的对象都是一个新的：

```java
Student student1 = context.getBean(Student.class);  //原型模式下，对象不再始终是同一个了
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);
```

### 懒加载 lazy-init

实际上，当Bean的作用域为单例模式时，那么它会在一开始（容器加载配置时）就被创建，我们之后拿到的都是这个对象。而处于原型模式下，只有在获取时才会被创建，也就是说，单例模式下，Bean会被IoC容器存储，只要容器没有被销毁，那么此对象将一直存在，而原型模式才是相当于在要用的时候直接new了一个对象，并不会被保存。

当然，如果我们希望单例模式下的Bean不用再一开始就加载，而是一样等到需要时再加载（加载后依然会被容器存储，之后一直使用这个对象了，不会再创建新的）我们也可以开启懒加载：

```xml
<bean class="com.test.bean.Student" lazy-init="true"/>
```

开启懒加载后，只有在真正第一次使用时才会创建对象。


## bean加载顺序
因为单例模式下Bean是由IoC容器加载，但是加载顺序我们并不清楚，如果我们需要维护Bean的加载顺序（比如某个Bean必须要在另一个Bean之前创建）那么我们可以使用`depends-on`来设定前置加载Bean，这样被依赖的Bean一定会在之前加载，比如Teacher应该在Student之前加载：

```xml
<bean name="teacher" class="com.test.bean.Teacher"/>
<bean name="student" class="com.test.bean.Student" depends-on="teacher"/>
```

这样就可以保证Bean的加载顺序了。