监听器并不是学习的重点内容，那么什么是监听器呢？

如果我们希望，在应用程序加载的时候，或是Session创建的时候，亦或是在Request对象创建的时候进行一些操作，那么这个时候，我们就可以使用监听器来实现。
![[Pasted image 20260508143623.png]]


默认为我们提供了很多类型的监听器，我们这里就演示一下监听Session的创建即可：

java复制代码

```java
@WebListener
public class TestListener implements HttpSessionListener {
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        System.out.println("有一个Session被创建了");
    }
}
```

有关监听器相关内容，了解即可。