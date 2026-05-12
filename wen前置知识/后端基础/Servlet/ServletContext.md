


# ServletContext是什么

ServletContext是 Servlet 规范中的一个顶级接口。

- **唯一性**：在一个 Web 应用（如你的 `BookManager`）中，**有且仅有一个** `ServletContext` 对象。
    
- **生命周期**：当 Tomcat 启动并加载你的应用时，它被创建；直到 Tomcat 关闭或应用被卸载，它才会被销毁。
    
- **角色地位**：它是 Servlet 之间通信的 **“公共枢纽”**。


### 它有什么用处？

由于它具有“全局共享”和“生命周期长”的特点，它的用途主要集中在以下方面：

#### ① 全局数据共享（数据中心）

你可以通过 `setAttribute` 在里面存东西，所有的 Servlet、JSP 都能拿到。

- **场景**：统计当前在线人数、网站访问总量、全局配置信息。

```java
// 在 Servlet A 中存入
getServletContext().setAttribute("appName", "图书管理系统"); 键值对的方式进行存储

// 在 Servlet B 中取出
String name = (String) getServletContext().getAttribute("appName");
```



---


# 使用ServletContext


模拟一个简单的场景：**系统启动公告（System Announcement）**。

`InitServlet` 负责从数据库或配置文件读取公告并存入 Context，
而 `DisplayServlet` 负责读取并展示给用户。

###  写入端：InitServlet

这个 Servlet 通常在系统初始化时运行，负责向“大管家”存入数据。

```java
@WebServlet("/init")
public class InitServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 获取 ServletContext 对象
        ServletContext context = this.getServletContext();
        
        // 2. 模拟从某处获取的数据
        String info = "欢迎来到图书管理系统！系统将于今晚 24:00 进行维护。";
        
        // 3. 将数据存入 Context 作用域
        // Key 为 "sysNotice"，Value 为具体的字符串内容
        context.setAttribute("sysNotice", info);
        
        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().write("公告已成功发布到全局作用域！");
    }
}
```


### 读取端：DisplayServlet

无论用户何时访问这个 Servlet，它都能从 Context 中拿到刚才存入的信息。

```java
@WebServlet("/display")
public class DisplayServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 获取同一个 ServletContext 对象
        ServletContext context = this.getServletContext();
        
        // 2. 从 Context 中取出数据
        // 注意：取出来的是 Object 类型，需要根据实际情况强转
        String notice = (String) context.getAttribute("sysNotice");
        
        // 3. 处理逻辑（防止还没写入就读取导致的空指针）
        if (notice == null) {
            notice = "暂无系统公告";
        }
        
        // 4. 展示给用户
        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().write("<h1>系统动态</h1>");
        response.getWriter().write("<p>" + notice + "</p>");
    }
}
```


### 过程中的逻辑演变

1. **容器启动**：Tomcat 启动 `BookManager` 项目，创建唯一的 `ServletContext` 实例。
    
2. **访问 `/init`**：
    
    - `InitServlet` 被调用。
        
    - 它在 `ServletContext` 的内部 Map 结构中存入了 `{"sysNotice": "..."}`。
        
    - 数据现在保存在**服务器内存**中，只要 Tomcat 不关，数据就在。
        
3. **访问 `/display`**：
    
    - `DisplayServlet` 被调用。
        
    - 它去同一个 `ServletContext` 实例里查找名为 `sysNotice` 的 Key。
        
    - 拿到 Value 并渲染到浏览器。