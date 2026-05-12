
源代码：

![[Pasted image 20260506173902.png]]


# 详解
`@WebServlet` 是 Java Servlet 3.0 规范引入的注解，它的出现彻底改变了以往必须在 `web.xml` 中编写大量繁琐配置的局面。

通过这个注解，你可以直接在 Java 类上声明它的访问路径、初始化参数等信息，实现“**零配置**”开发。

## 1. 基本语法与常用属性

要使用该注解，你的类必须继承 `HttpServlet`。

```
@WebServlet(
    name = "MyUserServlet", 
    urlPatterns = {"/user", "/user/info"}, 
    loadOnStartup = 1,
    initParams = {
        @WebInitParam(name = "encoding", value = "UTF-8")
    }
)
public class UserServlet extends HttpServlet { 
    // 实现 doGet 或 doPost
}
```

#### 核心属性详解：

- **`value` / `urlPatterns`**（最常用）：
    
    指定 Servlet 的访问映射路径。它们是等价的。可以配置一个字符串，也可以配置一个字符串数组（如上例，一个 Servlet 可以对应多个路径）。
    
- **`name`**：
    
    指定 Servlet 的名字，相当于 XML 中的 `<servlet-name>`。如果不写，默认是类的全限定名。
    
- **`loadOnStartup`**：
    
    指定 Servlet 的加载时机。
    
    - **负数（默认值 -1）**：Servlet 在第一次被访问时才实例化（懒加载）。
        
    - **正数或 0**：服务器启动时立即实例化。数值越小，优先级越高。
        
- **`initParams`**：
    
    配置初始化参数。可以通过 `getServletConfig().getInitParameter("name")` 获取。
    


## 2. urlPatterns 的匹配规则

路径的写法决定了浏览器如何找到你的 Servlet：

1. **精确匹配**：`@WebServlet("/login")`
    
    - 只有访问 `http://localhost:8080/context/login` 才能匹配。
        
2. **目录匹配**：`@WebServlet("/user/*")`
    
    - 访问 `/user/add`、`/user/delete` 都会交给这个 Servlet 处理。
        
3. **扩展名匹配**：`@WebServlet("*.do")`
    
    - 访问任何以 `.do` 结尾的请求（如 `save.do`）都会触发。**注意：这种写法前面不能加 `/`。**
        
4. **缺省匹配**：`@WebServlet("/")`
    
    - 作为默认 Servlet，当其他路径都匹配不上时，会交给它处理。
        



## 3. 注解与 XML 的映射关系

为了让你直观理解，我们看下注解是如何替代 XML 的：

|**注解属性**|**对应 web.xml 标签**|
|---|---|
|`name = "User"`|`<servlet-name>User</servlet-name>`|
|`urlPatterns = "/u"`|`<url-pattern>/u</url-pattern>`|
|`loadOnStartup = 1`|`<load-on-startup>1</load-on-startup>`|
|`@WebInitParam`|`<init-param>`|


## 4. 原理探秘：Tomcat 是如何识别它的？

虽然你只是写了一个注解，但 Tomcat 在启动时做了一系列“重活”：

1. **扫描 (Scanning)**：Tomcat 启动时会扫描项目路径下的所有 `.class` 文件。
    
2. **解析 (Parsing)**：利用字节码框架（如 ASM）读取类的元数据，一旦发现类上有 `@WebServlet` 注解，就提取其中的属性值。
    
3. **动态注册 (Registration)**：Tomcat 在内部维护一个映射表。它会将解析到的路径映射到该类的实例上。
    

> **注意：** 如果你的 `web.xml` 根标签中设置了 `metadata-complete="true"`，Tomcat 会跳过注解扫描，此时注解将失效。


## 5. 使用建议

- **简单项目**：直接用注解，开发效率极高。
    
- **大型项目/框架开发**：如果 Servlet 路径需要动态修改（比如根据环境切换），或者 Servlet 数量极多且需要统一管理，`web.xml` 依然有其优势。
    
- **冲突处理**：如果同一个路径在注解和 XML 中都配置了，**XML 的优先级更高**。