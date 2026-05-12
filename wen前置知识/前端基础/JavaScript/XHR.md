
**XHR**（`XMLHttpRequest`）是浏览器提供的一个 **JavaScript 对象**。它是 Web 开发史上的一座里程碑，正是它的出现，让网页从“死板的文档”变成了“交互式应用”。

# XHR 是什么？

虽然名字里带 `XML`，但它现在几乎可以传输任何类型的数据（最常用的是 **JSON**）。

- **技术定义**：它是浏览器环境（Web API）提供的一个内置对象。
    
- **核心功能**：它允许 JavaScript 在**不刷新整个页面**的情况下，通过 HTTP 协议发送请求并接收服务器响应。
    
- **现代替代品**：现在的开发中，大家更多使用更现代、更简洁的 `Fetch API` 或者封装好的库（如 `Axios`），但它们的底层逻辑和地位与 XHR 是一致的。
    

# 在 Web 服务器模型中的地位

在没有 XHR 之前，Web 模型是“**全量刷新**”式的；有了 XHR 之后，模型进化到了“**局部异步更新**”模式。

## 传统模型（同步）：

1. 用户点击链接。
    
2. 浏览器直接请求服务器。
    
3. 服务器处理完（比如 Servlet 处理登录），返回一个**全新的 HTML 页面**。
    
4. 浏览器整个屏幕白一下，然后重新渲染。
    

> **痛点**：如果你只是点个赞，却要刷新整个网页，体验极差。

## XHR/AJAX 模型（异步）：

1. 浏览器页面已经加载完成。
    
2. JavaScript 创建一个 **XHR 对象**。
    
3. XHR 在后台默默发请求给服务器（比如你的 `LoginServlet`）。
    
4. **服务器只返回数据**（比如一个字符串 `"success"` 或 JSON），而不是整个 HTML。
    
5. JavaScript 拿到数据后，通过 DOM 操作只改变页面上的一小块地方（比如弹个窗提示“登录成功”）。




# 常见语法格式

### 1. 标准异步请求代码

```JavaScript
 1. 创建 XHR 对象
var xhr = new XMLHttpRequest();

 2. 配置请求参数 (方法, URL, 是否异步)
xhr.open('POST', 'http://localhost:8080/BookManager/LoginServlet', true);

 3. 设置请求头 (如果是发送表单数据，这一步至关重要)
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

 4. 定义回调函数 (监听服务器的状态变化)
xhr.onreadystatechange = function() {
	     readyState 4 表示请求已完成
	     status 200 表示服务器响应成功
    if (xhr.readyState === 4 && xhr.status === 200) {
         打印服务器返回的结果 (文本或 JSON)
        console.log("响应内容：" + xhr.responseText);
        
         业务逻辑处理：例如登录成功跳转
        if (xhr.responseText === "success") {
            window.location.href = "index.jsp";
        }
    }
};

 5. 发送请求 (放入请求体数据)
xhr.send('username=test&password=123456');
```

---

### 2. 关键语法解析

- **`xhr.open(method, url, async)`**:
    
    - `method`: 常用 `GET` 或 `POST`。
        
    - `async`: 默认为 `true`。如果设为 `false`，浏览器会卡死直到请求结束（千万别这么做）。
        
- **`xhr.onreadystatechange`**:
    
    - 这是一个事件监听器。由于请求是**异步**的，JS 不会停下来等服务器，所以必须留一个“回拨电话”，等服务器有消息了自动触发这个函数。
        
- **`readyState` 的五个阶段**:
    
    - `0`: 未初始化。
        
    - `1`: 服务器连接已建立（已调用 `open`）。
        
    - `2`: 请求已接收。
        
    - `3`: 请求处理中。
        
    - **`4`: 请求已完成，且响应已就绪。**（这是我们最关心的状态）

