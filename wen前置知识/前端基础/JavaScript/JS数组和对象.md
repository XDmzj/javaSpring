
# 数组

JS中的数组定义与Java不同，它更像是Python中的列表，数组中的每个元素并不需要时同样的类型：

js复制代码

```js
let arr = [1, "lbwnb", false, undefined, NaN]
```

我们可以直接使用下标来访问：

js复制代码

```js
let arr = [1, "lbwnb", false, undefined, NaN]
console.info(arr[1])
```

我们一开始编写的排序算法，也是使用了数组。

数组还可以动态扩容，如果我们尝试访问超出数组长度的元素，并不会出现错误，而是得到undefined，同样的，我们也可以直接往超出数组长度的地方设置元素：

js复制代码

```js
let arr = [1, "lbwnb", false, undefined, NaN]
arr[5] = "???"
console.info(arr)
```

也可以使用`push`和`pop`来实现栈操作：

js复制代码

```js
let arr = [1, "lbwnb", false, undefined, NaN]
arr.push("bbb")
console.info(arr.pop())
console.info(arr)
```

数组还包括一些其他的方法，这里就不一一列出了：

js复制代码

```js
let arr = [1, "lbwnb", false, undefined, NaN]
arr.fill(1)
console.info(arr.map(o => {
    return 'xxx'+o
}))
```

我们接着来看对象，JS中也能定义对象，但是这里的对象有点颠覆我们的认知：

js复制代码

```js
let obj = new Object()
let obj = {}
```

以上两种写法都能够创建一个对象，但是更推荐使用下面的一种。


# 对象

我们接着来看对象，JS中也能定义对象，但是这里的对象有点颠覆我们的认知：

```js
let obj = new Object()
let obj = {}
```

以上两种写法都能够创建一个对象，但是更推荐使用下面的一种。


## 动态添加属性
JS中的对象也是非常随意的，我们可以动态为其添加属性：

```js
let obj = {}
obj.name = "伞兵一号"
console.info(obj)
```

## 动态添加函数
同理，我们也可以给对象动态添加一个函数：

```js
let obj = {}
obj.f = function (){
    console.info("我是对象内部的函数")
}

obj.f()
```

## this

我们可以在函数内使用this关键字来指定对象内的属性：

```js
let name = "我是外部变量"
let obj = {}
obj.name = "我是内部变量"
obj.f = function (){
    console.info("name属性为："+this.name)
}

obj.f()
```

**注意：** 如果使用lambda表达式，那么this并不会指向对象。


## 创建时指定对象属性 

除了动态添加属性，我们也可以在一开始的时候指定对象内部的成员：

```js
let obj = {
    name: "我是内部的变量",
  	f: function (){
        console.info("name属性为："+this.name)
    }
}

obj.f()
```

注意如果有多行属性，需要在属性定义后添加一个`,`进行分割！