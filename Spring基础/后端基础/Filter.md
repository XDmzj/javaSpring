
# Filter

### Filter 的核心定义

**Filter** 是处于客户端与服务器资源（Servlet、JSP、HTML）之间的一个**中间组件**。它可以拦截发送往后台的**请求（Request）**，也可以拦截发往浏览器的**响应（Response）**。

其核心思想是 **“横向抽取”**：将多个 Servlet 中重复的代码（如登录校验、编码设置）提取出来，统一在 Filter 中处理。

### Filter 的工作流程（责任链模式）

1. **拦截请求**：当浏览器请求某个资源时，Filter 首先拿到 Request。
    
2. **执行逻辑**：Filter 检查请求（比如检查 Session 里有没有用户信息）。
    
3. **放行或拦截**：
    
    - **放行**：调用 `chain.doFilter(request, response)`，请求继续传给下一个 Filter 或目标 Servlet。
        
    - **拦截**：不调用 `doFilter`，而是直接通过 Response 重定向到登录页面或返回错误信息。
        
4. **拦截响应**：当 Servlet 处理完返回时，Filter 还可以最后对 Response 进行修饰（比如压缩内容）。


### 在 Servlet 中如何编写一个 Filter

你需要实现 `javax.servlet.Filter` 接口，并重写它的三个方法：
Java
```
@WebFilter("/*") // 拦截所有路径
public class LoginFilter implements Filter {

    public void init(FilterConfig config) {
        // 过滤器创建时执行（服务器启动时）
    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;

        // 1. 执行拦截逻辑：比如检查是否登录
        Object user = req.getSession().getAttribute("user");

        if (user != null || req.getRequestURI().contains("login")) {
            // 2. 满足条件，放行
            chain.doFilter(request, response);
        } else {
            // 3. 不满足条件，重定向到登录页
            resp.sendRedirect(req.getContextPath() + "/login.jsp");
        }
    }

    public void destroy() {
        // 过滤器销毁时执行（服务器关闭时）
    }
}
```

### Filter 的三大典型应用场景

在你的 `BookManager` 项目中，Filter 几乎是必不可少的：

1. **统一设置字符编码**：
    
    不用在每个 Servlet 里写 `request.setCharacterEncoding("UTF-8")`。直接写一个 Filter，对所有请求执行这一句。
    
2. **登录权限校验**：
    
    防止用户直接在地址栏输入 `admin/manage_books.jsp` 绕过登录。Filter 会在进入页面前强制检查 Session。
    
3. **敏感词过滤**：
    
    在用户提交留言或评论时，Filter 可以拦截 Request，把“脏话”替换成 `***` 后再传给 Servlet。


### Filter vs Servlet 的区别

|**特性**|**Filter**|**Servlet**|
|---|---|---|
|**主要职责**|过滤、拦截、预处理|**处理业务逻辑、生成响应**|
|**执行顺序**|在 Servlet 之前执行|在 Filter 之后执行|
|**数量关系**|一个请求可以经过**多个** Filter（过滤器链）|一个请求通常只由**一个** Servlet 处理|
|**能否生成响应**|可以（通常用于报错），但不建议作为主要响应源|专门用于生成响应内容|

### 总结

Filter 就是 Web 开发中的“**预处理器**”。它让你的 Servlet 能够专注于核心业务（如：查询数据库、增删改查），而不用去操心编码、安全检查等琐事。

---

## FilterChain
### 定义

`FilterChain` 是 Servlet 容器（如 Tomcat）提供的一个对象，它只有一个核心方法：`doFilter(request, response)`。

它的作用是：**控制请求在过滤器序列中的流转**。当一个过滤器的任务完成后，它需要询问 `FilterChain`：“下一个是谁？请让请求继续走下去。”

### 它是如何工作的？（责任链模式）

在一个复杂的 Web 应用中，通常会有多个过滤器。例如：

1. **Filter A**: 统一设置编码（UTF-8）。
    
2. **Filter B**: 登录校验。
    
3. **Filter C**: 敏感词过滤。
    

**交互流程如下：**

1. **请求进入**：Tomcat 接收到请求，发现有 3 个过滤器符合拦截条件。
    
2. **执行 Filter A**：
    
    - A 处理完编码逻辑后，调用 `chain.doFilter()`。
        
    - **重点**：此时请求并没有直接去 Servlet，而是被 `chain` 交给了 **Filter B**。
        
3. **执行 Filter B**：
    
    - B 检查登录。如果通过，调用 `chain.doFilter()`，交给 **Filter C**。
        
4. **执行目标资源**：
    
    - 当链条中最后一个 Filter 执行完 `chain.doFilter()` 后，请求才会正式到达目标 **Servlet**
        
5. **响应返回**：
    
    - Servlet 处理完后，响应会沿着这条链**原路返回**，依次经过 Filter C -> B -> A，最后回到浏览器。
        


### 代码中的关键角色

在 Filter 的 `doFilter` 方法中，`chain.doFilter(request, response)` 这行代码具有决定性意义：

Java

```
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // 1. 【请求去程】在此处编写拦截/预处理逻辑
    
    // 2. 【核心动作】
    // 调用此方法：请求向后传递（交给下一个过滤器或 Servlet）
    // 不调用此方法：请求在此“死掉”（拦截），后续过滤器和 Servlet 都不会执行
    chain.doFilter(request, response); 
    
    // 3. 【响应回程】当后续资源处理完，代码会回到这里，执行响应修饰逻辑
}
```


### 过滤器链的顺序由谁决定？

既然是“链”，顺序就很重要。在 Servlet 规范中，顺序通常由以下规则决定：

- **基于配置文件 (`web.xml`)**：按照 `<filter-mapping>` 在文件中的出现先后顺序排列。
    
- **基于注解 (`@WebFilter`)**：这种方式无法精确控制顺序。如果有严格的先后需求（比如必须先设编码再检查登录），建议使用 `web.xml` 或者在 Spring 环境下使用 `FilterRegistrationBean` 来指定 `order` 值。


###  为什么需要 FilterChain？（解耦）

- **单一职责**：每个 Filter 只需要关注自己的业务（A 只管编码，B 只管登录）。
    
- **灵活组合**：你可以像插拔 U 盘一样，随时在 `web.xml` 里增加或删除一个过滤器，而不需要修改 Servlet 的代码。
    
- **动态控制**：`FilterChain` 保证了即使有 100 个过滤器，它们也能有条不紊地按序工作。
    

### 总结

**FilterChain 是连接多个过滤器的纽带。**

在 `BookManager` 项目中，如果既想给请求设置 UTF-8 编码，又想验证用户是否登录，就需要配置两个 Filter。`FilterChain` 会确保请求先被“编码保安”处理，再被“登录保安”检查，最后才进入的“图书管理业务”。




---

# HttpFilter

在 Java Web 开发的历史进程中，普通的 `Filter` 接口处理的是通用的 `ServletRequest` 和 `ServletResponse`。但由于我们绝大多数开发都是基于 HTTP 协议的，为了方便开发者，**HttpFilter** 应运而生。

### 1. 什么是 HttpFilter？

**HttpFilter** 是 Java EE 8（及 Servlet 4.0）引入的一个**工具类**。它实现了 `Filter` 接口，并专门为 HTTP 协议做了封装。

- **它的角色**：它是 `Filter` 接口的一个抽象实现类。
    
- **它的意义**：就像 `HttpServlet` 简化了 `Servlet` 一样，`HttpFilter` 帮我们做好了**类型强转**。
    

在使用普通的 `Filter` 时，你每次都要手动把 `ServletRequest` 强转为 `HttpServletRequest`。而使用 `HttpFilter`，你拿到的参数直接就是 HTTP 相关的对象。


### 2. HttpFilter 的核心方法

`HttpFilter` 继承并实现了 `Filter` 接口，它包含以下核心方法：

#### ① doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain)

这是你最常需要重写（Override）的方法。

- **区别**：注意参数类型！它不再是通用的 `ServletRequest`，而是已经帮你强转好的 `HttpServletRequest`。
    
- **作用**：编写你的过滤逻辑（如登录检查、权限校验）。
    

#### ② doFilter(ServletRequest request, ServletResponse response, FilterChain chain)

这是来自父接口 `Filter` 的原始方法。

- **逻辑**：在 `HttpFilter` 的默认实现中，这个方法已经写好了强转逻辑，并自动调用上面那个带有 `Http` 前缀的 `doFilter` 方法。**通常不需要重写它。**
    

#### ③ init() 和 destroy()

- **init()**：过滤器实例化后立即执行，用于初始化资源（如读取 `web.xml` 配置）。
    
- **destroy()**：过滤器销毁前执行，用于释放资源（如关闭数据库连接）。
    
- 在 `HttpFilter` 中，这两个方法都有默认的空实现，你可以根据需求选择性重写。
    


### 3. 代码对比：普通 Filter vs HttpFilter

通过对比，可以一眼看出 `HttpFilter` 的便利性：

#### 使用普通 Filter (传统方式):

Java
```
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // 每次都要痛苦地手动强转
    HttpServletRequest req = (HttpServletRequest) request;
    HttpServletResponse resp = (HttpServletResponse) response;
    
    String uri = req.getRequestURI();
    // ...
}
```

#### 使用 HttpFilter (更现代的方式):
Java
```
public class MyFilter extends HttpFilter { // 继承 HttpFilter
    @Override
    protected void doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain) 
            throws IOException, ServletException {
        
        // 直接使用 req 和 res，不需要强转！
        String uri = req.getRequestURI();
        
        if (uri.contains("/admin")) {
            // 处理逻辑...
        }
        chain.doFilter(req, res);
    }
}
```


### 4. 总结

`HttpFilter` 实际上就是一个**方便面版**的 `Filter`。

- **如果你在用 Servlet 4.0 或更高版本**：强烈建议直接继承 `HttpFilter`，代码会更简洁、可读性更好。
    
- **如果你在用旧版本**：你只能老老实实继承 `Filter` 接口并手动强转。
