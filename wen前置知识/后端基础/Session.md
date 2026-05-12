
# Session基本概念

**Session**（会话）是 Web 开发中用于在**服务器端**保存用户状态的机制。

如果说 **Cookie** 是服务器发给你的“通行证”（由你随身携带），那么 **Session** 就是服务器为你开设的“私人储物柜”（由服务器保管，你只持有钥匙）。

### Session 的核心原理（“钥匙与柜子”）

由于 HTTP 协议是无状态的，为了识别用户，Session 采取了以下交互流程：

1. **创建柜子**：当用户第一次访问服务器时，服务器在内存中开辟一块空间（Session 对象），并生成一个唯一的 ID（**JSESSIONID**）。
    
2. **发放钥匙**：服务器通过响应头（`Set-Cookie`），将这个 `JSESSIONID` 发送给浏览器。
    
3. **对号入座**：浏览器之后的每次请求都会自动带上这个带有 ID 的 Cookie。服务器根据 ID 找到对应的“储物柜”，从而获取该用户的数据。
    


### 在 Servlet 中如何使用

在 Java Servlet 中，我们通过 `HttpServletRequest` 对象来操作 Session。

#### A. 获取/创建 Session

Java

```
// 如果当前有 Session 则返回，没有则创建一个新的
HttpSession session = request.getSession();
```

#### B. 存储与读取数据

Java

```
// 存储用户信息（存入储物柜）
session.setAttribute("user", userObject);

// 读取用户信息（从储物柜取出）
User user = (User) session.getAttribute("user");
```

#### C. 销毁 Session

Java

```
// 立即销毁当前 Session（如：退出登录）
session.invalidate();
```





### Session 的生命周期

- **创建时刻**：第一次执行 `request.getSession()` 时。
    
- **销毁时刻**：
    
    1. **超时销毁**：用户长时间没有操作（默认通常是 30 分钟）。可以在 `web.xml` 中配置。
        
    2. **手动销毁**：调用 `invalidate()` 方法。
        
    3. **服务器关闭**：服务器停止运行。
        
- **注意**：关闭浏览器**并不会**立即销毁服务器端的 Session，只是浏览器把那把“钥匙”（Cookie）丢了。等 Session 过期时间到了，服务器才会清理掉它。
    


### Session vs Cookie 的严谨对比

|**特性**|**Cookie**|**Session**|
|---|---|---|
|**存储位置**|客户端（浏览器）|**服务器端**（内存/数据库）|
|**安全性**|较低（易被伪造或截获）|**较高**（敏感信息不发给客户端）|
|**存储容量**|很小（通常限制 4KB）|**很大**（受服务器内存限制）|
|**数据类型**|只能存字符串|**可以存任何 Java 对象**|
|**对服务器压力**|无压力|压力大（用户多时占用大量内存）|


### 实际开发中的典型场景

在你的 `BookManager` 项目中，Session 最常见的用途是：

- **登录状态维护**：用户登录成功后，将 `User` 对象存入 Session。之后在每一个需要权限的页面（如：修改图书）判断 `session.getAttribute("user")` 是否为空。
    
- **验证码校验**：生成验证码图片时，将正确答案存在 Session 中，等用户提交表单时取出对比。
    
- **购物车**：虽然现在大型电商多用数据库存储，但在小型演示项目中，Session 是实现购物车的理想场所。
    

---

### 安全建议

虽然 Session 比 Cookie 安全，但仍存在 **Session 劫持** 的风险（即攻击者偷走了你的 `JSESSIONID`）。

- **HttpOnly**：确保 `JSESSIONID` 的 Cookie 设置了 `HttpOnly`，防止 JS 读取。
    
- **强制校验**：在敏感操作前，除了校验 Session，还可以结合用户 IP 或浏览器指纹进行二次验证。




# Session使用时浏览器和服务器的交互流程



### 交互流程的严谨拆解

在物理层面上，**Session 对象从未离开过服务器**。

1. **Request 到达**：浏览器发起请求。
    
2. **创建/检索**：服务器执行 `request.getSession()`。
    
    - 如果请求头里没有 `JSESSIONID`，服务器在内存里创建一个 **HttpSession 对象**。
        
    - 如果请求头里带了 `JSESSIONID`，服务器就在自己的内存里找对应的那个对象。
        
3. **返回“钥匙”**：服务器通过 Response 返回给浏览器的**不是对象本身**，而是包含 `JSESSIONID` 的 **Set-Cookie 响应头**。
    
4. **本地存储**：浏览器把这个 ID 存入自己的 Cookie 管理器中。


