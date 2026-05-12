
# JavaScript

也称为js，是我们整个前端基础的重点内容，只有了解了JavaScript语言，我们才能了解前端如何与后端交互。

JavaScript与Java没有毛关系，仅仅只是名字中包含了Java而已，跟Java比起来，它更像Python，它是一门解释型语言，不需要进行编译，它甚至可以直接在浏览器的命令窗口中运行。

它相当于是前端静态页面的一个补充，它可以让一个普通的页面在后台执行一些程序，比如我们点击一个按钮，我们可能希望执行某些操作，比如下载文件、页面跳转、页面弹窗、进行登陆等，都可以使用JavaScript来帮助我们实现。

我们来看看一个简单的JavaScript程序：

```js
const arr = [0, 2, 1, 5, 9, 3, 4, 6, 7, 8]

for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - 1; j++) {
        if(arr[j] > arr[j+1]){
            const tmp = arr[j]
            arr[j] = arr[j+1]
            arr[j+1] = tmp
        }
    }
}

window.alert(arr)
```

这段代码实际上就是实现了一个冒泡排序算法，我们可以直接在页面的头部中引用此js文件，浏览器会在加载时自动执行js文件中编写的内容：

```html
<script src="test.js"></script>
```

我们发现JS的语法和Java非常相似，但是它还是和Java存在一些不同之处，而且存在很多阴间语法，那么我们来看看JS的语法。

# JavaScript基本语法

在js中，定义变量和Java中有一些不同，定义一个变量可以使用`let`关键字或是`var`关键字，IDEA推荐我们使用`let`关键字，因为`var`存在一定的设计缺陷（这里就不做讲解了，之后一律使用let关键字进行变量声明）：

```js
let a = 10;
a++;
window.alert(a)
```

上面的结果中，我们得到了a的结果是11，也就是说自增和自减运算在JS中也是支持的，并且JS每一句结尾可以不用加分号。

js并不是Java那样的强类型语言（任意变量的类型一定是明确的），它是一门弱类型语言，变量的类型并不会在一开始确定，因此我们在定义变量时无需指定变量的确切类型，而是在运行时动态解析类型：

```js
let a = 10;
a = "HelloWorld！"
console.info(a)
```

我们发现，变量a已经被赋值为数字类型，但是我们依然在后续能将其赋值一个字符串，它的类型是随时可变的。

很多人说，这种变态的类型机制是JS的一大缺陷。

世界上只有两种语言：一种是很多人骂的，一种是没人用的。

我们接着来看看，JS中存在的基本数据类型：

- Number：数字类型（包括小数和整数）
- String：字符串类型（可以使用单引号或是双引号）
- Boolean：布尔类型（与Java一致）

还包括一些特殊值：

- undefined：未定义 - 变量声明但不赋值默认为undefined
    
- null：空值 - 等同于Java中的null
    
- NaN：非数字 - 值不是合法数字，比如：

```js
window.alert(100/'xx')  
```


我们可以使用`typeof`关键字来查看当前变量值的类型：

```js
let a = 10;
console.info(typeof a)
a = 'Hello World'
console.info(typeof a)
```