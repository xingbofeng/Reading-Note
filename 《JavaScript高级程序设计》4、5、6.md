# 第四章：变量、作用域和内存问题

按照`ECMA-262`的定义，`JavaScript`松散类型的本质，决定了它只是在特定时间用于保存特定的值的一个名字而已。由于不存在定义某个变量必须保存何种数据类型值的规则，变量的值，其数据类型可以在脚本的声明周期内改变。

## 基本类型和引用类型的值

* `JavaScript`中包括两种不同数据类型的值：
  * 基本类型值 (`undefined` `null` `string` `number` `boolean`)
  * 引用类型值 (`对象`)

### 动态的属性

* 我们可以给引用类型添加属性。

```javascript
var person = new Object();
person.name = "Nicholas";
console.log(person.name); // "Nicholas"
```

* 但是不能给基本类型添加属性。

```javascript
var name = "Nicholas";
name.age = 27;
console.log(name.age); // "Nicholas"
```

### 复制变量值

* 基本类型复制变量会在新的内存空间上创建新值，而当从一个变量向另一个变量复制引用类型的值时，同样也会将存储在变量对象中的值复制一份放到为新变量分配的空间中。不同的是，这个值的副本实际上是一个指针，而这个指针指向存储在堆中的一个对象。

### 传递参数


* `ECMAScript`中所有函数的参数都是**按值传递**的，也就是说，把函数外部的值复制给函数内部的参数，就和把值从一个变量复制到另一个变量一样。

```javascript
function addTen(num) {
  num += 10;
  return num;
}
var count = 20;
var result = addTen(count);
console.log(count); // 20，没有变化
console.log(result); // 30
```

在函数内部，参数`num`的值被加上了`10`，但这一变化不会影响函数外部的`count`变量。

但是对于引用类型来说：

```javascript
function setName(obj) {
  obj.name = "Nicholas";
}
var person = new Object();
setName(person);
console.log(person.name); // "Nicholas"
```

在这个函数内部，`obj`和`person`引用的是同一个对象。换句话说，即使这个变量是按值传递的，`obj` 也会按引用来访问同一个对象。于是，当在函数内部为`obj`添加`name`属性后，函数外部的`person`也将有所反映。

为了证明对象是按值传递的，我们再看一看下面这个经过修改的例子：

```javascript
function setName(obj) {
  obj.name = "Nicholas";
  obj = new Object();
  obj.name = "Greg";
}
var person = new Object();
setName(person);
console.log(person.name); // "Nicholas"
```

如果`person`是按引用传递的，那么`person`就会自动被修改为指向其`name`属性值为`"Greg"`的新对象。但是，当接下来再访问`person.name`时，显示的值仍然是`"Nicholas"`。这说明即使在函数内部修改了参数的值，但原始的引用仍然保持未变。

实际上，当在函数内部重写`obj`时，这个变量引用的就是一个局部对象了。而这个局部对象会在函数执行完毕后立即被销毁。

### 检测类型

* 使用`typeof`
* 使用`instanceof`
* 使用`Object.prototype.toString.call`

## 执行环境及作用域
### 延长作用域
* 使用`try... catch`或`with`延长作用域

使用`with`的例子：

```javascript
function buildUrl() {
  var qs = "?debug=true";
  with(location) {
    var url = href + qs;
  }
  return url;
}
```

在此，`with`语句接收的是`location`对象，因此其变量对象中就包含了`location`对象的所有属性和方法，而这个变量对象被添加到了作用域链的前端。当在`with`语句中引用变量`href`时（实际引用的是`location.href`），可以在当前执行环境的变量对象中找到。当引用变量`qs`时，引用的则是在`buildUrl()`中定义的那个变量，而该变量位于函数环境的变量对象中。至于`with`语句内部，则定义了一个名为`url`的变量，因而`url`就成了函数执行环境的一部分，所以可以作为函数的值被返回。

### 没有块级作用域
* `JavaScript`没有块级作用域，只有全局作用域和函数作用域，`ES6`可以通过`let`、`const`声明创建词法作用域。

## 垃圾收集
`JavaScript`的`GC`有两种方式：

* 标记清除：一旦变量不处于执行环境中，则`GC`会给它带上一个标记，再下一次`GC`生命周期时，清除带标记的变量所占内存。
* 引用计数：有一个弊端，相互引用的情况。

### 性能问题
### 管理内存

* 将不用的变量手动设置为`null`，解除引用关系来手动释放内存。

# 第五章：引用类型
## Object类型

* 访问对象可以使用点表示法和方括号表示法。但是如果属性名中包含会导致语法错误的字符，或者属性使用的是关键字或保留字，也可以使用方括号表示法(这个时候使用点表示法就报错了)。

```javascript
person["first name"] = "Nicholas";
```

> 上面的属性名含有一个空格，因此只能用方括号表示法。

* 除了必须使用方括号表示法的情况，不然建议使用点表示法。

## Array类型

* 特别注意数组最多可以包含`4294967294`个项，如果添加的项数超过这个数，可能会导致错误

### 检测数组

* 对于一个网页或者一个全局作用域而言，可使用`instanceof`

* `Object.prototype.toString`方法

``` javascript
var isArray = function (obj) {
  return obj == null ? false : Object.prototype.toString.call(obj) === '[object Array]';
}

```
* `Array.isArray()`方法

### 转换方法

```javascript
var colors = ["red", "blue", "green"];
console.log(colors.valueOf()); // 返回数组["red", "blue", "green"]
console.log(colors.toString()); // 返回字符串red,blue,green
console.log(colors.toLocaleString()); // 返回字符串red,blue,green
```

### 栈方法
* [push()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push) 将一个或多个元素添加到数组的末尾，并返回新数组的长度。
* [pop()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/pop)从数组中删除最后一个元素，并返回该元素的值。此方法更改数组的长度。

### 队列方法
* [shift()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/shift)从数组中删除第一个元素，并返回该元素的值。此方法更改数组的长度。
* [unshift()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift)将一个或多个元素添加到数组的开头，并返回新数组的长度。

### 重排序方法
* [reverse()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse)将数组中元素的位置颠倒。
* [sort()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)在适当的位置对数组的元素进行排序，并返回数组。sort排序不一定是稳定的。默认排序顺序是根据字符串`Unicode`码点。`sort()方法`可以传参，用来指定按某种顺序进行排列的函数。如果省略，元素按照转换为的字符串的诸个字符的`Unicode`位点进行排序。

### 操作方法
* [concat()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)方法用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。
* [slice()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)返回一个从开始到结束（不包括结束）选择的数组的一部分浅拷贝到一个新数组对象。原始数组不会被修改。
* [splice()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)通过删除现有元素和/或添加新元素来更改一个数组的内容。它始终会返回一个数组，该数组中包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）。

### 位置方法
* [indexOf()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回`-1`。
* [lastIndexOf()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/lastIndexOf)返回指定元素（也即有效的`JavaScript`值或变量）在数组中的最后一个的索引，如果不存在则返回`-1`。从数组的后面向前查找，从`fromIndex`处开始。

### 迭代方法
* [every()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。
* [filter()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)对数组中的每一项运行给定函数，返回该函数会返回true 的项组成的数组。
* [forEach()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)对数组中的每一项运行给定函数。这个方法没有返回值。
* [map()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
* [some()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true。

### 归并方法
* [reduce()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。
* [reduceRight()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight)接受一个函数作为累加器（accumulator）和数组的每个值（从右到左）将其减少为单个值。

## Date类型
> 一堆方法，查APIㄟ( ▔, ▔ )ㄏ

## RegExp类型
* 可以阅读[《JavaScript 正则表达式迷你书》](https://zhuanlan.zhihu.com/p/29707385)

## Function类型
* 函数名其实仅仅是指向函数的指针，因此函数名与包含对象指针的其它变量没有什么不同，也就是说函数可能会有多个名字

### 没有重载
* `JavaScript`的函数没有重载，将函数想象为指针，有助于理解为什么函数没有重载的概念。

### 函数声明与函数表达式
* 解析器在向执行环境中加载数据时，对函数声明和函数表达式并不是一视同仁，解析器会率先读取函数声明（**存在函数声明提升**），并使其在执行任何代码之前可用，而函数表达式，则是必须等到解析器执行到它所在的代码行，才会真正地被执行。

``` javascript
console.log(sum(10, 10)) // 20

function sum (num1, num2) {
  return num1 + num2;
}

console.log(fn()) // 报错

var fn = function () {
  return 1;
}
```

### 作为值的函数
* 因为`ECMAScript`中的函数名本身就是变量，所以函数也可以作为值来使用。所以可以作为参数传递，可以返回一个函数。

### 函数内部属性
* 在函数内部，有两个特殊的对象：`arguments`和`this`。
* 重点是`arguments.callee`和`arguments.callee.caller`。
* `arguments`是一个伪数组对象，主要用途是保存函数参数，但这个对象还有一个名叫`callee` 的属性，该属性是一个指针，指向拥有这个`arguments`对象的函数。

```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * factorial(num - 1)
  }
}
```

上述的阶乘函数没有任何问题，但这个函数的执行与函数名`factorial`紧密耦合在一起，为了消除这种紧密耦合的现象，可以像下面这样使用`arguments.callee`。

```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * arguments.callee(num - 1)
  }
}
```

* `ECMAScript 5`也规范化了另一个函数对象的属性：`caller`。这个属性中保存着调用当前函数的函数的引用，如果是在全局作用域中调用当前函数，它的值为`null`。

```javascript
function outer() {
  inner()
}

function inner() {
  console.log(inner.caller)
}
outer(); // function outer(){inner()}
```

> 为了实现更松散的耦合，也可以通过arguments.callee.caller来访问相同的信息。

* 当函数在严格模式下运行时，访问`arguments.callee.caller`或`arguments.callee`会导致错误。
* 之前在听的`pigo`的代码解读，严格模式下可以使用`try ... catch`的错误栈获得当前函数调用者的函数名，即可获取当前引用。

### 函数属性和方法
* 每个函数都包含两个属性：`length`和`prototype`。其中，`length`属性表示函数希望接收的命名参数的个数。

```javascript
function sayName(name) {
  console.log(name);
}

function sum(num1, num2) {
  return num1 + num2;
}

function sayHi() {
  console.log('hi');
}

console.log(sayName.length); // 1
console.log(sum.length); // 2
console.log(sayHi.length); // 0
```

* `call`、`apply`、`bind`方法

> 之前疏漏的点：bind和call、apply类似，也可以预传参数，实现偏函数

摘自[MDN Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)：`bind()`的另一个最简单的用法是使一个函数拥有预设的初始参数。这些参数（如果有的话）作为`bind()`的第二个参数跟在`this`（或其他对象）后面，之后它们会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们的后面。

```javascript
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Create a function with a preset leading argument
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

## 基本包装类型
* 引用类型与基本包装类型的主要区别就是对象的生存期。使用`new`操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中。而自动创建的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。这意味着我们不能在运行时为基本类型值添加属性和方法，这样做是没有意义的：

```javascript
var s1 = "some text";
s1.color = "red";
console.log(s1.color); // 打印undefined，因为在第二行创建的s1的基本包装类型已经被销毁了
```

* 注意转换函数和使用`new`创建基本包装类型使用`typeof`的结果不一样。

```javascript
var falseObject = new Boolean(false);
var falseValue = false;

console.log(typeof falseObject); // "object"
console.log(typeof falseValue); // "boolean"
console.log(falseObject instanceof Boolean); // true
console.log(falseValue instanceof Boolean); // false
```

### Boolean类型
> 注意new Boolean(false)实际上会被转化为true

```javascript
var falseObject = new Boolean(false);
var result = falseObject && true;
console.log(result); // true

var falseValue = false;
result = falseValue && true;
console.log(result); // false
```

在这个例子中，我们使用`false`值创建了一个`Boolean`对象。然后，将这个对象与基本类型值`true`构成了逻辑与表达式。在布尔运算中，`false && true`等于`false`。可是，示例中的这行代码是对`falseObject`而不是对它的值（`false`）进行求值。前面讨论过，布尔表达式中的所有对象都会被转换为`true`，因此`falseObject` 对象在布尔表达式中代表的是`true`。结果，`true && true`当然就等于`true`了。

### Number类型
* [toFixed()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed)使用定点表示法来格式化一个数。
* [toExponential()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/toExponential)以指数表示法返回该数值字符串表示形式。
* [toPrecision()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/toPrecision)以指定的精度返回该数值对象的字符串表示。（科学表示法）

### String类型
* [charAt()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charAt)从一个字符串中返回指定的字符。
* [charCodeAt()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)返回`0`到`65535`之间的整数，表示给定索引处的`UTF-16`代码单元 (在`Unicode`编码单元表示一个单一的`UTF-16`编码单元的情况下，`UTF-16`编码单元匹配`Unicode`编码单元。但在——例如`Unicode`编码单元 `> 0x10000` 的这种——不能被一个`UTF-16`编码单元单独表示的情况下，只能匹配`Unicode`代理对的第一个编码单元) 。如果你想要整个代码点的值，使用`codePointAt()`。
* [slice()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/slice)提取一个字符串的一部分，并返回一新的字符串。
* [substring()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/substring)返回一个字符串在开始索引到结束索引之间的一个子集, 或从开始索引直到字符串的末尾的一个子集。
* [substr()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/substr)返回一个字符串中从指定位置开始到指定字符数的字符。


> slice()和substring()的第二个参数指定的是子字符串最后一个字符后面的位置。而substr()的第二个参数指定的则是返回的字符个数。在传递给这些方法的参数是负值的情况下，slice()方法会将传入的负值与字符串的长度相加。substr()方法将负的第一个参数加上字符串的长度，而将负的第二个参数转换为0。substring()方法会把所有负值参数都转换为0。

```javascript
var stringValue = "hello world";
console.log(stringValue.slice(-3)); // "rld"，slice()方法接受负值
console.log(stringValue.substring(-3)); // "hello world"，substring()方法第一个参数会把负值转化成0
console.log(stringValue.substr(-3)); // "rld"，substr()方法第一个参数接受负值
console.log(stringValue.slice(3, -4)); // "lo w"，slice()方法接受负值
console.log(stringValue.substring(3, -4)); // "hel"，substring()方法第二个参数接受负值
console.log(stringValue.substr(3, -4)); // ""（空字符串），substr()方法第二个参数会把负值转化成0
```

* [indexOf()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf)返回调用`String`对象中第一次出现的指定值的索引，开始在 fromIndex进行搜索。
* [lastIndexOf()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/lastIndexOf)返回指定值在调用该方法的字符串中最后出现的位置，如果没找到则返回`-1`。从该字符串的后面向前查找，从`fromIndex`处开始。

> indexOf()方法和lastIndexOf()方法都可以接收可选的第二个参数，表示从字符串中的哪个位置开始搜索。

* [trim()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/Trim)会从一个字符串的两端删除空白字符。在这个上下文中的空白字符是所有的空白字符 (`space`, `tab`, `no-break space`等) 以及所有行终止符字符（如`LF`，`CR`）。
* `toUpperCase()`、`toLocaleUpperCase()`、`toLowerCase()`、`toLocaleLowerCase()`都是转换字符串大小写的方法，对于是否带`Locale`，一般说来，`toUpperCase()`、`toLocaleUpperCase()`返回一样的结果，但对于特殊地区的语言（如土耳其语），会有特殊的转换规则。

* `HTML方法`，即将废弃。

## 单体内置对象
### Global对象
* `Global`对象的`encodeURI()`和`encodeURIComponent()`方法可以对`URI`进行编码，以便发送给浏览器。
* `encodeURI()`不会对本身属于`URI`的特殊字符进行编码，例如冒号、正斜杠、问号和井字号。
* `encodeURIComponent()`则会对它发现的任何非标准字符进行编码。

```javascript
var uri = "http://www.wrox.com/illegal value.htm#start";
console.log(encodeURI(uri)); // "http://www.wrox.com/illegal%20value.htm#start"
console.log(encodeURIComponent(uri)); // "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start"
```

> 一般来说， 我们使用encodeURIComponent() 方法的时候要比使用encodeURI()更多，因为在实践中更常见的是对查询字符串参数而不是对基础URI进行编码。

* 与`encodeURI()`和`encodeURIComponent()`方法对应的两个方法分别是`decodeURI()`和`decodeURIComponent()`。
* `decodeURI()`只能对使用`encodeURI()`替换的字符进行解码。例如，它可将`%20`替换成一个空格，但不会对`%23`作任何处理。
* `decodeURIComponent()`能够解码使用`encodeURIComponent()`编码的所有字符。

```javascript
var uri = "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start";
console.log(decodeURI(uri)); // http%3A%2F%2Fwww.wrox.com%2Fillegal value.htm%23start
console.log(decodeURIComponent(uri)); // http://www.wrox.com/illegal value.htm#start
```

* `eval()方法`，不推荐使用。
* `ECMAScript`明确禁止给`undefined`、`NaN`和`Infinity`赋值，这样做即使在非严格模式下也会导致错误。
* 取得`Global`对象的方法。

```javascript
var global = function() {
  return this;
}();
```

### Math对象
> 查API就行了。

常用的一个公式，生成指定返回的随机数。

```javascript
var value = Math.floor(Math.random() * (higher - lower + 1) + lower);
```

> 上述式子，生成lower ~ higher的随机数，包括lower和higher

# 第六章：面向对象的程序设计
## 理解对象
两种方式创建对象：

* 构造函数创建
* 对象字面量创建

### 属性类型
> 对象具有两种属性：数据属性和访问器属性。

#### 数据属性
* 数据属性包含一个数据值的位置，在这个位置可以读取和写入值，数据属性有4个描述其行为的特性。
  * [[Configurable]] : 表示能否通过delete删除属性重而重新定义属性，能否修改属性的特性，能否把属性修改为访问器属性,使用`new Object`或对象字面量创建的对象，默认值为`true`
  * [[Enumerable]] : 表述能否通过`for in`循环返回属性，使用`new Object`或对象字面量，默认值为`true`
  * [[Writable]] : 表述能否修改属性的值，使用`new Object`或对象字面量，默认值为`true`
  * [[Value]] : 包含这个属性的数据值，读取属性值的时候，从这个位置读，写入属性值的时候，把新值保存到这个位置。这个特性的默认值为`undefined`。

要修改属性默认的特性，必须使用`ECMAScript`的`Object.defineProperty()`方法。这个方法接收三个参数：属性所在的对象、属性的名字和一个描述符对象。其中，描述符（`descriptor`）对象的属性必须是：`configurable`、`enumerable`、`writable`和`value`。

```javascript
var person = {};
Object.defineProperty(person, "name", {
  writable: false,
  value: "Nicholas"
});
console.log(person.name); // "Nicholas"
person.name = "Greg";
console.log(person.name); // "Nicholas"
```

> 在非严格模式下，赋值操作 person.name = "Greg" 将被忽略；在严格模式下，赋值操作将会导致抛出错误。

可以多次调用`Object.defineProperty()`方法修改同一属性，但在把`configurable`特性设置为`false`之后就会有限制了。

#### 访问器属性

* 访问器属性不包括数据值，包含一堆`getter`和`setter`函数（这两个函数不是必须的）。在读取属性的时，会调用`getter`函数，这个函数负责返回有效值，在写入访问器属性时，会调用`getter`函数并传入新值，这个函数负责决定如何处理数据。分别有以下属性
  * [[Configurable]] : 表示能否通过delete删除属性重而重新定义属性，能否修改属性的特性，能否把属性修改为访问器属性,使用`new Object`或对象字面量，默认值为`true`
  * [[Enumerable]] : 表述能否通过`for in`循环返回属性，使用`new Object`或对象字面量，默认值为`true`
  * [[Get]] : 在读取属性时调用的函数，默认值为`undefined`
  * [[Set]] : 在写入属性时调用的函数，默认值为`undefined`

访问器属性不能直接定义，必须使用`Object.defineProperty()`方法定义。

> 不一定非要同时指定getter 和setter。只指定getter 意味着属性是不能写，尝试写入属性会被忽略。 在严格模式下，尝试写入只指定了getter 函数的属性会抛出错误。类似地，只指定setter 函数的属性也不能读，否则在非严格模式下会返回undefined，而在严格模式下会抛出错误。


以前要创建访问器属性， 一般都使用两个非标准的方法：defineGetter()和defineSetter()

``` javascript
var book = {
  _year: 2004,
  edition: 1
};
// 定义访问器的旧有方法
book.__defineGetter__("year", function() {
  return this._year;
});
book.__defineSetter__("year", function(newValue) {
  if (newValue > 2004) {
    this._year = newValue;
    this.edition += newValue - 2004;
  }
});
book.year = 2005;
console.log(book.edition); // 2
```

### 定义多个属性
> ECMAScript 5 又定义了一个Object.defineProperties()方法。利用这个方法可以通过描述符一次定义多个属性。这个方法接收两个对象参数：第一个对象是要添加和修改其属性的对象，第二个对象的属性与第一个对象中要添加或修改的属性一一对应。例如：

``` javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
    get: function() {
      return this._year;
    },
    set: function(newValue) {
      if (newValue > 2004) {
        this._year = newValue;
        this.edition += newValue - 2004;
      }
    }
  }
});
```

### 读取属性的特性
> 使用ECMAScript 5 的Object.getOwnPropertyDescriptor()方法，可以取得给定属性的描述符。这个方法接收两个参数：属性所在的对象和要读取其描述符的属性名称，返回值是一个对象。如果是访问器属性，这个对象的属性有configurable、enumerable、get 和set；如果是数据属性，这个对象的属性有configurable、enumerable、writable 和value。

```javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
    get: function() {
      return this._year;
    },
    set: function(newValue) {
      if (newValue > 2004) {
        this._year = newValue;
        this.edition += newValue - 2004;
      }
    }
  }
});
var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
console.log(descriptor.value); // 2004
console.log(descriptor.configurable); // false
console.log(typeof descriptor.get); // "undefined"
var descriptor = Object.getOwnPropertyDescriptor(book, "year");
console.log(descriptor.value); // undefined
console.log(descriptor.enumerable); // false
console.log(typeof descriptor.get); // "function"
```

## 创建对象

> 使用Object构造函数和对象字面量的形式来创建对象有一个明显的缺点，使用同一个接口创建很多对象，会产生很多垃圾代码。

## 创建对象的各种模式


### 工厂模式

> 工厂模式是软件工程领域一种广为人知的设计模式，这种模式抽象了创建具体对象的过程，用函数来封装以特定接口来创建对象的细节。

``` javascript

function createPerson(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function() {
    console.log(this.name);
  };
  return o;
}
var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");

```

`createPerson`函数能够根据传入的参数来构建包含三个所有必要信息的`Person`对象,可以无数次的调用这个函数，而它每次都会返回包含三个属性的一个方法的对象。工厂模式虽然解决了创建多个相似对象的代码冗余问题，但是却没有解决`对象识别`的问题（如何知道一个对象的类型）。

### 构造函数模式

> 构造函数可以用来创建特定类型的对象，有些类似Array和Object在运行时就自动出现在执行环境中，此外，也可以自定义构造函数。从而自定义对象类型的属性和方法。


``` javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function() {
    console.log(this.name);
  };
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

```

> 注意，按照惯例，构造函数一般要以大写字母开头。

要创建`Person`的新实例，必须使用`new`操作符。以这种方式调用构造函数实际上会经历以下4个步骤：

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此this 就指向了这个新对象）
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回新对象

* 构造函数当做普通函数调用时，会把相关属性赋予当前作用域的`this`指向的对象上，在全局作用域下，把属性添加到`window`。
* 上述构造函数有一个问题，`sayName()`方法创建了两次。

```javascript
console.log(person1.sayName == person2.sayName); // false
```

创建两个完成同样任务的`Function`实例的确没有必要；况且有`this`对象在，根本不用在执行代码前就把函数绑定到特定对象上面。因此，大可像下面这样，通过把函数定义转移到构造函数外部来解决这个问题。

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}

function sayName() {
  console.log(this.name);
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

* 可是新问题又来了：在全局作用域中定义的函数实际上只能被某个对象调用，这让全局作用域有点名不副实。而更让人无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了。好在，这些问题可以通过使用**原型模式**来解决。

### 原型模式
我们创建的每个函数都有一个`prototype`（原型）属性，这个属性是一个指针，指向一个**对象**，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。

```javascript
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};
var person1 = new Person();
person1.sayName(); //"Nicholas"
var person2 = new Person();
person2.sayName(); //"Nicholas"
console.log(person1.sayName == person2.sayName); //true
```

与构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的。换句话说，`person1`和`person2`访问的都是同一组属性和同一个`sayName()`函数。

#### 理解原型对象

只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个`prototype`属性，这个属性指向函数的原型对象。

在默认情况下，所有原型对象都会自动获得一个`constructor`（构造函数）属性，这个属性包含一个指向`prototype`属性所在函数的指针。

创建了自定义的构造函数之后，其原型对象默认只会取得`constructor`属性；至于其他方法，则都是从`Object`继承而来的。

当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。

以前面使用`Person` 构造函数和`Person.prototype` 创建实例的代码为例，图6-1 展示了各个对象之间的关系。

![](https://sinacloud.net/pro-js/6-1.png)

可以通过`isPrototypeOf()`方法来确定对象之间是否存在原型关系。

```javascript
console.log(Person.prototype.isPrototypeOf(person1)); // true
console.log(Person.prototype.isPrototypeOf(person2)); // true
```

`ECMAScript 5`增加了一个新方法，叫`Object.getPrototypeOf()`，在所有支持的实现中，这个方法返回`[[Prototype]]`的值：

```javascript
console.log(Object.getPrototypeOf(person1) == Person.prototype); // true
console.log(Object.getPrototypeOf(person1).name); // "Nicholas"
```

> 支持这个方法的浏览器有IE9+、Firefox 3.5+、Safari 5+、Opera 12+和Chrome。

每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。搜索首先从对象实例本身开始。如果在实例中找到了具有给定名字的属性，则返回该属性的值；如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性。如果在原型对象中找到了这个属性，则返回该属性的值。

当为对象实例添加一个属性时，这个属性就会**屏蔽**原型对象中保存的同名属性；换句话说，添加这个属性只会阻止我们访问原型中的那个属性，但不会修改那个属性。即使将这个属性设置为`null`，也只会在实例中设置这个属性，而不会恢复其指向原型的连接。不过，使用`delete` 操作符则可以完全删除实例属性，从而让我们能够重新访问原型中的属性，如下所示。

```javascript
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};
var person1 = new Person();
var person2 = new Person();
person1.name = "Greg";
console.log(person1.name); // "Greg"——来自实例
console.log(person2.name); // "Nicholas"——来自原型

delete person1.name;
console.log(person1.name); // "Nicholas"——来自原型
```

使用`hasOwnProperty()`方法可以检测一个属性是存在于实例中，还是存在于原型中。这个方法（不
要忘了它是从`Object`继承来的）只在给定属性存在于对象实例中时，才会返回`true`：

```javascript
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};
var person1 = new Person();
var person2 = new Person();
console.log(person1.hasOwnProperty("name")); // false
person1.name = "Greg";
console.log(person1.name); // "Greg"——来自实例
console.log(person1.hasOwnProperty("name")); // true
console.log(person2.name); // "Nicholas"——来自原型
console.log(person2.hasOwnProperty("name")); // false
delete person1.name;
console.log(person1.name); // "Nicholas"——来自原型
console.log(person1.hasOwnProperty("name")); // false
```

#### 原型与in操作符

有两种方式使用`in`操作符：单独使用和在`for-in`循环中使用。在单独使用时，`in`操作符会在通过对象能够访问给定属性时返回`true`，无论该属性存在于实例中还是原型中。看一看下面的例子。

```javascript
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};
var person1 = new Person();
var person2 = new Person();
console.log(person1.hasOwnProperty("name")); // false
console.log("name" in person1); // true
person1.name = "Greg";
console.log(person1.name); // "Greg" ——来自实例
console.log(person1.hasOwnProperty("name")); // true
console.log("name" in person1); // true
console.log(person2.name); // "Nicholas" ——来自原型
console.log(person2.hasOwnProperty("name")); // false
console.log("name" in person2); // true
delete person1.name;
console.log(person1.name); // "Nicholas" ——来自原型
console.log(person1.hasOwnProperty("name")); // false
console.log("name" in person1); // true
```

同时使用`hasOwnProperty()`方法和`in`操作符，就可以确定该属性到底是存在于对象中，还是存在于原型中，如下所示。

```javascript
function hasPrototypeProperty(object, name){
  return !object.hasOwnProperty(name) && (name in object);
}
```

在使用`for-in`循环时，返回的是所有能够通过对象访问的、可枚举的（`enumerated`）属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。

屏蔽了原型中不可枚举属性（即将`[[Enumerable]]`标记为`false`的属性）的实例属性也会在`for-in`循环中返回。

要取得对象上所有可枚举的实例属性，可以使用`ECMAScript 5`的`Object.keys()`方法。这个方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。

```javascript
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};
var keys = Object.keys(Person.prototype);
console.log(keys); // "name,age,job,sayName"
var p1 = new Person();
p1.name = "Rob";
p1.age = 31;
var p1keys = Object.keys(p1);
console.log(p1keys); // "name,age"
```

如果你想要得到所有实例属性，无论它是否可枚举，都可以使用`Object.getOwnPropertyNames()`方法。

```javascript
var keys = Object.getOwnPropertyNames(Person.prototype);
console.log(keys); // "constructor,name,age,job,sayName"
```

#### 更简单的原型语法
> 会使得constructor指向不再指向默认的构造函数，解决办法是 Object.defineProperty()方法来定义不可枚举的构造函数。

为减少不必要的输入，也为了从视觉上更好地封装原型的功能，更常见的做法是用一个包含所有属性和方法的对象字面量来重写整个原型对象，如下面的例子所示。

```javascript
function Person() {}
Person.prototype = {
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function() {
    console.log(this.name);
  }
};
```

注意，我们在这里使用的语法，本质上完全重写了默认的`prototype`对象，`constructor`属性不再指向`Person`了：

```javascript
var friend = new Person();
console.log(friend instanceof Object); // true
console.log(friend instanceof Person); // true
console.log(friend.constructor == Person); // false
console.log(friend.constructor == Object); // true
```

如果`constructor`的值真的很重要，可以像下面这样特意将它设置回适当的值。

```javascript
function Person() {}
Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function() {
    console.log(this.name);
  }
};
```

>  以这种方式重设constructor 属性会导致它的[[Enumerable]]特性被设置为true。应该使用Object.defineProperty()方法定义。

#### 原型的动态性

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来——即使是先创建了实例后修改原型也照样如此。

```javascript
var friend = new Person();
Person.prototype.sayHi = function() {
  console.log("hi");
};
friend.sayHi(); // "hi"（没有问题！）
```

但如果是重写整个原型对象，那么情况就不一样了。我们知道，调用构造函数时会为实例添加一个指向最初原型的`[[Prototype]]`指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。

```javascript
function Person() {}
var friend = new Person();
Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function() {
    console.log(this.name);
  }
};
friend.sayName(); // error
```

下图展示了这个过程的内幕。

![](https://sinacloud.net/pro-js/6-3.jpg)

#### 原生对象的原型

所有原生引用类型（`Object`、`Array`、`String`，等等）都在其构造函数的原型上定义了方法。
例如，在`Array.prototype`中可以找到`sort()`方法，而在`String.prototype` 中可以找到`substring()`方法，如下所示。

```javascript
console.log(typeof Array.prototype.sort); // "function"
console.log(typeof String.prototype.substring); // "function"
```

通过原生对象的原型，不仅可以取得所有默认方法的引用，而且也可以定义新方法。

```javascript
String.prototype.startsWith = function(text) {
  return this.indexOf(text) == 0;
};
var msg = "Hello world!";
console.log(msg.startsWith("Hello")); // true
```

#### 原型对象的问题

原型模式的最大问题是由其**共享的本性**所导致的。

原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。对于那些包含基本值的属性倒也说得过去，毕竟（如前面的例子所示），通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。然而，对于包含引用类型值的属性来说，问题就比较突出了。来看下面的例子。

> 因此，不应该在原型对象上定义引用类型，而应该定义在实例属性上。即组合使用构造函数模式和原型模式。

```javascript
function Person() {}
Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  friends: ["Shelby", "Court"],
  sayName: function() {
    console.log(this.name);
  }
};
var person1 = new Person();
var person2 = new Person();
person1.friends.push("Van");
console.log(person1.friends); //"Shelby,Court,Van"
console.log(person2.friends); //"Shelby,Court,Van"
console.log(person1.friends === person2.friends); //true
```

### 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。

构造函数模式用于定义**实例属性**，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。

另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。下面的代码重写了前面的例子。

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ["Shelby", "Court"];
}
Person.prototype = {
  constructor: Person,
  sayName: function() {
    console.log(this.name);
  }
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
person1.friends.push("Van");
console.log(person1.friends); // "Shelby,Count,Van"
console.log(person2.friends); // "Shelby,Count"
console.log(person1.friends === person2.friends); // false
console.log(person1.sayName === person2.sayName); // true
```

这种构造函数与原型混成的模式，是目前在`ECMAScript`中使用**最广泛、认同度最高**的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。

### 动态原型模式
> 可以通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。

```javascript
function Person(name, age, job) {
  //属性
  this.name = name;
  this.age = age;
  this.job = job;
  //方法
  if (typeof this.sayName != "function") {
    Person.prototype.sayName = function() {
      console.log(this.name);
    };
  }
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```

这段代码只会在**初次调用构造函数**时才会执行。此后，原型已经完成初始化，不需要再做什么修改了。

这里对原型所做的修改，能够立即在所有实例中得到反映。因此，这种方法确实可以说非常完美。

其中，`if`语句检查的可以是初始化之后应该存在的任何属性或方法——不必用一大堆`if`语句检查每个属性和每个方法；只要检查其中一个即可。

### 寄生构造函数模式

通常，在前述的几种模式都不适用的情况下，可以使用寄生（`parasitic`）构造函数模式。这种模式的基本思想是创建一个函数，该函数的作用**仅仅是封装创建对象的代码**，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数。下面是一个例子。

```javascript
function Person(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function() {
    console.log(this.name);
  };
  return o;
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName(); // "Nicholas"
```

除了使用`new`操作符并把使用的包装函数叫做构造函数之外，这个模式跟工厂模式其实是一模一样的。构造函数在不返回值的情况下，默认会返回新对象实例。而通过在构造函数的末尾添加一个`return`语句，可以重写调用构造函数时返回的值。

这个模式可以在特殊的情况下用来为对象创建构造函数。假设我们想创建一个具有额外方法的特殊数组。由于不能直接修改`Array`构造函数，因此可以使用这个模式：

```javascript
function SpecialArray() {
  // 创建数组
  var values = new Array();
  // 添加值
  values.push.apply(values, arguments);
  // 添加方法
  values.toPipedString = function() {
    return this.join("|");
  };
  // 返回数组
  return values;
}
var colors = new SpecialArray("red", "blue", "green");
console.log(colors.toPipedString()); // "red|blue|green"
```

在这个例子中，我们创建了一个名叫`SpecialArray`的构造函数。在这个函数内部，首先创建了一个数组，然后`push()`方法（用构造函数接收到的所有参数）初始化了数组的值。随后，又给数组实例添加了一个`toPipedString()`方法，该方法返回以竖线分割的数组值。最后，将数组以函数值的形式返回。接着，我们调用了`SpecialArray`构造函数，向其中传入了用于初始化数组的值，此后又调用了`toPipedString()`方法。

> 构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此，不能依赖instanceof 操作符来确定对象类型。由于存在上述问题，我们建议在可以使用其他模式的情况下，不要使用这种模式。

### 稳妥构造函数模式
> 稳妥构造函数模式，不使用new来调用

稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用`this`和`new`），或者在防止数据被其他应用程序（如`Mashup`程序）改动时使用。

稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建对象的实例方法不引用`this`；二是不使用`new`操作符调用构造函数。按照稳妥构造函数的要求，可以将前面的`Person`构造函数重写如下。

```javascript
function Person(name, age, job) {
  // 创建要返回的对象
  var o = new Object();
  // 可以在这里定义私有变量和函数
  // 添加方法
  o.sayName = function() {
    console.log(name);
  };
  // 返回对象
  return o;
}
```

这样，变量`friend`中保存的是一个稳妥对象，而除了调用`sayName()`方法外，没有别的方式可以访问其数据成员。即使有其他代码会给这个对象添加方法或数据成员，但也不可能有别的办法访问传入到构造函数中的原始数据。

## 继承

继承是`OO`语言中的一个最为人津津乐道的概念。许多`OO`语言都支持两种继承方式：**接口继承**和**实现继承**。

接口继承只继承方法签名，而实现继承则继承实际的方法。

如前所述，由于函数没有签名，在`ECMAScript`中无法实现接口继承。**ECMAScript 只支持实现继承**，而且其实现继承主要是依靠原型链来实现的。

### 原型链

简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓原型链的基本概念。

实现原型链有一种基本模式，其代码大致如下。

```javascript
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}
//继承了SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function() {
  return this.subproperty;
};
var instance = new SubType();
console.log(instance.getSuperValue());
```

这个例子中的实例以及构造函数和原型之间的关系如图所示。

![https://sinacloud.net/pro-js/6-4.jpg](https://sinacloud.net/pro-js/6-4.jpg)

原型继承是通过创建`SuperType`的实例，并将该实例赋给`SubType.prototype`实现的。实现的本质是重写原型对象，代之以一个新类型的实例。换句话说，原来存在于`SuperType`的实例中的所有属性和方法，现在也存在于`SubType.prototype`中了。在确立了继承关系之后，我们给SubType.prototype 添加了一个方法，这样就在继承了`SuperType`的属性和方法的基础上又添加了一个新方法。

> 注意instance.constructor 现在指向的是SuperType，这是因为原来SubType.prototype 中的constructor 被重写了的缘故。

#### 别忘记默认的原型
* 默认的原型指向`Object.prototype`

![https://sinacloud.net/pro-js/6-5.jpg](https://sinacloud.net/pro-js/6-5.jpg)

#### 确定原型和实例的关系
* 使用`instanceof`操作符，只要用这个操作符来测试**实例与原型链中出现过的构造函数**，结果就会返回`true`
* 使用`isPrototypeOf()`方法。同样，只要是**原型链中出现过的原型**，都可以说是该原型链所派生的实例的原型，因此`isPrototypeOf()`方法也会返回`true`，如下所示。

#### 谨慎地定义方法

* 子类型有时候需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎样，给原型添加方法的代码一定要放在替换原型的语句之后。

```javascript
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}
// 继承了SuperType
SubType.prototype = new SuperType();
// 添加新方法
SubType.prototype.getSubValue = function() {
  return this.subproperty;
};
// 重写超类型中的方法
SubType.prototype.getSuperValue = function() {
  return false;
};
var instance = new SubType();
console.log(instance.getSuperValue()); // false
```

* 在通过原型链实现继承时，不能使用对象字面量创建原型方法。因为这样做就会重写原型链，如下面的例子所示。

```javascript
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}
// 继承了SuperType
SubType.prototype = new SuperType();
// 使用字面量添加新方法，会导致上一行代码无效
SubType.prototype = {
  getSubValue: function() {
    return this.subproperty;
  },
  someOtherMethod: function() {
    return false;
  }
};
var instance = new SubType();
console.log(instance.getSuperValue()); // error!
```

以上代码展示了刚刚把`SuperType`的实例赋值给原型，紧接着又将原型替换成一个对象字面量而导致的问题。由于现在的原型包含的是一个`Object`的实例，而非`SuperType`的实例，因此我们设想中的原型链已经被**切断**——`SubType`和`SuperType`之间已经没有关系了。

#### 原型链的问题
* 问题一：父类原型中有引用类型，实例会共享，造成影响。
* 问题二：不能给父类构造函数传参。
* 问题三：子类实例对象构造函数指向需重写。

### 借用构造函数（call、apply继承）

在解决原型中包含引用类型值所带来问题的过程中，开发人员开始使用一种叫做**借用构造函数**（`constructor stealing`）的技术（有时候也叫做伪造对象或经典继承）。这种技术的基本思想相当简单，即在子类型构造函数的内部调用超类型构造函数。别忘了，函数只不过是在特定环境中执行代码的对象，因此通过使用`apply()`和`call()`方法也可以在（将来）新创建的对象上执行构造函数，如下所示：

```javascript
function SuperType() {
  this.colors = ["red", "blue", "green"];
}

function SubType() {
  // 继承了SuperType
  SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors); // "red,blue,green,black"
var instance2 = new SubType();
console.log(instance2.colors); // "red,blue,green"
```

通过使用`call()`方法（或`apply()`方法也可以），我们实际上是在（未来将要）新创建的`SubType`实例的环境下调用了`SuperType`构造函数。这样一来，就会在新`SubType`对象上执行`SuperType()`函数中定义的所有对象初始化代码。结果，`SubType`的每个实例就都会具有自己的`colors`属性的副本了。

#### 传递参数
* 相对于原型链而言，借用构造函数有一个很大的优势，即可以在子类型构造函数中向超类型构造函数传递参数。

> 为了确保SuperType构造函数不会重写子类型的属性，可以在调用超类型构造函数后，再添加应该在子类型中定义的属性。

#### 借用构造函数的问题
* 无法做到函数复用

### 组合继承
> 结合原型继承和构造函数的优势的组合继承。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  //继承属性
  SuperType.call(this, name);
  this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  console.log(this.age);
};
var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors); // "red,blue,green,black"
instance1.sayName(); // "Nicholas";
instance1.sayAge(); // 29
var instance2 = new SubType("Greg", 27);
console.log(instance2.colors); // "red,blue,green"
instance2.sayName(); // "Greg";
instance2.sayAge(); // 27
```

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为`JavaScript`中**最常用的继承模式**。而且，`instanceof`和`isPrototypeOf()`也能够用于识别基于组合继承创建的对象。

### 原型式继承（Object.create方法）

```javascript
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

在`object()`函数内部，先创建了一个临时性的构造函数，然后将传入的对象作为这个构造函数的
原型，最后返回了这个临时类型的一个新实例。`object()`对传入其中的对象执行了一次浅复制:

```javascript
var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");
var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
console.log(person.friends); //"Shelby,Court,Van,Rob,Barbie"
```

在这个例子中，可以作为另一个对象基础的是`person`对象，于是我们把它传入到`object()`函数中，然后该函数就会返回一个新对象。这个新对象将`person`作为原型，所以它的原型中就包含一个基本类型值属性和一个引用类型值属性。这意味着`person.friends`不仅属于`person`所有，而且也会被`anotherPerson`以及`yetAnotherPerson`共享。

通过`Object.create()`方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。在传入一个参数的情况下，`Object.create()`与`object()`方法的行为相同。

```javascript
var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");
var yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie"
```

`Object.create()`方法的第二个参数与`Object.defineProperties()`方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。例如：

```javascript
var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = Object.create(person, {
  name: {
    value: "Greg"
  }
});
console.log(anotherPerson.name); // "Greg"
```

> 缺陷：由于原型式继承是浅复制，无法继承引用类型。

### 寄生式继承

寄生式（`parasitic`）继承是与原型式继承紧密相关的一种思路，并且同样也是由克罗克福德推而广之的。

寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。以下代码示范了寄生式继承模式。

```javascript
function createAnother(original) {
  var clone = object(original); // 通过调用函数创建一个新对象
  clone.sayHi = function() { // 以某种方式来增强这个对象
    console.log("hi");
  };
  return clone; // 返回这个对象
}
```

可以像下面这样来使用`createAnother()`函数：

```javascript
var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = createAnother(person);
anotherPerson.sayHi(); // "hi"
```

> 使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率；这一点与构造函数模式类似。

### 寄生组合式继承（综合使用原型链、构造函数和Object.create的继承方式）

组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。没错，子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。再来看一看下面组合继承的例子。

> 为解决多调用一次构造函数造成的内存开销浪费，并且添加了不必要的属性到原型上，所以寄生组合式继承是目前最好的继承方式。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name); // 第二次调用SuperType()
  this.age = age;
}
SubType.prototype = new SuperType(); // 第一次调用SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  console.log(this.age);
};
```

在第一次调用`SuperType`构造函数时，`SubType.prototype`会得到两个属性：`name`和`colors`；它们都是`SuperType`的实例属性，只不过现在位于`SubType`的原型中。当调用`SubType`构造函数时，又会调用一次`SuperType`构造函数，这一次又在新对象上创建了实例属性`name`和`colors`。于是，这两个属性就屏蔽了原型中的两个同名属性。下图展示了上述过程。

![https://sinacloud.net/pro-js/6-6.jpg](https://sinacloud.net/pro-js/6-6.jpg)

如上图所示，有两组`name`和`colors`属性：一组在实例上，一组在`SubType`原型中。这就是调用两次`SuperType`构造函数的结果。好在我们已经找到了解决这个问题方法——寄生组合式继承。

寄生组合式继承的基本模式如下所示:

```javascript
function inheritPrototype(subType, superType) {
  var prototype = object(superType.prototype); //创建对象
  prototype.constructor = subType; //增强对象
  subType.prototype = prototype; //指定对象
}
```

在函数内部，第一步是创建超类型原型的一个副本。第二步是为创建的副本添加`constructor`属性，从而弥补因重写原型而失去的默认的`constructor`属性。最后一步，将新创建的对象（即副本）赋值给子类型的原型。这样，我们就可以用调用`inheritPrototype()`函数的语句，去替换前面例子中为子类型原型赋值的语句了，例如：

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function() {
  console.log(this.age);
};
```

这个例子的高效率体现在它只调用了一次`SuperType`构造函数，并且因此避免了在`SubType.prototype`上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用`instanceof`和`isPrototypeOf()`。开发人员普遍认为寄生组合式继承是引用类型**最理想的继承范式**。