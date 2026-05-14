

通过了解数据源，我们已经清楚，Mybatis实际上是在使用自己编写的数据源（数据源实现其实有很多，之后我们再聊其他的）默认使用的是池化数据源，它预先存储了很多的连接对象。

# Spring管理MyBatis 

那么我们来看一下，如何将Mybatis与Spring更好的结合呢，比如我们现在希望将SqlSessionFactory交给IoC容器进行管理，而不是我们自己创建工具类来管理（我们之前一直都在使用工具类管理和创建会话）

## 导入依赖
```xml
<!-- 这两个依赖不用我说了吧 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
  	<!-- 注意，对于Spring 6.0来说，版本需要在3.5以上 -->
    <version>3.5.13</version>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.31</version>
</dependency>
<!-- Mybatis针对于Spring专门编写的支持框架 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.2</version>
</dependency>
<!-- Spring的JDBC支持框架 -->
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-jdbc</artifactId>
     <version>6.0.10</version>
</dependency>
```


## 通过SqlSessionTemplate使用数据库


在mybatis-spring依赖中，为我们提供了SqlSessionTemplate类，它其实就是官方封装的一个工具类，我们可以将其注册为Bean，这样我们随时都可以向IoC容器索要对象，而不用自己再去编写一个工具类了，我们可以直接在配置类中创建。对于这种别人编写的类型，如果要注册为Bean，那么只能在配置类中完成：

```java
@Configuration
@ComponentScan("org.example.entity")
public class MainConfiguration {
  	//注册SqlSessionTemplate的Bean
    @Bean
    public SqlSessionTemplate sqlSessionTemplate() throws IOException {
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
        return new SqlSessionTemplate(factory);
    }
}
```


这里随便编写一个测试的Mapper类：

```java
@Data
public class Student {
    private int sid;
    private String name;
    private String sex;
}
```


```java
public interface TestMapper {
    @Select("select * from student where sid = 1")
    Student getStudent();
}
```


最后是配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/study"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
  	<mappers>
        <mapper class="org.example.mapper.TestMapper"/>
    </mappers>
</configuration>
```

我们来测试一下吧：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    SqlSessionTemplate template = context.getBean(SqlSessionTemplate.class);
    TestMapper testMapper = template.getMapper(TestMapper.class);
    System.out.println(testMapper.getStudent());
}
```



这样，我们就成功将Mybatis与Spring完成了初步整合，直接从容器中就能获取到SqlSessionTemplate，结合自动注入，我们的代码量能够进一步的减少。




## @MapperScan 让Spring直接管理所有的Mapper

虽然这样已经很方便了，但是还不够方便，我们依然需要手动去获取Mapper对象，那么能否直接得到对应的Mapper对象呢，我们希望让Spring直接帮助我们管理所有的Mapper，当需要时，可以直接从容器中获取，我们可以直接在配置类上方添加注解：

```java
@Configuration
@ComponentScan("org.example.entity")
@MapperScan("org.example.mapper")
public class MainConfiguration {
```

这样，Mybatis就会自动扫描对应包下所有的接口，并直接被注册为对应的Mapper作为Bean管理，那么我们现在就可以直接通过容器获取了：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    TestMapper mapper = context.getBean(TestMapper.class);
    System.out.println(mapper.getStudent());
}
```

在我们后续的SpringBoot学习阶段，会有更加方便的方式来注册Mapper，我们只需要一个`@Mapper`注解即可完成，非常简单。

请一定注意，必须存在`SqlSessionTemplate`或是`SqlSessionFactoryBean`的Bean，否则会无法初始化（毕竟要数据库的链接信息）


# 不用xml配置文件该怎么使用数据库
我们接着来看，如果我们希望直接去除Mybatis的配置文件，完全实现全注解配置，那么改怎么去实现呢？我们可以使用`SqlSessionFactoryBean`类：

```java
@Configuration
@ComponentScan("org.example.entity")
@MapperScan("org.example.mapper")
public class MainConfiguration {
    @Bean   //单独创建一个Bean，方便之后更换
    public DataSource dataSource(){
        return new PooledDataSource("com.mysql.cj.jdbc.Driver",
                "jdbc:mysql://localhost:3306/study", "root", "123456");
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){  //直接参数得到Bean对象
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean;
    }
}
```

首先我们需要创建一个数据源的实现类，因为这是数据库最基本的信息，然后再给到`SqlSessionFactoryBean`实例，这样，我们相当于直接在一开始通过IoC容器配置了`SqlSessionFactory`，这里只需要传入一个`DataSource`的实现即可，我们采用池化数据源。

删除配置文件，重新再来运行，同样可以正常使用Mapper。从这里开始，通过IoC容器，Mybatis已经不再需要使用配置文件了，在我们之后的学习中，基于Spring的开发将不会再出现Mybatis的配置文件。