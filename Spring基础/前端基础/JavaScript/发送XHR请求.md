

JS的大致内容我们已经全部学习完成了，那么如何使用JS与后端进行交互呢？

我们知道，如果我们需要提交表单，那么我们就需要将表单的信息全部发送给我们的服务器，那么，如何发送给服务器呢？

通过使用XMLHttpRequest对象，来向服务器发送一个HTTP请求，下面是一个最简单的请求格式：

```js
let xhr = new XMLHttpRequest();
xhr.open('GET', 'https://www.baidu.com');
xhr.send();
```

上面的例子中，我们向服务器发起了一次网络请求，但是我们请求的是百度的服务器，并且此请求的方法为GET请求。

我们现在将其绑定到一个按钮上作为事件触发：

```js
function http() {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', 'https://www.baidu.com');
    xhr.send();    
}
```

```html
<input id="button" type="button" onclick="http()">
```

我们可以在网络中查看我们发起的HTTP请求并且查看请求的响应结果，比如上面的请求，会返回百度这个页面的全部HTML代码。

实际上，我们的浏览器在我们输入网址后，也会向对应网站的服务器发起一次HTTP的GET请求。

在浏览器得到页面响应后，会加载当前页面，如果当前页面还引用了其他资源文件，那么会继续向服务器发起请求，直到页面中所有的资源文件全部加载完成后，才会停止。