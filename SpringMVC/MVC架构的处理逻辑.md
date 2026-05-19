在 Spring MVC 架构中，当浏览器发起一个 HTTP 请求时，整个体系会遵循一套严谨的前端控制器模式（Front Controller Pattern）进行运转。各核心元素职责分明，高度解耦。

---

### 一、 核心元素交互流程（请求正常响应）

当浏览器发送一个合法且服务器能处理的请求时，Spring MVC 内部的交互时序如下：

#### 1. 核心大门：`DispatcherServlet`（前端控制器）

- **反应：** 浏览器发出的 HTTP 请求首先会被 Servlet 容器（如 Tomcat）捕获，并根据 URL 映射规则分发给 `DispatcherServlet`。它是整个生命周期的核心调度枢纽，自身不处理具体业务，而是负责控制整个执行流程。
    

#### 2. 指路牌：`HandlerMapping`（处理器映射器）

- **反应：** `DispatcherServlet` 接收到请求后，将其传递给 `HandlerMapping`。`HandlerMapping` 根据请求的 URL、HTTP 方法（GET/POST）等信息，在内部的映射注册表（通常由 `@RequestMapping` 解析而来）中查找对应的处理器。
    
- **返回：** 它不会直接返回一个简单的类，而是返回一个 **`HandlerExecutionChain`（处理器执行链）**，该链条包含具体的处理器（Controller 方法）以及一组针对该 URL 的**拦截器（HandlerInterceptor）**。
    

#### 3. 关卡过滤：`HandlerInterceptor`（拦截器前置处理）

- **反应：** 在进入实际业务之前，执行链中的拦截器会按照顺序执行其 `preHandle()` 方法。如果任何一个拦截器返回 `false`（例如未通过权限或登录校验），请求在此处被拦截中断，直接向浏览器返回响应。
    

#### 4. 执行适配：`HandlerAdapter`（处理器适配器）

- **反应：** 当拦截器全部放行后，`DispatcherServlet` 将处理器交给 `HandlerAdapter`。由于 Controller 的方法签名各不相同（参数类型、返回值类型各异），`DispatcherServlet` 无法直接通过硬编码调用。`HandlerAdapter` 运用适配器模式，负责**参数绑定**（将 HTTP 请求参数、JSON 报文转换并封装为 Java 对象），然后真正调用 Controller 的目标方法。
    

#### 5. 业务实现：`Controller`（后端控制器）

- **反应：** 此时程序正式进入用户编写的 `Controller` 层面。`Controller` 接收到适配器组装好的参数，执行相应的内部逻辑（在此处会调用 Service 层的业务组件进行数据处理，但这不属于 MVC 的范畴）。
    
- **返回：**
    
    - _传统模式：_ 方法执行完毕后，返回一个 `ModelAndView` 对象（包含模型数据与逻辑视图名）。
        
    - _前后端分离模式（带有 `@ResponseBody`）：_ 方法直接返回普通的 Java 对象或集合。
        

#### 6. 结果后处理与渲染

- **在传统模式下：** `HandlerAdapter` 将 `ModelAndView` 交回 `DispatcherServlet`。随后调度 **`ViewResolver`（视图解析器）** 将逻辑视图名（如 `"index"`）转换为物理视图对象（`View`），并结合 Model 数据进行页面渲染，最终生成 HTML 传给浏览器。
    
- **在前后端分离模式下：** `HandlerAdapter` 会直接调度 **`HttpMessageConverter`（消息转换器，如 Jackson）**，将 Controller 返回的 Java 对象序列化为标准 JSON 字符串，绕过视图解析，直接写入 HTTP 响应体（Response Body）。
    

#### 7. 拦截器后置处理

- **反应：** 无论哪种模式，在流程即将结束、页面或数据即将响应给浏览器之前，拦截器的 `postHandle()` 和 `afterCompletion()` 方法会逆序执行，用于清理资源或记录日志。
    

---

### 二、 边界与异常状况处理机制

当浏览器发起不合法的请求，或者服务器内部无法承载该请求时，Spring MVC 会通过以下机制进行反应：

#### 1. 状况一：浏览器访问了不存在的 URL（404 Not Found）

当用户输入了一个服务器上没有映射的地址（例如访问 `/abc`，但没有任何 Controller 配置了该路径）：

- **反应链条：** `DispatcherServlet` 接收请求 $\rightarrow$ 询问 `HandlerMapping` $\rightarrow$ `HandlerMapping` 在注册表中遍历后，**无法匹配到任何对应的 Handler**，向 `DispatcherServlet` 返回 `null`。
    
- **默认行为：** `DispatcherServlet` 发现没有处理器能接单，会直接调用底层的 `noHandlerFound()` 方法。
    
    - 在标准的 Servlet 规范下，它会丢给容器（Tomcat）的默认处理器，最终由容器向浏览器写回一个 **HTTP 状态码 404** 的错误页面。
        
    - 在现代配置中，如果开启了全局异常处理器（`@ControllerAdvice`），此处的 404 异常（`NoHandlerFoundException`）可以被捕捉，并由后端统一格式化为 JSON 提示信息返回给前端。
        

#### 2. 状况二：服务器不支持该 HTTP 请求方法（405 Method Not Allowed）

当用户通过浏览器发送了一个 `POST` 请求，但是后端的 Controller 对应路径明确限制了只接收 `GET` 请求（例如使用了 `@GetMapping`）：

- **反应链条：** `DispatcherServlet` 接收请求 $\rightarrow$ `HandlerMapping` 进行匹配。此时 `HandlerMapping` 能够根据 URL 找到对应的类，但比对 HTTP Method 时发现不匹配。
    
- **抛出异常：** Spring MVC 底层会明确抛出一个 **`HttpRequestMethodNotSupportedException`** 异常。
    
- **最终响应：** 该异常会触发内置的异常解析器，将 HTTP 响应状态码设为 **405**，并在响应头（Headers）中附带 `Allow: GET`，明确告知浏览器：该 URL 存在，但你使用了错误的请求方式。
    

#### 3. 状况三：服务器内部发生故障（500 Internal Server Error）

当 URL 正确、请求方式也正确，但是在执行到 `HandlerAdapter` 调用 `Controller`（或其下层的 Service/Dao）时，代码内部抛出了未捕获的运行时异常（如 `NullPointerException`、数据库连接超时等）：

- **反应链条：** 异常在 Controller 中产生并向上抛出，被 `HandlerAdapter` 捕获并继续向上抛给 `DispatcherServlet`。
    
- **救场组件：`HandlerExceptionResolver`（异常解析器）**
    
    - `DispatcherServlet` 收到异常后，不会直接崩溃，而是立刻转交给配置好的 `HandlerExceptionResolver`（例如基于 `@ExceptionHandler` 注解的 `ExceptionHandlerExceptionResolver`）。
        
    - 如果程序员编写了自定义的异常拦截方法，程序会执行该方法并返回一个安全的错误提示（JSON 或兜底错误页）。
        
    - 如果系统中**没有任何**异常处理器去捕获这个异常，它将最终透传给 Tomcat 容器，Tomcat 会强行向浏览器返回 **HTTP 状态码 500**，代表服务器内部严重错误。

