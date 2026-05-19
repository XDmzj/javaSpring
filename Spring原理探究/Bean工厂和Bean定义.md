
我们先从宏观的架构图开始，一步步拆解 `BeanFactory` 的底层源码实现以及 `ApplicationContext` 的“管家”职能。

---

### 一、 BeanFactory 的底层是怎么实现的？

`BeanFactory` 并不是一个孤立的类，而是一套**高度解耦的工业流水线**。在底层，它主要依赖三个核心组件协同工作：

1. **配置注册表（`BeanDefinitionRegistry`）：** 负责存储 Bean 的“说明书”。
    
2. **说明书对象（`BeanDefinition`）：** 里面记录了 Bean 的全类名、是否单例、懒加载状态、构造函数参数、依赖的属性等信息。
    
3. **单例池（`SingletonBeanRegistry`）：** 一个底层的并发哈希表（`ConcurrentHashMap`），用来缓存所有已经创建好的单例 Bean。
    

#### 核心源码实现：以 `DefaultListableBeanFactory` 为例

在 Spring 底层，最核心、最常用的 BeanFactory 实现类是 `DefaultListableBeanFactory`。如果你去看它的源码，你会发现它的核心结构非常朴素：

Java

```
public class DefaultListableBeanFactory extends ... implements ... {

    // 1. 底层的单例池：存放所有创建好的单例对象（我们常说的 Spring 容器实体）
    // 继承自 DefaultSingletonBeanRegistry
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

    // 2. 说明书池：存放所有解析出来的 Bean 描述信息
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

    // 3. 记录所有 Bean 名字的列表，用于保证顺序
    private final List<String> beanDefinitionNames = new ArrayList<>(256);
}
```

当你调用 `getBean("bookService")` 时，它的底层逻辑是：

1. 先去 `singletonObjects`（单例池）里找，如果找到了，直接返回。
    
2. 如果找不到，去 `beanDefinitionMap` 里找到 `bookService` 的“说明书”（`BeanDefinition`）。
    
3. 利用 Java 反射机制（`Constructor.newInstance()`）把这个类实例化。
    
4. 根据说明书里的依赖关系，把其他的 Bean 注入进去（属性填充）。
    
5. 执行初始化方法（如 `@PostConstruct`），然后把完整的 Bean 放进 `singletonObjects` 单例池，方便下次直接使用。
    

---

### 二、 BeanFactory 是怎么接收我给的配置的？

无论是 XML、注解（`@Component`），还是 Spring Boot 的 Java 配置（`@Configuration`），`BeanFactory` 自身是**不直接去读文件**的。它非常高冷，只认识 `BeanDefinition`（说明书）。

这中间靠的是一个核心角色：**`BeanDefinitionReader`（说明书读取器）**。

整个接收和解析配置的流程如下：

1. **资源定位 (Resource):** Spring 根据你提供的路径（如 XML 路径或包扫描路径），把配置文件抽象成一个 `Resource` 对象。
    
2. **解析读取 (Reader):**
    
    - 如果你用 **XML**，Spring 会调度 `XmlBeanDefinitionReader` 去解析 `<bean>` 标签。
        
    - 如果你用 **注解/Spring Boot**，Spring 会调度 `ClassPathBeanDefinitionScanner` 去扫描指定包下的类，寻找含有 `@Component`、`@Service` 的类。
        
3. **翻译成说明书 (BeanDefinition):** Reader 会把这些配置信息“翻译”成一个 `BeanDefinition` 对象。例如：把 XML 里的 `<bean id="book" class="com.xdmzj.pojo.Book">` 翻译成一个存储了类名字符串和 ID 的 Java 对象。
    
4. **注册入库:** 最后，Reader 调用 `registry.registerBeanDefinition()` 方法，把这张说明书塞进 `DefaultListableBeanFactory` 的 `beanDefinitionMap` 中。
    

**至此，配置接收完成。此时还没有任何真实的 Java 对象被 new 出来，容器里全是“说明书”。**

---

### 三、 ApplicationContext 除了维护 BeanFactory 还做了什么？

正如我们前面打的比方，`BeanFactory` 是发动机，而 `ApplicationContext` 是整辆豪华轿车。

如果你去看 `ApplicationContext` 的源码，你会发现它继承了 `BeanFactory` 的子接口，同时还**继承了另外好几个接口**。这些接口赋予了它除了管理 Bean 之外的四大核心超能力：

#### 1. 国际化支持 (`MessageSource`)

它支持多语言切换。在开发大型电商系统或跨国后台时，它可以根据请求来自中国还是美国，自动读取 `messages_zh_CN.properties` 或 `messages_en_US.properties`，从而返回中文或英文的提示信息。

#### 2. 事件发布与监听机制 (`ApplicationEventPublisher`)

这是解耦神技（观察者模式的实现）。

- _场景：_ 用户在你的项目里注册成功了。传统的写法是：在注册代码里调用“发短信的类”和“发邮件的类”。
    
- _ApplicationContext 的做法：_ 注册成功后，直接向容器广播一个“用户已注册”的事件（Event）。容器里监听了这个事件的短信服务、邮件服务会自动收到通知并干活。注册逻辑不需要关心谁去发短信，实现了彻底的解耦。
    

#### 3. 统一资源加载 (`ResourcePatternResolver`)

它是一个强大的“文件寻宝手”。无论你的配置文件是在类路径下（`classpath:`）、本地磁盘里（`file:`）、还是网络 URL 上（`http:`），它都能用一套统一的 API（`getResource()`）帮你把它加载进来。

#### 4. 强大的环境参数管理 (`EnvironmentCapable`)

它内部有一个 `Environment` 对象，负责聚合管理整个系统的配置。

- 它可以同时读取你的 `application.yml`、操作系统的环境变量（比如系统用户名）、JVM 启动参数（`-D` 参数）。
    
- 利用它，你可以轻松实现**多环境切换**（开发环境用一套数据库配置 `dev`，生产环境用另一套 `prod`）。
    

---

### 总结

- **`BeanFactory` 的底层：** 两个 `ConcurrentHashMap`，一个存说明书，一个存真实的单例对象。
    
- **配置的接收：** 靠各种 `Reader` 把你的代码/XML 扫描并翻译成说明书存进工厂。
    
- **`ApplicationContext` 的真面目：** 它内部**组合**了一个 `DefaultListableBeanFactory` 来管对象，同时自己**继承**了国际化、事件、资源、环境等接口，摇身一变成为了企业级的“全能大管家”。