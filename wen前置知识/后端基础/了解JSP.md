
**JSP（Java Server Pages）** 是一种由 Sun Microsystems 公司倡导的**动态网页技术标准**。

如果在 Servlet 里用 `response.getWriter().write("<html>...</html>")` 拼装 HTML 页面太痛苦，JSP 就是为了解决这个问题而生的。

它允许在 HTML 代码中直接嵌入 Java 代码。

### 1. JSP 的本质是什么？

**JSP 的本质就是一个 Servlet。**

虽然你写的是 `.jsp` 后件的文件，看起来像网页，但在服务器运行时，它会被容器（如 Tomcat）转换成一个 Java 类（即 Servlet），编译并执行。

- **HTML 部分**：负责页面的结构布局和静态内容。
    
- **Java 部分**：负责逻辑处理（从数据库取书、判断用户权限等）。
    

### 2. JSP 的执行/加载规则（核心流程）

JSP 的加载并不是简单的读取，它经历了一个从“文本文件”到“运行对象”的复杂转化过程。这个过程通常发生在**第一次访问该 JSP** 时：

#### 第一步：代码转换（Translation）

当你第一次请求 `index.jsp` 时，Tomcat 会把这个 JSP 文件转换成一个 `.java` 源文件。

- 所有的 HTML 内容会被包裹在 `out.write()` 方法中。
    
- JSP 脚本（`<% ... %>`）里的代码会原封不动地放进 Servlet 的 `_jspService` 方法里。
    

#### 第二步：编译（Compilation）

容器调用 Java 编译器，将生成的 `.java` 源文件编译成字节码文件（`.class`）。

#### 第三步：类加载与实例化

JVM 将编译后的类加载到内存中，并创建该 Servlet 的实例。

#### 第四步：初始化与服务

1. 调用 `jspInit()` 方法（初始化）。
    
2. 调用 `_jspService()` 方法，处理当前的 HTTP 请求，并生成 HTML 返回给浏览器。
    

### 3. “加载规则”的几个关键特性

1. **第一次访问惩罚**： 由于上述转换和编译过程非常耗时，你会发现部署项目后，**第一次**打开 JSP 页面会比较慢。之后因为类已经驻留在内存中，访问速度会显著提升。
    
2. **热部署（自动更新）**： 如果你在项目运行时修改了 `index.jsp` 的内容，Tomcat 会检测到文件的修改时间。它会**自动触发重新转换和编译**。这意味着你不需要重启服务器就能看到页面的改动。
    
3. **缓存机制**： 转换后的 `.java` 和 `.class` 文件通常存储在 Tomcat 安装目录的 `work` 文件夹下。如果你遇到了莫名其妙的页面显示问题，有时候清理一下 `work` 目录会有奇效。
    

### 4. JSP 的主要组成部分

在编写 JSP 时，你会用到这些规则：

- **指令 (`<%@ ... %>`)**：设置页面属性，如编码 `contentType="text/html;charset=UTF-8"`。
    
- **脚本段 (`<% ... %>`)**：编写普通的 Java 代码。
    
- **表达式 (`<%= ... %>`)**：快速输出变量值到页面，相当于 `out.print()`。
    
- **声明 (`<%! ... %>`)**：定义类的成员变量或方法（较少用）。
    
- **注释 (`<%-- ... --%>`)**：JSP 特有的注释，不会发送到客户端。

示例代码：
```jsp
<%-- 1. 指令 (Directives): 设置页面属性、导入包、引入标签库 --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="UTF-8" %>
<%@ page import="java.util.*, java.text.SimpleDateFormat" %>
<%-- 引入 JSTL 核心标签库 (通常需要导入 standard.jar 和 jstl.jar) --%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!DOCTYPE html>
<html>
<head>
    <title>JSP 语法全解析 - BookManager</title>
    <style>
        .highlight { color: #2c3e50; font-weight: bold; }
        .footer { margin-top: 20px; font-size: 0.8em; color: gray; }
    </style>
</head>
<body>

    <h2>JSP 核心语法示例</h2>

    <%-- 2. 声明 (Declarations): 定义成员变量或方法 (在 Servlet 类中定义) --%>
    <%! 
        private int visitCount = 0; // 这是一个成员变量，所有线程共享
        public String getSystemStatus() {
            return "服务器运行正常";
        }
    %>

    <%-- 3. 脚本段 (Scriptlets): 编写业务逻辑 (在 _jspService 方法中执行) --%>
    <%
        // 逻辑处理
        visitCount++;
        Date now = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String formattedDate = sdf.format(now);
        
        // 模拟从数据库获取的图书列表
        List<String> books = Arrays.asList("Java并发编程实战", "深入理解JVM", "Spring实战");
        // 将数据存入请求域，供 EL 表达式使用
        request.setAttribute("bookList", books);
    %>

    <%-- 4. 表达式 (Expressions): 直接输出变量值到页面 --%>
    <p>当前系统时间：<span class="highlight"><%= formattedDate %></span></p>
    <p>本页面累计被访问次数：<span class="highlight"><%= visitCount %></span></p>
    <p>系统状态：<%= getSystemStatus() %></p>

    <hr>

    <%-- 5. 现代推荐用法：EL 表达式 (Expression Language) --%>
    <h3>图书列表 (使用 JSTL + EL):</h3>
    <ul>
        <%-- 使用 JSTL 的 c:forEach 循环遍历 --%>
        <c:forEach var="book" items="${bookList}">
            <li>书名：<span class="highlight">${book}</span></li>
        </c:forEach>
    </ul>

    <%-- 6. JSP 标准动作 (Standard Actions) --%>
    <p>
        <%-- 动态包含另一个页面 --%>
        <jsp:include page="footer_info.jsp" />
    </p>

    <div class="footer">
        <%-- 7. JSP 注释: 客户端（右键查看源代码）不可见 --%>
        <%-- 这是一个内部逻辑注释，只存在于 .jsp 文件中 --%>
        &copy; 2026 BookManager 系统
    </div>

</body>
</html>
```