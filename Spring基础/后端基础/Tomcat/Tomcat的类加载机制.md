
# 层级详细解析

从上往下，每一层都有其特定的职责和可见性范围：
![[Pasted image 20260508182119.png]]

#### ① 顶层：JVM 固有加载器

- **Bootstrap ClassLoader**：加载 Java 核心库（如 `rt.jar`、`java.lang.*`）。这是所有加载器的基石。
    
- **Extension ClassLoader**：加载 `jre/lib/ext` 目录下的扩展包。
    
- **Application (System) ClassLoader**：加载系统环境变量 `CLASSPATH` 指定的类，以及 Tomcat 启动脚本（`bootstrap.jar`）。
    

#### ② 中间层：Tomcat 共享层

- **Common ClassLoader**：
    
    - **加载位置**：Tomcat 安装目录下的 `lib/` 文件夹。
        
    - **可见性**：对 **Tomcat 容器本身**和**所有 Web 应用**都可见。如果你有所有项目都要用的通用 Jar 包，可以放在这里。
        
- **Catalina ClassLoader** (私有)：加载 Tomcat 服务器内部运行所需的类。这些类对 Web 应用是**不可见**的，实现了容器与应用的隔离。**Catalina** 是 Tomcat 核心引擎的名字。这个加载器的存在是为了**保护服务器自身**。
    
- **Shared ClassLoader** (私有)：如果多个 Web 应用需要共享某些类，但又不希望 Tomcat 容器看到这些类，会用到它（默认配置中，它与 Common 是同一个）。**Shared** 加载器是为了解决**多个应用重复占用内存**的问题

在 Tomcat 8.5 及以后的默认配置中，**Shared ClassLoader**与**Catalina ClassLoader** 通常与 `Common` 加载器合并为同一个，但在架构逻辑上，它们扮演着截然不同的角色

#### ③ 底层：Web 应用隔离层（核心）

- **Webapp ClassLoader**：
    
    - **独特性**：**每个 Web 应用（WAR 包）都有自己独立的 Webapp ClassLoader 实例**。
        
    - **加载位置**：`/WEB-INF/classes` 和 `/WEB-INF/lib`。
        
    - **作用**：这是实现隔离的关键。WebApp A 里的类永远不会被 WebApp B 看到。




# 加载逻辑详解


## 加载顺序的“反常规”逻辑

这是 Tomcat 类加载机制中最严谨的细节。对于 **Webapp ClassLoader**，它的搜索顺序如下：

1. **JVM 核心类**：首先委派给 Bootstrap 加载器，确保像 `System`、`String` 这种类不会被篡改。
    
2. **本地仓库（Local First）**：尝试在 `WEB-INF/classes` 和 `WEB-INF/lib` 中寻找。**这打破了双亲委派**，因为它优先于父类加载器。
    
3. **向上委派**：如果本地找不到，再依次询问 Common、System、Extension 加载器。
