前面我们已经完成了大部分的配置文件学习，但是我们发现，使用配置文件进行配置，貌似有点太累了吧？可以想象一下，如果我们的项目非常庞大，整个配置文件将会充满Bean配置，并且会继续庞大下去，能否有一种更加高效的方法能够省去配置呢？还记得我们在JavaWeb阶段用到的非常方便东西吗？没错，就是注解。

# 使用注解创建、配置、使用Bean


## 配置文件变成配置类

既然现在要使用注解来进行开发，那么我们就删掉之前的xml配置文件吧，我们来看看使用注解能有多方便。

```java
ApplicationContext context = new AnnotationConfigApplicationContext();
```

现在我们使用AnnotationConfigApplicationContext作为上下文实现，它是注解配置的。

既然现在采用注解，我们就需要使用类来编写配置文件，在之前，我们如果要编写一个配置的话，需要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

现在我们只需要创建一个配置类就可以了：

```java
@Configuration
public class MainConfiguration {
}
```

这两者是等价的，同样的，在一开始会提示我们没有配置上下文：

![image-20221123175555433](https://oss.itbaima.cn/internal/markdown/2022/11/23/YNl5nVkfJriogdD.png)

这里按照要求配置一下就可以，同上，这个只是会影响IDEA的代码提示，不会影响程序运行。

我们可以为AnnotationConfigApplicationContext指定一个默认的配置类：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
//这个构造方法可以接收多个配置类（更准确的说是多个组件）
```


### 导入其他配置类
通过@Import还可以引入其他配置类：

```java
@Import(LBWConfiguration.class)  //在讲解到Spring原理时，我们还会遇到它，目前只做了解即可。
@Configuration
public class MainConfiguration {
```


## 在配置类中创建并配置Bean

那么现在我们该如何配置Bean呢？

```java
@Configuration
public class MainConfiguration {

    @Bean("student")
    public Student student(){
        return new Student();
    }
}
```

这样写相对于配置文件中的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean name = "student" class="com.test.bean.Student"/>
</beans>
```

只不过现在变成了由Java代码为我们提供Bean配置，这样会更加的灵活，也更加便于控制Bean对象的创建。

```java
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
Student student = context.getBean(Student.class);
System.out.println(student);
```

使用方法是相同的，这跟使用XML配置是一样的。


### Bean特性的配置


#### name、initmethod、destroymethod等属性

那么肯定就有小伙伴好奇了，我们之前使用的那么多特性在哪里配置呢？首先，初始化方法和摧毁方法、自动装配可以直接在@Bean注解中进行配置：

```java
@Bean(name = "", initMethod = "", destroyMethod = "", autowireCandidate = false)
public Student student(){
    return new Student();
}
```


#### lazyinit、scope、dependon等属性

其次，我们可以使用一些其他的注解来配置其他属性，比如：

```java
@Bean
@Lazy(true)     //对应lazy-init属性
@Scope("prototype")    //对应scope属性
@DependsOn("teacher")    //对应depends-on属性
public Student student(){
    return new Student();
}
```



### 对bean进行依赖注入

#### 注入一个变量


#### 注入其他bean

##### 将其他bean作为参数传递到@Bean标注的方法中
对于那些我们需要通过构造方法或是Setter完成依赖注入的Bean，比如：

```xml
<bean name="teacher" class="com.test.bean.ProgramTeacher"/>
<bean name="student" class="com.test.bean.Student">
    <property name="teacher" ref="teacher"/>
</bean>
```

像这种需要引入其他Bean进行的注入，我们可以直接将其作为形式参数放到方法中：

```java
@Configuration
public class MainConfiguration {
    @Bean
    public Teacher teacher(){
        return new Teacher();
    }

    @Bean
    public Student student(Teacher teacher){
        return new Student(teacher);
    }
}
```

此时我们可以看到，旁边已经出现图标了：

![image-20221123213527325](https://oss.itbaima.cn/internal/markdown/2022/11/23/wy5JtiVp8zK1bmG.png)

运行程序之后，我们发现，这样确实可以直接得到对应的Bean并使用。


##### 在对应的类中打注解@Autowired

只不过，除了这种基于构造器或是Setter的依赖注入之外，我们也可以直接到Bean对应的类中使用自动装配：

```java
public class Student {
    @Autowired   //使用此注解来进行自动装配，由IoC容器自动为其赋值
    private Teacher teacher;
}
```

现在，我们甚至连构造方法和Setter都不需要去编写了，就能直接完成自动装配，是不是感觉比那堆配置方便多了？

当然，@Autowired并不是只能用于字段，对于构造方法或是Setter，它同样可以：

```java
public class Student {
    private Teacher teacher;

    @Autowired
    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }
}
```

@Autowired默认采用byType的方式进行自动装配，也就是说会使用类型进行配，
##### 指定使用某个bean@Qualifier

那么要是出现了多个相同类型的Bean，如果我们想要指定使用其中的某一个该怎么办呢？

```java
@Bean("a")
public Teacher teacherA(){
    return new Teacher();
}

@Bean("b")
public Teacher teacherB(){
    return new Teacher();
}
```

此时，我们可以配合@Qualifier进行名称匹配：

```java
public class Student {
    @Autowired
    @Qualifier("a")   //匹配名称为a的Teacher类型的Bean
    private Teacher teacher;
}
```


##### 旧版本的一些注解

这里需要提一下，在我们旧版本的SSM教程中讲解了@Resource这个注解，但是现在它没有了。

随着Java版本的更新迭代，某些javax包下的包，会被逐渐弃用并移除。在JDK11版本以后，javax.annotation这个包被移除并且更名为jakarta.annotation（我们在JavaWeb篇已经介绍过为什么要改名字了）其中有一个非常重要的注解，叫做@Resource，它的作用与@Autowired是相同的，也可以实现自动装配，但是在IDEA中并不推荐使用@Autowired注解对成员字段进行自动装配，而是推荐使用@Resource，如果需要使用这个注解，还需要额外导入包：

```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

使用方法一样，直接替换掉就可以了：

```java
public class Student {
    @Resource
    private Teacher teacher;
}
```

只不过，他们两有些机制上的不同：

- @Resource默认**ByName**如果找不到则**ByType**，可以添加到set方法、字段上。
- @Autowired默认是**byType**，只会根据类型寻找，可以添加在构造方法、set方法、字段、方法参数上。

因为@Resource的匹配机制更加合理高效，因此官方并不推荐使用@Autowired字段注入，当然，实际上Spring官方更推荐我们使用基于构造方法或是Setter的@Autowired注入，比如Setter 注入的一个好处是，Setter 方法使该类的对象能够在以后重新配置或重新注入。其实，最后使用哪个注解，还是看你自己，要是有强迫症不能忍受黄标但是又实在想用字段注入，那就用@Resource注解。


除了这个注解之外，还有@PostConstruct和@PreDestroy，它们效果和init-method和destroy-method是一样的：

```java
@PostConstruct
public void init(){
    System.out.println("我是初始化方法");
}

@PreDestroy
public void destroy(){
    System.out.println("我是销毁方法");
}
```

我们只需要将其添加到对应的方法上即可：

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
Student student = context.getBean(Student.class);
context.close();
```

![image-20221123225232979](https://oss.itbaima.cn/internal/markdown/2022/11/23/wXRuAjVF2ykCOJ4.png)

可以看到效果是完全一样的，这些注解都是jakarta.annotation提供的，有关Spring和JakartaEE的渊源，还请各位小伙伴自行了解。



## 在要注册的类上打注解来注册bean

前面我们介绍了使用@Bean来注册Bean，但是实际上我们发现，如果只是简单将一个类作为Bean的话，这样写还是不太方便，因为都是固定模式，就是单纯的new一个对象出来，能不能像之前一样，让容器自己反射获取构造方法去生成这个对象呢？

肯定是可以的，我们可以在需要注册为Bean的类上添加`@Component`注解来将一个类进行注册 **（现在最常用的方式）** ，不过要实现这样的方式，我们需要添加一个自动扫描来告诉Spring，它需要在哪些包中查找我们提供的`@Component`声明的Bean。

### @Component

```java
@Component("lbwnb")   //同样可以自己起名字
public class Student {

}
```

要注册这个类的Bean，只需要添加@Component即可，

### @ComponentScan

然后配置一下包扫描：

```java
@Configuration
@ComponentScan("com.test.bean")   //包扫描，这样Spring就会去扫描对应包下所有的类
public class MainConfiguration {

}
```


Spring在扫描对应包下所有的类时，会自动将那些添加了@Component的类注册为Bean，是不是感觉很方便？只不过这种方式只适用于我们自己编写类的情况，如果是第三方包提供的类，只能使用前者完成注册，并且这种方式并不是那么的灵活。


### 注解注册bean的默认name
不过，无论是通过@Bean还是@Component形式注册的Bean，Spring都会为其添加一个默认的name属性，比如：

```java
@Component
public class Student {
}
```

它的默认名称生产规则依然是类名并按照首字母小写的驼峰命名法来的，所以说对应的就是student：

```java
Student student = (Student) context.getBean("student");   //这样同样可以获取到
System.out.println(student);
```

同样的，如果是通过@Bean注册的，默认名称是对应的方法名称：

```java
@Bean
public Student artStudent(){
    return new Student();
}
```


```java
Student student = (Student) context.getBean("artStudent");
System.out.println(student);
```



### 对有参构造类的自动注入

相比传统的XML配置方式，注解形式的配置确实能够减少我们很多工作量。并且，对于这种使用`@Component`注册的Bean，如果其构造方法不是默认无参构造，那么默认会对其每一个参数都进行自动注入：

```java
@Component
public class Student {
    Teacher teacher;
    public Student(Teacher teacher){   //如果有Teacher类型的Bean，那么这里的参数会被自动注入
        this.teacher = teacher;
    }
}
```


### 工厂模式的使用

最后，对于我们之前使用的工厂模式，Spring也提供了接口，我们可以直接实现接口表示这个Bean是一个工厂Bean：

```java
@Component
public class StudentFactory implements FactoryBean<Student> {
    @Override
    public Student getObject() {   //生产的Bean对象
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {   //生产的Bean类型
        return Student.class;
    }

    @Override
    public boolean isSingleton() {   //生产的Bean是否采用单例模式
        return false;
    }
}
```

实际上跟我们之前在配置文件中编写是一样的，这里就不多说了。
FactoryBean的概念与使用参考[[FactoryBean接口]]


注解与xml配置各有用处
请注意，使用注解虽然可以省事很多，代码也能变得更简洁，但是这并不代表XML配置文件就是没有意义的，它们有着各自的优点，在不同的场景下合理使用，能够起到事半功倍的效果，官方原文：

> Are annotations better than XML for configuring Spring?
> 
> The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML. The short answer is “it depends.” The long answer is that each approach has its pros and cons, and, usually, it is up to the developer to decide which strategy suits them better. Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control.
> 
> No matter the choice, Spring can accommodate both styles and even mix them together. It is worth pointing out that through its [JavaConfig](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) option, Spring lets annotations be used in a non-invasive way, without touching the target components source code and that, in terms of tooling, all configuration styles are supported by the [Spring Tools for Eclipse](https://spring.io/tools).


# 思考

至此，关于Spring的IoC基础部分，我们就全部介绍完了。在最后，留给各位小伙伴一个问题，现在有两个类：

```java
@Component
public class Student {
    @Resource
    private Teacher teacher;
}
```

```java
@Component
public class Teacher {
    @Resource
    private Student student;
}
```

这两个类互相需要注入对方的实例对象，这个时候Spring会怎么进行处理呢？如果Bean变成原型模式，Spring又会怎么处理呢？

这个问题我们会在实现原理探究部分进行详细介绍。