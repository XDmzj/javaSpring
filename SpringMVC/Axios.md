在现代前端开发中，**Axios 异步请求**是前端用来向后端“要数据”或“送数据”的最主流方式。

简单来说，**Axios** 是一个基于 Promise 的网络请求库（可以运行在浏览器和 Node.js 中），而 **异步请求（Asynchronous Request）** 则是一种聪明的通信方式，能让网页在“不刷新整体页面”的情况下，偷偷在后台和服务器交换数据。

为了让你彻底搞懂它，我们可以把它拆成两部分来看：

### 1. 什么是“异步请求”？（为什么需要它）

在传统的网页中，所有的请求都是**同步**的。这意味着如果你想看新的数据（比如点击“下一页”），浏览器就必须重新加载、刷新整个页面。

- ❌ **同步的痛苦：** 整个网页会瞬间变白，所有之前加载好的图片、输入框里的文字都会消失，体验极差。
    

而**异步请求**（常说的 Ajax 技术）打破了这种死板的规则。

- **异步的快乐：** 当你点击“点赞”或者“加载更多”时，浏览器会在后台悄悄给后端发一个请求。在此期间，**你依然可以正常滚动网页、打字输入，网页不会卡死，更不会白屏**。当后端把数据传回来时，前端只会精准地更新网页中的某一个小区域。
    

### 2. 什么是 Axios？

在 Axios 出现之前，前端主要用原生的 `XMLHttpRequest`、jQuery 的 `$.ajax` 或者浏览器原生的 `fetch` 来发请求。

**Axios 能够脱颖而出，成为当今 Vue、React 等框架的标配，是因为它非常好用：**

- **基于 Promise：** 可以完美使用现代 JavaScript 的 `async/await` 语法，写出来的异步代码就像写同步代码一样直观。
    
- **自动转换 JSON：** 后端传过来的 JSON 字符串，Axios 会自动帮你转成 JavaScript 对象，不需要你手动 `JSON.parse()`。
    
- **强大的拦截器（Interceptor）：** 没错，**前端也有拦截器**！Axios 可以在请求发送前（比如统一加上 Token 登录令牌），或者在响应回来后（比如统一处理 401、500 错误）进行拦截。
    

### Axios 异步请求代码长什么样？

在实际开发中，我们通常配合 `async` 和 `await` 来发送 Axios 异步请求。

#### 示例 A：向后端获取数据（GET 请求）

比如你要在网页上展示图书列表，网页一加载，就异步去后端请求数据：



```javascript
// 使用 async 声明这是一个异步函数
async function getBookList() {
    try {
        // await 会“等待”请求结果回来，期间浏览器不会卡死
        const response = await axios.get('/api/books');
        
        // 请求成功回来后，直接拿到后端传过来的数据
        console.log("拿到图书列表啦：", response.data);
        
        // 接下来用这些数据去局部更新你的 Vue/React 页面，实现不刷新网页更新内容
    } catch (error) {
        // 如果网络断了或者后端报错（比如 500），会走到这里
        console.error("请求失败了：", error);
    }
}
```

#### 示例 B：向后端提交数据（POST 请求）

比如用户在前端填好表单后，点击“注册”：

```javascript
async function registerUser(username, password) {
    // 异步把对象传给后端，网页不需要刷新
    const response = await axios.post('/api/register', {
        username: username,
        password: password
    });
    
    if(response.data.success) {
        alert("注册成功！");
    }
}
```



前端通过 Axios 在后台默默地和你的 Spring MVC 控制器（Controller）进行数据交互，从而让用户享受到丝滑、不卡顿、不需要频繁刷新的现代网页体验。

Axios 的代码本质上是 JavaScript（JS）代码，但它既可以写在独立的 `.js` 文件中，也可以写在 `.html` 文件里的 `<script>` 标签中。