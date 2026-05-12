

# 创建parser

Spring官方为我们提供了一套非常高级SpEL表达式，通过使用表达式，我们可以更加灵活地使用Spring框架。

首先我们来看看如何创建一个SpEL表达式：

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");  //使用parseExpression方法来创建一个表达式
System.out.println(exp.getValue());   //表达式最终的运算结果可以通过getValue()获取
```

这里得到的就是一个很简单的 Hello World 字符串，字符串使用单引号囊括，SpEL是具有运算能力的。


# 对表达式进行操作

我们可以像写Java一样，对这个字符串进行各种操作，比如调用方法之类的：

```java
Expression exp = parser.parseExpression("'Hello World'.toUpperCase()");   //调用String的toUpperCase方法
System.out.println(exp.getValue());
```

![image-20221125173157008](https://oss.itbaima.cn/internal/markdown/2022/11/25/PZmheYn5EVTvURN.png)

不仅能调用方法、还可以访问属性、使用构造方法等，是不是感觉挺牛的，居然还能这样玩。

对于Getter方法，我们可以像访问属性一样去使用：

```java
//比如 String.getBytes() 方法，就是一个Getter，那么可以写成 bytes
Expression exp = parser.parseExpression("'Hello World'.bytes");
System.out.println(exp.getValue());
```

表达式可以不止一级，我们可以多级调用：

```java
Expression exp = parser.parseExpression("'Hello World'.bytes.length");   //继续访问数组的length属性
System.out.println(exp.getValue());
```

是不是感觉挺好玩的？我们继续来试试看构造方法，其实就是写Java代码，只是可以写成这种表达式而已：

```java
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
System.out.println(exp.getValue());
```

![image-20221125173157008](https://oss.itbaima.cn/internal/markdown/2022/11/25/PZmheYn5EVTvURN.png)

它甚至还支持根据特定表达式，从给定对象中获取属性出来：

```java
@Component
public class Student {
    private final String name;
    public Student(@Value("${test.name}") String name){
        this.name = name;
    }

    public String getName() {    //比如下面要访问name属性，那么这个属性得可以访问才行，访问权限不够是不行的
        return name;
    }
}
```

```java
Student student = context.getBean(Student.class);
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");
System.out.println(exp.getValue(student));    //直接读取对象的name属性
```

拿到对象属性之后，甚至还可以继续去处理：

```java
Expression exp = parser.parseExpression("name.bytes.length");   //拿到name之后继续getBytes然后length
```

除了获取，我们也可以调用表达式的setValue方法来设定属性的值：

```java
Expression exp = parser.parseExpression("name");
exp.setValue(student, "刻师傅");   //同样的，这个属性得有访问权限且能set才可以，否则会报错
```

除了属性调用，我们也可以使用运算符进行各种高级运算：

```java
Expression exp = parser.parseExpression("66 > 77");   //比较运算
System.out.println(exp.getValue());
```

```java
Expression exp = parser.parseExpression("99 + 99 * 3");   //算数运算
System.out.println(exp.getValue());
```

对于那些需要导入才能使用的类，我们需要使用一个特殊的语法：

```java
Expression exp = parser.parseExpression("T(java.lang.Math).random()");   //由T()囊括，包含完整包名+类名
//Expression exp = parser.parseExpression("T(System).nanoTime()");   //默认导入的类可以不加包名
System.out.println(exp.getValue());
```*