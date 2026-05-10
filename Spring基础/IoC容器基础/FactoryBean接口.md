

通常情况下，Spring 通过反射机制直接实例化 Bean。但如果一个 Bean 的创建过程非常复杂（例如：需要大量的配置、依赖底层 API、或者需要创建代理对象），使用传统的 XML 或 `@Bean` 注解会显得代码非常臃肿。这时，`FactoryBean` 就派上用场了。

---

## 1. 核心接口定义

`FactoryBean` 接口包含三个核心方法：

- **`T getObject()`**: 返回由 FactoryBean 创建的 Bean 实例。这是 Spring 容器实际管理的那个对象。
    
- **`Class<?> getObjectType()`**: 返回创建的 Bean 的类型。
    
- **`boolean isSingleton()`**: 返回创建的 Bean 是否为单例（默认为 true）。
    

---

## 2. 基础用法示例

假设我们有一个复杂的类 `SmartDevice`，它的初始化需要连接网络、验证秘钥等操作。

### 步骤一：实现接口

```java
@Component
public class SmartDeviceFactoryBean implements FactoryBean<SmartDevice> {

    @Override
    public SmartDevice getObject() throws Exception {
        // 这里可以包含极其复杂的初始化逻辑
        SmartDevice device = new SmartDevice();
        device.connect("192.168.1.1");
        device.activate("SECRET_KEY");
        return device;
    }

    @Override
    public Class<?> getObjectType() {
        return SmartDevice.class;
    }
}
```

### 步骤二：从容器获取

```java
// 正常获取：得到的是 getObject() 返回的对象（SmartDevice）
SmartDevice device = context.getBean(SmartDevice.class);

// 特殊获取：如果想获取 FactoryBean 本身，需在 ID 前加 '&'
Object factory = context.getBean("&smartDeviceFactoryBean");
```


## FactoryBean的透明机制

在 Spring 中，`FactoryBean` 有一种“**欺骗性**”或“**透明性**”：虽然你注册的是 `SmartDeviceFactoryBean` 类，但 Spring 会认为这个 Bean 的**真实身份**是它所生产的那个对象（即 `getObject()` 返回的对象）。

以下是详细的原因和内部机制：

### 1. Spring 的自动识别机制

当你把一个类实现 `FactoryBean` 接口并交给 Spring 管理时，Spring 内部会进行特殊的逻辑判断：

- **注册阶段**：Spring 确实在容器里登记了一个名为 `smartDeviceFactoryBean` 的 Bean。
    
- **暴露阶段**：Spring 会调用该工厂的 `getObjectType()` 方法。当它发现这是一个 `FactoryBean` 时，它会向外“宣称”：_“我这里有一个类型为 `SmartDevice` 的对象可用”_。
    

因此，当你执行 `context.getBean(SmartDevice.class)` 时：

1. Spring 会在容器中寻找类型为 `SmartDevice` 的 Bean。
    
2. 它发现 `SmartDeviceFactoryBean` 声明自己是生产 `SmartDevice` 的工厂（通过 `getObjectType()`）。
    
3. Spring 自动调用该工厂的 `getObject()` 方法，并将返回的 `SmartDevice` 实例交给你。
    

---

### 2. 这里的“潜规则”：Bean 的两个身份

一个 `FactoryBean` 在容器中其实对应着**两个对象**：

1. **产品对象 (Product)**：即 `getObject()` 返回的结果。这是**默认暴露**的对象。
    
2. **工厂对象 (Factory)**：即 `SmartDeviceFactoryBean` 实例本身。这是**隐藏**的对象。
    

**代码对比：**
java
```
// 1. 获取产品 (最常用的方式)
// Spring 发现 SmartDevice 是产品类型，自动去工厂里取货
SmartDevice device = context.getBean(SmartDevice.class); 

// 2. 获取工厂本身 (特殊需求)
// 如果你非要拿到那个工厂类，必须在 ID 前加个 & 符号
Object factory = context.getBean("&smartDeviceFactoryBean");
System.out.println(factory instanceof SmartDeviceFactoryBean); // 结果为 true
```