
最初确实是 Tomcat 在调用。但 Tomcat 不能直接调用 Spring 的 Bean，中间有一个非常精妙的“接力棒”流程。**

如果你不理清这个流程，Spring Security 在你眼里永远是一个充满魔法的黑盒。我们用大白话配合刚才讲的源码，把这个“从 Web 服务器（Tomcat）跨界到 Spring 容器”的接力赛彻底复盘一遍。

### 为什么要搞一个“接力”流程？（痛点所在）

- **Tomcat 的世界：** 它是 Servlet 容器，生命周期由 Tomcat 自己管理。它只认识标准的 `Servlet`、`Filter`（过滤器）。它启动的时候，Spring 可能都还没初始化呢，所以 **Tomcat 根本拿不到、也无法直接调用你用 `@Bean` 或 `@Component` 声明的 Spring 内部对象**。
    
- **Spring 的世界：** 它是一个依赖注入（DI）容器，管理着你写的各种 Service、Controller 和 Spring Security 的各种过滤器链。
    

为了让 Tomcat 接收到的网络请求，能正确走到 Spring Security 的门卡里，Spring 祭出了一个“间谍过滤器”。

### 核心接力流程：四大核心角色

整个调用链条就像一个四人接力赛：

#### 第一棒：Tomcat 核心大闸（`ApplicationFilterChain`）

1. 用户在浏览器输入网址或点击登录，Tomcat 接收到底层的 TCP 网络请求，将其解析封装成 `HttpServletRequest`。
    
2. Tomcat 内部有一个过滤器链，叫做 **`ApplicationFilterChain`**。Tomcat 开始依次执行里面的标准 Filter。
    

#### 第二棒：跨界内鬼/桥梁（`DelegatingFilterProxy`）

3. Tomcat 的过滤器链走着走着，来到了一个由 Spring 注册进去的标准 Filter，叫 **`DelegatingFilterProxy`**（由它的名字就能看出：**委托过滤器代理**）。
    
4. 它的 `doFilter()` 方法被 Tomcat 触发了。它自己不干任何安全校验的活，它的唯一使命是：**去 Spring 的 IoC 容器里，通过名字死磕找一个叫 `"springSecurityFilterChain"` 的 Bean。**
    
5. 找到之后，它通过动态代理，把 Tomcat 传给它的 `request` 和 `response` 原封不动地交出去。
    

> 这就是破局的关键： `DelegatingFilterProxy` 拿着 Tomcat 的铁饭碗（生命周期由 Tomcat 管），但灵魂属于 Spring（能去 Spring 容器里捞 Bean）。它成功完成了跨界交接！

#### 第三棒：安全大总管（`FilterChainProxy`）

6. 被捞出来的那个名字叫 `"springSecurityFilterChain"` 的 Bean，本质上就是 **`FilterChainProxy`**。
    
7. 从这一刻起，请求正式脱离 Tomcat 的掌控，进入了 Spring Security 的独立王国。
    
8. `FilterChainProxy` 内部自己维持着一个 `List<SecurityFilterChain>`（就是你用配置类配的那堆东西）。它会遍历这个列表，去执行我们前面提到过的那些具体的安全过滤器（比如查数据库的 `UsernamePasswordAuthenticationFilter`，或者管权限的 `AuthorizationFilter`）。
    

#### 🏃 第四棒：回归正常业务（`DispatcherServlet`）

9. 如果 Spring Security 的这一大堆安全过滤器全部顺利放行（Pass）了，`FilterChainProxy` 就会转头对第二棒说：“我这边检查完了，没问题，你继续吧。”
    
10. `DelegatingFilterProxy` 收到通知，就会调用 Tomcat 过滤器链的 `chain.doFilter(request, response)`，让 Tomcat 继续往下走。
    
11. Tomcat 过滤器链的最后一站，通常就是 Spring MVC 的核心总入口 —— **`DispatcherServlet`**。
    
12. `DispatcherServlet` 拿到请求，再通过反射，最终调用到你写的 `@RestController` 里的具体方法。