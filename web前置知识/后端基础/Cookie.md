

**Cookie** 是一种由服务器发送给浏览器并保存在本地的**小型文本文件**，它本质上是 HTTP 协议无状态性（Stateless）的补丁，用于在客户端持久化存储特定的状态信息。

以下是从定义、交互机制到 Servlet 实践的严谨分析：

---

### Cookie定义、用处

HTTP 协议是无状态的，这意味着服务器无法识别两次请求是否来自同一个用户。**Cookie** 作为“状态管理”的核心工具，其地位在于充当了服务器给客户端分发的“身份标签”。

- **存储位置**：由浏览器维护，存储在内存（会话 Cookie）或磁盘（持久 Cookie）中。
    
- **数据结构**：以 **Key-Value（键值对）** 形式存在，并包含域名（Domain）、路径（Path）、有效期（Max-Age/Expires）等属性。
    

---

### 浏览器与服务器的交互过程

Cookie 的发挥作用的过程遵循 **“响应设置 -> 本地存储 -> 请求带回”** 的闭环模型。

1. **服务器响应设置（Set-Cookie）**：
    
    当用户访问服务器（如登录成功后），服务器在 HTTP 响应头中加入 `Set-Cookie` 字段。
    
    - _示例_：`Set-Cookie: user_id=12345; Max-Age=3600; HttpOnly`
        
2. **浏览器存储**：
    
    浏览器接收到响应后，根据指令将 Cookie 存储在本地。
    
3. **后续请求带回（Cookie 头）**：
    
    当浏览器再次请求**同域名、同路径**下的资源时，会自动在 HTTP 请求头中带上 `Cookie` 字段。
    
    - _示例_：`Cookie: user_id=12345`
        
4. **服务器识别**：
    
    服务器解析请求头中的 Cookie，从而识别出用户身份或偏好，实现个性化响应。
    

---

### 在 Servlet 中的具体应用

在 Java Web 开发中，Servlet API 对 Cookie 进行了封装，主要通过 `javax.servlet.http.Cookie` 类进行操作。

#### 1.向浏览器发送 Cookie
java
```
// 1. 创建 Cookie 对象
Cookie userCookie = new Cookie("lastLogin", "2026-05-07");

// 2. 设置生命周期（单位：秒）
// 正数：持久化到磁盘；负数：内存存储（关浏览器即失效）；0：立即删除
userCookie.setMaxAge(60 * 60 * 24); 

// 3. 设置安全属性（防止 XSS 攻击获取 Cookie）
userCookie.setHttpOnly(true);

// 4. 由 response 对象带回给浏览器
response.addCookie(userCookie);
```

#### 2.从浏览器读取 Cookie
Java
```
// 1. 通过 request 获取所有 Cookie（注意返回的是数组）
Cookie[] cookies = request.getCookies();

if (cookies != null) {
    for (Cookie c : cookies) {
        // 2. 匹配特定的 Key
        if ("lastLogin".equals(c.getName())) {
            String value = c.getValue();
            System.out.println("上次登录时间：" + value);
        }
    }
}
```

---

### Cookie 的核心属性与安全性

为了严谨地使用 Cookie，必须掌握其控制属性：

| **属性**       | **描述**                                                       |
| ------------ | ------------------------------------------------------------ |
| **Domain**   | 指定哪些域名可以接收该 Cookie（防止跨站信息泄露）。                                |
| **Path**     | 指定哪些路径下的请求会带上该 Cookie（缩小作用范围）。                               |
| **Max-Age**  | 相对过期时间（秒），比传统的 `Expires`（绝对时间）更可靠。                           |
| **HttpOnly** | **关键安全属性**。设置为 `true` 后，JavaScript 无法读取该 Cookie，有效防止 XSS 攻击。 |
| **Secure**   | 设置为 `true` 后，该 Cookie 仅在 HTTPS 安全连接下传输。                      |
