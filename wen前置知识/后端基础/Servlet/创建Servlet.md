
# 创建Servlet


## 注解创建

那么如何创建一个Servlet呢，非常简单，我们只需要实现`Servlet`类即可，并添加注解`@WebServlet`来进行注册。

```java
@WebServlet("/test")
public class TestServlet implements Servlet {
		...实现接口方法
}
```

我们现在就可以去访问一下我们的页面：[http://localhost:8080/test/test](http://localhost:8080/test/test)

我们发现，直接访问此页面是没有任何内容的，这是因为我们还没有为该请求方法编写实现，这里先不做讲解，后面我们会对浏览器的请求处理做详细的介绍。


## .xml文件创建
除了直接编写一个类，我们也可以在`web.xml`中进行注册，现将类上`@WebServlet`的注解去掉：

```xml
<servlet>
    <servlet-name>test</servlet-name>
    <servlet-class>com.example.webtest.TestServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>test</servlet-name>
    <url-pattern>/test</url-pattern>
</servlet-mapping>
```

这样的方式也能注册Servlet，但是显然直接使用注解更加方便，因此之后我们一律使用注解进行开发。只有比较新的版本才支持此注解，老的版本是不支持的哦。


## 默认Servlet

实际上，Tomcat服务器会为我们提供一些默认的Servlet，也就是说在服务器启动后，即使我们什么都不编写，Tomcat也自带了几个默认的Servlet，他们编写在conf目录下的web.xml中：

```xml
<!-- The mapping for the default servlet -->
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- The mappings for the JSP servlet -->
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>
```

我们发现，默认的Servlet实际上可以帮助我们去访问一些静态资源，这也是为什么我们启动Tomcat服务器之后，能够直接访问webapp目录下的静态页面。

我们可以将之前编写的页面放入到webapp目录下，来测试一下是否能直接访问。