
它不是一种新技术，也不是一种具体的代码框架，而是一套“设计指南”。满足这套指南要求的接口架构，我们就称之为 **RESTful 架构**。

在没有 RESTful 之前，后端的接口命名非常混乱。

---

### 一、 传统接口设计 vs RESTful 接口设计

假设你现在正在写你的 `BookManager` 项目，需要对“图书（book）”进行增删改查。

#### 1. 传统的命名方式（混乱、动作泛滥）

在传统设计中，程序员习惯把动作（动词）写进 URL 路径里，且请求方式几乎全用 `GET` 或 `POST`：

- 新增图书：`POST /api/addBook` 或 `POST /api/create_book`
    
- 删除图书：`GET /api/deleteBook?id=1` 或 `POST /api/removeBook`
    
- 修改图书：`POST /api/updateBook`
    
- 查询图书：`GET /api/findBook?id=1`
    

> **传统痛点：** 每个人、每个公司的命名习惯都不同（有人用 add，有人用 create）。前端开发人员在调用接口时，必须死记硬背或者频繁查阅极其冗长的 API 文档，沟通成本极高。

#### 2. RESTful 的命名方式（干净、优雅、统一）

RESTful 提出了一个核心颠覆性思想：**在网络世界中，一切皆是“资源（Resource）”。URL 应该只代表资源的位置，而不应该包含任何动词！对资源的操作，应该由 HTTP 的“请求方式（Method）”来决定。**

同样的图书增删改查，在 RESTful 风格下会变成这样：

|**操作**|**HTTP 请求方式 (Method)**|**统一的资源定位 (URL)**|**释义**|
|---|---|---|---|
|**新增**|**`POST`**|`/api/books`|往图书集合里“邮寄”一个新的|
|**删除**|**`DELETE`**|`/api/books/1`|删掉路径上 ID 为 1 的那本图书|
|**修改**|**`PUT`**|`/api/books/1`|更新/替换路径上 ID 为 1 的那本图书|
|**查询单本**|**`GET`**|`/api/books/1`|获取路径上 ID 为 1 的那本图书|
|**查询列表**|**`GET`**|`/api/books`|获取所有图书的列表|

> **RESTful 的核心魅力：** **URL 终身不变，只变动词。** 前端开发人员只要看到 `/api/books/1`，就知道这是在对 ID 为 1 的图书进行操作。发 `GET` 就是查，发 `DELETE` 就是删，不需要看文档也能猜个八九不离十。

---

### 二、 RESTful 的三大核心规范

要设计出一个纯正的 RESTful 接口，程序员在写代码时需要遵守以下三个基本法：

#### 1. 资源用“名词”复数表示

URL 中代表资源的单词一律使用名词，且推荐使用复数形式。

- ❌ 错误：`/api/getUsers`（带了动词）、`/api/user/show/1`
    
- 正解：`/api/users`、`/api/users/1`
    

#### 2. 善于利用“路径变量（`@PathVariable`）”传递 ID

传统方式喜欢用 Query 参数传参（`?id=1`），而 RESTful 强调用斜杠（`/`）把资源 ID 嵌进路径里，作为资源定位的一部分。

- ❌ 传统：`/api/books?id=102`
    
- 正解：`/api/books/102`
    

#### 3. 状态码与统一 JSON 响应

RESTful 要求服务器返回正确的 HTTP 状态码（Status Code）来直观反映结果，而不是一律返回 `200 OK`：

- **`200 OK`**：成功查询或修改了数据。
    
- **`201 Created`**：成功新增了数据（常用于 POST）。
    
- **`401 Unauthorized`**：客户端未登录或 Token 失效。
    
- **`403 Forbidden`**：已登录但权限不足（比如普通用户想删数据）。
    
- **`404 Not Found`**：用户请求的资源路径或者具体的 ID 压根不存在。
    

同时，后端返回的数据格式一律使用标准的 **JSON 报文**。

---

### 三、 程序员在 Spring MVC 源代码里怎么写 RESTful？

既然我们上一节聊到了 Spring MVC 里的各种注解，下面这段代码就是专门用来承载 RESTful 风格接口的。请注意这些注解打的位置：

Java

```java
package org.example.controller;

import org.example.dto.BookDTO;
import org.springframework.web.bind.annotation.*;

@RestController // 1. 确保所有方法直接返回 JSON 数据
@RequestMapping("/api/books") // 2. 抽取统一的资源名词复数
public class BookRestController {

    // 【查】GET /api/books/102 
    @GetMapping("/{id}")
    public String getBook(@PathVariable("id") Long id) { // 绑定路径变量
        return "成功获取ID为 " + id + " 的图书信息";
    }

    // 【增】POST /api/books
    @PostMapping
    public String addBook(@RequestBody BookDTO bookDTO) { // 接收前端的 JSON 报文
        return "成功创建图书：" + bookDTO.getTitle();
    }

    // 【改】PUT /api/books/102
    @PutMapping("/{id}")
    public String updateBook(@PathVariable("id") Long id, @RequestBody BookDTO bookDTO) {
        return "成功修改ID为 " + id + " 的图书数据";
    }

    // 【删】DELETE /api/books/102
    @DeleteMapping("/{id}")
    public String deleteBook(@PathVariable("id") Long id) {
        return "成功删除ID为 " + id + " 的图书";
    }
}
```