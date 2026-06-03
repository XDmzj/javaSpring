

在没有 JPA 之前，程序员需要写大量的 SQL 语句并将结果集手动封装成 Java 对象；

而 JPA 让我们能够像操作内存中的 Java 对象一样去自动增删改查数据库。

下面为你梳理 JPA 的宏观地位、核心工作原理、高频实战对比以及核心注解。
# 宏观架构地位：它是一套接口，不是一个框架

初学者最容易混淆 JPA 与 Hibernate 的关系。在宏观架构中，JPA 属于标准制定者，而 Hibernate 属于标准执行者。

- JPA 是规范（Specification）： 它只定义了一堆接口（Interface）和注解（Annotation），属于 Java 官方标准（JSR）。它本身是没有办法直接运行并干活的。
    
- Hibernate 是实现（Provider）： 它是具体的持久化框架。Hibernate 实现了 JPA 定义的所有接口。Spring Boot 底层默认就是用 Hibernate 来作为 JPA 的底层引擎。
    
- Spring Data JPA 是封装（Wrapper）： 它是 Spring 家族对 JPA 的进一步高级封装。它让我们连底层的 EntityManager 都不用写，只需要定义一个接口继承 `JpaRepository`，连 SQL 都不用写就能自动实现增删改查。

在企业级实战中，Java 后端技术选型通常会在 Spring Data JPA 和 MyBatis（或 MyBatis-Plus）之间做抉择。两者的宏观差异如下：

| 比较维度   | Spring Data JPA (现代 ORM 流派)                | MyBatis / MyBatis-Plus (SQL 流派)     |
| ---------- | ---------------------------------------------- | --------------------------------------- |
| 核心哲学   | 面向对象。尽量让你忘记 SQL，全靠对象驱动。                    | 面向 SQL。把 SQL 的控制权完全交还给程序员。          |
| 自动化程度  | 极高。单表增删改查连一条 SQL 都不用写。                         | 中等。单表靠 MyBatis-Plus，复杂单表或多表需手写 SQL。     |
| SQL 调优 | 较难。底层 SQL 由框架自动生成，复杂多表联查时难以控制。                 | 极易。SQL 写在 XML 里，可以针对高并发场景做极致的 SQL 性能调优。 |
| 数据库移植性 | 极强。通过更换“方言（Dialect）”，代码不改就能从 MySQL 切换到 Oracle。 | 较差。如果写了 MySQL 特有的函数，切换数据库需要重写 XML。      |
| 高频适用场景 | 政企项目、中小型电商、单表高频增删改查、DDD（领域驱动设计）微服务。        | 国内互联网大厂（如秒杀、高并发外卖）、复杂报表系统、多表频繁关联查询。 |


# 使用JPA上手

同样的，我们只需要导入stater依赖即可：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

接着我们可以直接创建一个类，比如用户类，我们只需要把一个账号对应的属性全部定义好即可：

```java
@Data
public class Account {
    int id;
    String username;
    String password;
}
```

### 通过注解在属性上添加数据库映射关系

接着，我们可以通过注解形式，在属性上添加数据库映射关系，这样就能够让JPA知道我们的实体类对应的数据库表长啥样，这里用到了很多注解：

```java
@Data
@Entity   //表示这个类是一个实体类
@Table(name = "account")    //对应的数据库中表名称
public class Account {

    @GeneratedValue(strategy = GenerationType.IDENTITY)   //生成策略，这里配置为自增
    @Column(name = "id")    //对应表中id这一列
    @Id     //此属性为主键
    int id;

    @Column(name = "username")   //对应表中username这一列
    String username;

    @Column(name = "password")   //对应表中password这一列
    String password;
}
```

### 配置修改


接着我们来修改一下配置文件，把日志打印给打开：

```yaml
spring:
  jpa:
    #开启SQL语句执行日志信息
    show-sql: true
    hibernate:
      #配置为检查数据库表结构，没有时会自动创建
      ddl-auto: update
```

`ddl-auto`属性用于设置自动表定义，可以实现自动在数据库中为我们创建一个表，表的结构会根据我们定义的实体类决定，它有以下几种：

- `none`: 不执行任何操作，数据库表结构需要手动创建。
- `create`: 框架在每次运行时都会删除所有表，并重新创建。
- `create-drop`: 框架在每次运行时都会删除所有表，然后再创建，但在程序结束时会再次删除所有表。
- `update`: 框架会检查数据库表结构，如果与实体类定义不匹配，则会做相应的修改，以保持它们的一致性。
- `validate`: 框架会检查数据库表结构与实体类定义是否匹配，如果不匹配，则会抛出异常。

这个配置项的作用是为了避免手动管理数据库表结构，使开发者可以更方便地进行开发和测试，但在生产环境中，更推荐使用数据库迁移工具来管理表结构的变更。

我们可以在日志中发现，在启动时执行了如下SQL语句：

![image-20230720235136506](https://oss.itbaima.cn/internal/markdown/2023/07/20/kABZVhJ8vjKSqzT.png)

我们的数据库中对应的表已经自动创建好了。


### Repository实现类

我们接着来看如何访问我们的表，我们需要创建一个Repository实现类：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
}
```

注意JpaRepository有两个泛型，前者是具体操作的对象实体，也就是对应的表，后者是ID的类型，接口中已经定义了比较常用的数据库操作。编写接口继承即可，我们可以直接注入此接口获得实现类：

```java
@Resource
AccountRepository repository;

@Test
void contextLoads() {
    Account account = new Account();
    account.setUsername("小红");
    account.setPassword("1234567");
    System.out.println(repository.save(account).getId());   //使用save来快速插入数据，并且会返回插入的对象，如果存在自增ID，对象的自增id属性会自动被赋值，这就很方便了
}
```

执行结果如下：

![image-20230720235640148](https://oss.itbaima.cn/internal/markdown/2023/07/20/ksI3J5eidzTrvyL.png)

同时，查询操作也很方便：

```java
@Test
void contextLoads() {
  	//默认通过通过ID查找的方法，并且返回的结果是Optional包装的对象，非常人性化
    repository.findById(1).ifPresent(System.out::println);
}
```

得到结果为：

![image-20230720235949290](https://oss.itbaima.cn/internal/markdown/2023/07/20/TRHOWbop267Al4Q.png)

包括常见的一些计数、删除操作等都包含在里面，仅仅配置该接口就能完美实现增删改查：

![image-20230721000050875](https://oss.itbaima.cn/internal/markdown/2023/07/21/uIBciLqFsH5tdDR.png)

我们发现，使用了JPA之后，整个项目的代码中没有出现任何的SQL语句，可以说是非常方便了，JPA依靠我们提供的注解信息自动完成了所有信息的映射和关联。

相比Mybatis，JPA几乎就是一个全自动的ORM框架，而Mybatis则顶多算是半自动ORM框架。




# 方法名称拼接自定义简单SQL


虽然接口预置的方法使用起来非常方便，但是如果我们需要进行条件查询等操作或是一些判断，就需要自定义一些方法来实现，同样的，我们不需要编写SQL语句，而是通过方法名称的拼接来实现条件判断，这里列出了所有支持的条件判断名称：

| 属性                | 拼接方法名称示例                                                    |
| ----------------- | ----------------------------------------------------------- |
| Distinct          | findDistinctByLastnameAndFirstname                          |
| And               | findByLastnameAndFirstname                                  |
| Or                | findByLastnameOrFirstname                                   |
| Is，Equals         | findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals |
| Between           | findByStartDateBetween                                      |
| LessThan          | findByAgeLessThan                                           |
| LessThanEqual     | findByAgeLessThanEqual                                      |
| GreaterThan       | findByAgeGreaterThan                                        |
| GreaterThanEqual  | findByAgeGreaterThanEqual                                   |
| After             | findByStartDateAfter                                        |
| Before            | findByStartDateBefore                                       |
| IsNull，Null       | findByAge(Is)Null                                           |
| IsNotNull，NotNull | findByAge(Is)NotNull                                        |
| Like              | findByFirstnameLike                                         |
| NotLike           | findByFirstnameNotLike                                      |
| StartingWith      | findByFirstnameStartingWith                                 |
| EndingWith        | findByFirstnameEndingWith                                   |
| Containing        | findByFirstnameContaining                                   |
| OrderBy           | findByAgeOrderByLastnameDesc                                |
| Not               | findByLastnameNot                                           |
| In                | findByAgeIn(Collection ages)                                |
| NotIn             | findByAgeNotIn(Collection ages)                             |
| True              | findByActiveTrue                                            |
| False             | findByActiveFalse                                           |
| IgnoreCase        | findByFirstnameIgnoreCase                                   |
|                   |                                                             |

比如我们想要实现根据用户名模糊匹配查找用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    //按照表中的规则进行名称拼接，不用刻意去记，IDEA会有提示
    List<Account> findAllByUsernameLike(String str);
}
```

我们来测试一下：

```java
@Test
void contextLoads() {
    repository.findAllByUsernameLike("%明%").forEach(System.out::println);
}
```

![image-20230721001035279](https://oss.itbaima.cn/internal/markdown/2023/07/21/mioZaUk7Yj3QDxb.png)

又比如我们想同时根据用户名和ID一起查询：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);

    Account findByIdAndUsername(int id, String username);
    //也可以使用Optional类进行包装，Optional<Account> findByIdAndUsername(int id, String username);
}
```

```java
@Test
void contextLoads() {
    System.out.println(repository.findByIdAndUsername(1, "小明"));
}
```

比如我们想判断数据库中是否存在某个ID的用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);
    Account findByIdAndUsername(int id, String username);
    //使用exists判断是否存在
    boolean existsAccountById(int id);
}
```

注意自定义条件操作的方法名称一定要遵循规则，不然会出现异常：

```sh
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract  ...
```

有了这些操作，我们在编写一些简单SQL的时候就很方便了，用久了甚至直接忘记SQL怎么写。


但是JPA中通过方法名来自动生成SQL语句，这种方式无法处理复杂查询的情况，因为复杂查询的语句太长了，用方法名来表示会长的没边



# 关联查询
[[JPA关联查询]]



# JPQL自定义SQL语句

虽然SpringDataJPA能够简化大部分数据获取场景，但是难免会有一些特殊的场景，需要使用复杂查询才能够去完成，这时你又会发现，如果要实现，只能用回Mybatis了，因为我们需要自己手动编写SQL语句，过度依赖SpringDataJPA会使得SQL语句不可控。

使用JPA，我们也可以像Mybatis那样，直接编写SQL语句，不过它是JPQL语言，与原生SQL语句很类似，但是它是面向对象的，当然我们也可以编写原生SQL语句。

比如我们要更新用户表中指定ID用户的密码：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    @Transactional    //DML操作需要事务环境，可以不在这里声明，但是调用时一定要处于事务环境下
    @Modifying     //表示这是一个DML操作
    @Query("update Account set password = ?2 where id = ?1") 
    这里操作的是一个实体类对应的表，用的是类名不是表名，参数使用?代表，后面接第n个参数
    int updatePasswordById(int id, String newPassword);
}
```

```java
@Test
void updateAccount(){
    repository.updatePasswordById(1, "654321");
}
```

现在我想使用原生SQL来实现根据用户名称修改密码：

```java
@Transactional
@Modifying
@Query(value = "update users set password = :pwd where username = :name", nativeQuery = true) //使用原生SQL，和Mybatis一样，这里使用 :名称 表示参数，当然也可以继续用上面那种方式。
int updatePasswordByUsername(@Param("name") String username,   //我们可以使用@Param指定名称
                             @Param("pwd") String newPassword);
```

```java
@Test
void updateAccount(){
    repository.updatePasswordByUsername("Admin", "654321");
}
```

通过编写原生SQL，在一定程度上弥补了SQL不可控的问题。

虽然JPA能够为我们带来非常便捷的开发体验，但是正是因为太便捷了，保姆级的体验有时也会适得其反，尤其是一些国内用到复杂查询业务的项目，可能开发到后期特别庞大时，就只能从底层SQL语句开始进行优化，而由于JPA尽可能地在屏蔽我们对SQL语句的编写，所以后期优化是个大问题，并且Hibernate相对于Mybatis来说，更加重量级。不过，在微服务的时代，单体项目一般不会太大，JPA的劣势并没有太明显地体现出来。