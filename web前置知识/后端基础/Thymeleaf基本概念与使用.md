

作为一名正在进阶 Java 后端的开发者，理解 **Thymeleaf** 是从“传统 JSP 开发”转向“现代 Spring Boot 开发”的关键一步。

## 1. Thymeleaf 的核心概念

**Thymeleaf** 是一个现代的、服务器端的 Java **模板引擎**。它的任务是处理 HTML、XML、JavaScript、CSS 甚至文本，并将后台数据动态地嵌入其中。

它的核心在于 **“动静结合” (Natural Templates)**：

- **静态时**：它就是一个标准的 HTML 文件，前端设计师可以直接在浏览器中打开预览，不会看到乱码或逻辑碎屑。
    
- **动态时**：服务器解析 HTML 标签中的 `th:*` 属性，将后台数据替换进去，生成最终页面。


## 2. 核心语法与模板应用

Thymeleaf 的语法主要通过在 HTML 标签中添加属性来实现。

### 常用基础语法

- **变量输出**：`${...}`。
    `<p th:text="${user.name}">默认名称</p>`（运行后“默认名称”会被 `user.name` 替换）。
    
- **链接表达式**：`@{...}`。
    `<a th:href="@{/order/details(id=${orderId})}">查看</a>`（自动处理 ContextPath）。
    
- **条件判断**：`th:if` / `th:unless`。
    `<div th:if="${user.isAdmin}">管理员可见</div>`。user.isAdmin为真时”管理员可见“会显示在前端
    
- **循环遍历**：`th:each`。
    `<li th:each="book : ${books}" th:text="${book.title}">书名</li>`。

- **分支**：`th:switch`
```html
<div th:switch="${eval}">
    <div th:case="1">我是1</div>
    <div th:case="2">我是2</div>
    <div th:case="3">我是3</div>
</div>
```
没有th:default属性，但是我们可以使用`th:case="*"`来代替:
```html
<div th:case="*">我是Default</div>
```




Thymeleaf还可以进行一些算术运算，几乎Java中的运算它都可以支持：
```html
<div th:text="${value % 2}"></div>
```

同样的，它还支持三元运算：
```html
<div th:text="${value % 2 == 0 ? 'yyds' : 'lbwnb'}"></div>
```

多个属性也可以通过`+`进行拼接，就像Java中的字符串拼接一样，这里要注意一下，字符串不能直接写，要添加单引号：
```html
<div th:text="${name}+' 我是文本 '+${value}"></div>
```



### 模板碎片 (Fragments)
参考[[Thymeleaf碎片(Fragment]]

为了提高代码复用（如每个页面都有相同的导航栏），Thymeleaf 提供了模板碎片功能：

- **定义碎片**：`<div th:fragment="copy">© 2026 BookManager</div>`。
    
- **引用碎片**：`<div th:insert="~{footer :: copy}"></div>`。



----
## 3. 底层运行原理

Thymeleaf 的运行机制与 JSP 有本质区别。JSP 是编译成 Java 类（Servlet）执行，而 Thymeleaf 是基于 **DOM 树解析** 的。

### 运行步骤：

1. **模板读取**：引擎将 HTML 模板文件读取到内存中。
    
2. **建立 DOM 树**：使用解析器（如 HTML5 模式）将 HTML 文本构建成一个文档对象模型（DOM）树。
    
3. **节点遍历与解析**：
    
    - 引擎遍历 DOM 树中的每一个节点。
        
    - 一旦发现带有 `th:` 前缀的属性，引擎就会调用对应的 **处理器 (Processor)**。
        
    - 处理器执行逻辑（如计算表达式、进行循环克隆节点、执行条件分支）。
        
4. **属性移除与替换**：处理完逻辑后，引擎会移除这些自定义的 `th:` 属性，并将结果（文本或新节点）插入到 DOM 树中。
    
5. **渲染输出**：最后将修改后的 DOM 树序列化为纯 HTML 文本，写入 Response 输出流。
    


---

## 总结

Thymeleaf 是一个**基于 DOM 树、逻辑与结构高度分离、支持离线预览**的现代化引擎。