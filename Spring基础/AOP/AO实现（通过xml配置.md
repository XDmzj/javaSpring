
# xml配置使用AOP

### `<aop:config>`：开启 AOP 

这是 AOP 配置的**根标签**。

- **作用**：所有的 AOP 逻辑（切点定义、通知关联）都必须包裹在这个标签内部。它相当于告诉 Spring：“注意，接下来的配置是要生成动态代理对象的。”
    

---

### `<aop:pointcut>` 与 `expression`：

这是最核心的部分。**Pointcut（切点）** 决定了你的代码会在哪些地方被拦截。而 **`expression`（切点表达式）** 就是筛选方法的“搜索语句”。

#### `expression` 的标准写法：`execution(...)`

最常用的语法格式如下：

> `execution(修饰符? 返回值类型 包名.类名.方法名(参数类型) 异常?)`

- `*`：匹配任意字符。
    
- `..`：匹配任意层级的包，或者匹配任意个数/类型的参数。
    

**常见写法举例：**

|**表达式示例**|**含义**|
|---|---|
|`execution(* com.test.Student.study(..))`|匹配 Student 类中**名为 study** 的任意返回值、任意参数的方法。|
|`execution(public * com.test.*.*(..))`|匹配 com.test 包下**所有类**的**所有公开方法**。|
|`execution(* com.test..*.*(..))`|匹配 com.test 包**及其子包**下所有类的所有方法。|
|`execution(* save*(..))`|匹配所有**以 save 开头**的方法（常用于事务管理）。|

---

### 整体图解

我们将你提到的元素串联起来，看它们是如何“打配合”的：

```xml
<!-- 1. 具体的逻辑实现类 -->
<bean id="studentAOP" class="com.xidian.aspect.StudentAspect" />

<aop:config>
    <!-- 2. 引用上面的 Bean，正式声明这是一个“切面” -->
    <aop:aspect ref="studentAOP">
        
        <!-- 3. 定义切点：这行代码的意思是“我要盯住 Student 类里的所有方法” -->
        <!-- id 相当于起个名字，方便下面引用 -->
        <aop:pointcut id="myPointcut" 
                      expression="execution(* com.xidian.service.Student.*(..))" />

        <!-- 4. 绑定通知：当 myPointcut 匹配的方法执行前，去 studentAOP 里找 logStart 方法 -->
        <aop:before method="logStart" pointcut-ref="myPointcut" />
        
        <!-- 5. 绑定通知：执行后找 logEnd 方法 -->
        <aop:after method="logEnd" pointcut-ref="myPointcut" />

    </aop:aspect>
</aop:config>
```