

# 关系运算

我们接着来看看js中的关系运算符，包括如下8个关系运算符：大于（>）,小于（<）,小于等于（<=）,大于等于（>=）,`相等（==），不等（!=），全等（===），不全等（!==）`

其实关系运算符大致和Java中的使用方法一致，不过它还可以进行字符串比较，有点像C++的语法：

```js
console.info(666 > 777)
console.info('aa' > 'ab')
```

那么，相等和全等有什么区别呢？

```java
console.info('10' == 10)
console.info('10' === 10)
```

我们发现，在Java中，若运算符两边是不同的基本数据类型，会直接得到false，而JS中却不像这样，我们发现字符串的10居然等于数字10，而使用全等判断才是我们希望的结果。

`==`的比较规则是：当操作数类型一样时，比较的规则和恒等运算符一样，都相等才相等，如果两个操作数是字符串，则进行字符串的比较，如果里面有一个操作数不是字符串，那两个操作数通过Number()方法进行转换，转成数字进行比较。

因此，我们上面进行的判断实际上是运算符两边都进行了数字转换的结果进行比较，自然也就得到了true，而全等判断才是我们在Java中认识的相等判断。


# 逻辑运算
我们接着来看逻辑运算，JS中包括&&、||、&、|、?:等，我们先来看看位运算符：

```js
console.info(4 & 7)
console.info(4 | 7)
```

实际上和Java中是一样的，那么我再来看看逻辑运算：

```js
console.info(true || false)
```

对于boolean变量的判断，是与Java一致的，但是JS也可以使用非Boolen类型变量进行判断：

```js
console.info(!0)
console.info(!1)
```

和C/C++语言一样，0代表false，非0代表true，那么字符串呢？

```js
console.info(!"a")
console.info(!"")
```

我们发现，空串为false，非空串为true，我们再来看看：

```js
console.info(true || 7)
console.info(7 || true)
```

我们发现，前者得到的结果为true，而后者得到的结果却是是7，真是滑天下之大稽，什么鬼玩意，实际上是因为，默认非0都是true，而后者又是先判断的7，因此会直接得到7而不是被转换为true

那么我们再来看看几个特殊值默认代表什么：

```js
console.info(!undefined)
console.info(!null)
console.info(!NaN)
```
结果：
![[Pasted image 20260428092506.png]]
# 三元运算符

最后来使用一下三元运算符，实际上和Java中是一样的：

```js
let a = true ? "xx" : 20
console.info(a)
```

得益于JS的动态类型，三元运算符不一定需要固定的返回值类型。


# 分支与循环

JS的分支结构，实际上和Java是一样的，也是使用if-else语句来进行：

```js
if("lbwnb"){   //非空串为true
    console.info("!!!")
} else {
    console.info("???")
}
```

同理，多分支语句也能实现：

```js
if(""){
    console.info("!!!")
} else if(-666){
    console.info("???")
} else {
    console.info("O.O")
}
```

当然，多分支语句也可以使用switch来完成：

```js
let a = "a"
switch (a){
    case "a":
        console.info("1")
        break
    case "b":
        console.info("2")
        break
    case "c":
        console.info("3")
        break
    default:
        console.info("4")
}
```

接着我们来看看循环结构，其实循环结构也和Java相差不大：

```js
let i = 10
while(i--){
    console.info("100")
}
```

```js
for (let i = 0; i < 10; i++) {
    console.info("??")
}
```