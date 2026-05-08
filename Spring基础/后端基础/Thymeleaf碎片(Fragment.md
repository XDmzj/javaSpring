
在 Thymeleaf 中，**模板布局（Template Layout）** 是解决代码复用、保持 UI 一致性的核心技术。它允许你将页面中重复的部分（如导航栏、侧边栏、页脚）抽取出来，统一定义和管理。

Thymeleaf 主要通过 **碎片（Fragment）** 的概念来实现布局。

### 1. 核心概念：什么是碎片 (Fragment)？

碎片就是一段可以被其他页面引用的 HTML 代码块。你可以在一个专门的公共文件中定义这些块。

**公共文件：`common/header.html`**

HTML
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <!-- 使用 th:fragment 定义一个名为 "nav" 的碎片 -->
    <div th:fragment="nav">
        <nav>
            <a th:href="@{/}">首页</a>
            <a th:href="@{/books}">图书管理</a>
            <span th:text="${session.user.name}">用户名</span>
        </nav>
    </div>
</body>
</html>
```

---

### 2. 引用碎片的三种方式

当你需要在具体页面（如 `index.html`）引入上面的导航栏时，有三种指令，它们的区别在于“谁留谁走”。

|**指令**|**逻辑描述**|**最终 HTML 效果**|
|---|---|---|
|**`th:insert`**|**插入**：将碎片内容放入当前标签内。|保留原有标签，里面嵌套碎片的标签。|
|**`th:replace`**|**替换**：用碎片标签替换掉当前标签。|原有标签消失，完全由碎片标签代替。|
|**`th:include`**|**包含**：只取碎片的内容，不取碎片的标签。|保留原有标签，里面只放碎片的内容（Servlet 3.0后不推荐）。|

**代码示例：**

HTML
```
<!-- 假设碎片定义为 <div th:fragment="nav">...</div> -->

<div th:insert="~{common/header :: nav}"></div>
<!-- 结果：<div><div>...</div></div> -->

<div th:replace="~{common/header :: nav}"></div>
<!-- 结果：<div>...</div> -->
```

---

### 3. 语法：如何定位碎片？

引用碎片的通用表达式为：`~{模板名 :: 碎片名}` 或 `~{模板名 :: #id选择器}`。

- **`~{common/header :: nav}`**：寻找 `common/header.html` 模板中 `th:fragment="nav"` 的部分。
    
- **`~{common/header :: #main-nav}`**：寻找该模板中 `id="main-nav"` 的部分（无需显式定义 `th:fragment`）。
    
- **`~{ :: nav}`**：寻找当前页面内定义的名为 `nav` 的碎片。
    

---

### 4. 高级技巧：碎片参数传递

你可以像调用函数一样给碎片传递参数，这在处理动态标题或激活菜单项时非常有用。

**碎片定义：**

HTML
```
<div th:fragment="title_part(title_text)">
    <h1 th:text="${title_text}">默认标题</h1>
</div>
```

**页面引用：**

HTML
```
<div th:replace="~{common/header :: title_part('图书管理系统 - 详情页')}"></div>
```


---

### 5. 整体布局方案：Hierarchical Layouts

除了简单的“碎片插入”，如果你想实现“母版页”的效果（即定义一个大框架，每个子页面只填充中间的内容区域），通常有两种做法：

#### A. 传统的“拼凑法”

每个页面手动引入 header、footer、sidebar。

- 优点：简单直观。
    
- 缺点：如果页面结构变了（比如多加了个侧边栏），要改几十个文件。
    

#### B. 布局装饰器（推荐）

结合 `th:replace` 和参数传递，实现父模板控制结构。

**父模板 `layout.html`：**

HTML

```
<html>
<head th:fragment="head(title)">
    <title th:text="${title}">母版标题</title>
</head>
<body>
    <div th:replace="~{common/nav :: nav}"></div>
    
    <!-- 留出一个内容槽位 -->
    <main th:fragment="content">这是默认内容</main>
    
    <footer th:replace="~{common/footer :: foot}"></footer>
</body>
</html>
```

**子页面 `book_list.html`：**

HTML

```
<body th:replace="~{layout :: content}">
    <main th:fragment="content">
        <!-- 这里写图书列表特有的 HTML -->
        <table>...</table>
    </main>
</body>
```

---

### 6. 注意事项

1. **路径问题**：模板路径通常相对于 `templates/` 目录，不需要加 `.html` 后缀。
    
2. **动静结合的权衡**：使用 `th:replace` 时，如果碎片在另一个文件，直接预览子页面 html 可能会看到一片空白。
    
    - **技巧**：在子页面标签里写一些占位内容，Thymeleaf 运行时会把它们刷掉。
        
3. **选择器性能**：使用 `th:fragment` 名称引用比使用 DOM 选择器（如 `#id`）解析速度稍快，建议优先使用碎片名。
    

在你的 `BookManager` 项目中，建议先做一个 `common.html` 存放所有的 CSS 引用和导航栏，然后每个页面用 `th:replace` 引入，这样当你改 UI 风格时，只需要改一个地方就能全站生效。
