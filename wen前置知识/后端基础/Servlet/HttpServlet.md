


前面我们已经学习了如何创建、注册和使用Servlet，那么我们继续来深入学习Servlet接口的一些实现类。

这些Servlet实现类的源代码在IDEA中就可以查看，CTRL+鼠标左键点击即可

# GenericServlet

首先`Servlet`有一个直接实现抽象类`GenericServlet`，那么我们来看看此类做了什么事情。

我们发现，这个类完善了配置文件读取和Servlet信息相关的的操作，但是依然没有去实现service方法，因此此类仅仅是用于完善一个Servlet的基本操作，


# HttpServlet
那么我们接着来看`HttpServlet`，它是遵循HTTP协议的一种Servlet，继承自`GenericServlet`，它根据HTTP协议的规则，完善了service方法。

在阅读了HttpServlet源码之后，我们发现，其实我们只需要继承HttpServlet来编写我们的Servlet就可以了，并且它已经帮助我们提前实现了一些操作，这样就会给我们省去很多的时间。

```java
@Log
@WebServlet("/test")
public class TestServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().write("<h1>恭喜你解锁了全新玩法</h1>");
    }
}
```

现在，我们只需要重写对应的请求方式，就可以快速完成Servlet的编写。