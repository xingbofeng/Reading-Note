# 第一章：JavaScript简介
## JavaScript实现
* 核心(`ECMAScript`)
* 文档对象模型(`DOM`)
* 浏览器对象模型(`BOM`)

### ECMAScript
`ECMAScript`由[TC39](https://github.com/tc39)组织提出的`ECMA-262`并维护，现在的`ECMAScript`版本为[ECMAScript 2018](https://tc39.github.io/ecma262/)

各大浏览器厂商都在尽力做到与`ECMA-262`的兼容。

### DOM
`文档对象模型(DOM)`是针对`XML`但经过扩展用于`HTML`的应用程序编程接口。`DOM`把整个页面映射为一个多层节点结构。通过`DOM`创建的这个表示文档的树形图，开发人员获得了控制页面内容和结构的主动权。借助`DOM`提供的`API`，开发人员可以轻松自如地删除、添加、替换或修改任何节点。

`DOM`级别：

1. `DOM1级`由两个模块组成：`DOM核心(DOM Core)`和`DOM HTML`。
2. `DOM2级`引入`DOM视图(DOM View)`、`DOM事件(DOM Events)`、`DOM样式(DOM Style)`、`DOM遍历和范围(DOM Traversal and Range)`。
3. `DOM3级`进一步拓展`DOM`，引入了以统一方式加载和保存文档的方法。增加了`DOM加载和保存(DOM Load and Save)`和`DOM验证(DOM Validation)`。

### BOM
从根本上讲，`BOM`只处理浏览器窗口和框架；但人们习惯上也把所有针对浏览器的`JavaScript`扩展算作`BOM`的一部分。

* 弹出新浏览器窗口的功能
* 移动、缩放和关闭浏览器窗口的功能
* 提供浏览器详细信息的`navigator`对象
* 提供浏览器所加载页面的详细信息的`location`对象
* 提供用户显示器分辨率详细信息的`screen`对象
* 对`cookie`的支持
* 像`XMLHttpRequest`和`IE`的`ActiveXObject`这样的自定义对象

## 小结
`JavaScript`是一种专为与网页交互而设计的脚本语言，由下列三个不同的部分组成：

* `ECMAScript`，由`ECMA-262`定义，提供核心语言的功能
* `文档对象模型(DOM)`，提供访问和操作网页内容的方法和接口
* `浏览器对象模型(BOM)`，提供与浏览器交互的方法和接口

# 第二章：在HTML中使用JavaScript
## <script>元素
`<script>`定义了下列6个属性：

* `async`：可选，表示应该立即下载脚本，但不妨碍页面中的其他操作，比如下载其他资源或等待下载其他脚本。只对外部脚本文件有效。
* `charset`：可选，表示通过`src`属性指定的代码的字符集，由于大多数浏览器会忽略它的值，因此这个属性很少有人用。
* `defer`：可选，表示脚本可以延迟到文档完全被解析和显示之后再执行。只对外部脚本文件有效。
* `language`：已废弃，原来用于表示编写代码使用的脚本语言，大多数浏览器会忽略这个属性，因此没必要再用了。
* `src`：可选，表示包含要执行代码的外部文件。
* `type`：可选，可以看做`language`的替代属性，一般不用设定，或者设定为`text/javascript`。

> 在使用<script>嵌入JavaScript代码时，记住不要在代码中的任何地方出现"</script>"字符串。例如，浏览器在加载下面代码会产生一个错误。

```html
<script>
  function sayScript() {
    alert("</script>");
  }
</script>
```

因为按照解析嵌入式代码的规则，当浏览器遇到字符串`"</script>"`时，就会认为那是结束的`</script>`标签，而通过转义字符`"\"`解决这个问题，例如：

```html
<script>
  function sayScript() {
    alert("<\/script>");
  }
</script>
```

> 需要注意的是，带有src属性的<script>元素不应该在其<script>和</script>标签之间再包含额外的JavaScript代码。如果包含了嵌入的代码，则只会下载并执行外部脚本文件，嵌入的代码会被忽略。

### 标签的位置
在文档的`<head>`元素中包含的所有`JavaScript`文件，意味着必须等到全部`JavaScript`代码都被下载，解析和执行完成以后，才能开始呈现页面的内容，对于那些需要很多`JavaScript`代码的页面来说，这无疑会导致浏览器在呈现页面时出现明显的延迟，而延迟期间的浏览器窗口将是一片空白。为了避免这个问题，现代`Web应用程序`一般都把全部`JavaScript`引用放到`<body>`元素中页面内容的后面。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example HTML Page</title>
</head>
<body>
  <!-- 这里放内容 -->
  <script src="example1.js"></script>
  <script src="example2.js"></script>
</body>
</html>
```

### 延迟脚本
`HTML 4.01`为`<script>`标签定义了`defer`属性。这个属性的用途是表明脚本在执行时不会影响页面的构造。也就是说，脚本会被延迟到整个页面都解析完毕后再运行。因此，在`<script>`元素中设置`defer`属性，相当于告诉浏览器立即下载，但延迟执行。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example HTML Page</title>
  <script defer="defer" src="example1.js"></script>
  <script defer="defer" src="example2.js"></script>
</head>
<body>
  <!-- 这里放内容 -->
</body>
</html>
```

### 异步脚本
`HTML5`为`<script>`元素定义了`async`属性。这个属性与`defer`属性类似，都用于改变处理脚本的行为。同样与`defer`类似，`async`只适用于外部脚本文件，并告诉浏览器立即下载文件。但与`defer`不同的是，标记为`async`的脚本并不保证按照指定它们的先后顺序执行。例如：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example HTML Page</title>
  <script async src="example1.js"></script>
  <script async src="example2.js"></script>
</head>
<body>
  <!-- 这里放内容 -->
</body>
</html>
```

在以上代码，第二个脚本文件可能会在第一个脚本文件之前执行。因此，确保两者之间互不依赖非常重要。指定`async`属性的目的是不让页面等待两个脚本下载和执行，从而异步加载页面的其他内容。

## 嵌入代码与外部文件
在`HTML`中嵌入`JavaScript`代码虽然没有问题，但一般认为最好的做法还是尽可能使用外部文件包含`JavaScript`代码。不过并不存在必须使用外部文件的硬性规定，但支持使用外部文件的人多会强调以下优点：

* 可维护性
* 可被浏览器缓存
* 适应未来

## 文档模式
两种模式：

* 混杂模式
* 标准模式

混杂模式会让`IE`的行为与低版本浏览器相同，而标准模式则让`IE`的行为更接近标准行为。尽量定义为标准模式`<!DOCTYPE html>`

## <noscript>元素
`<noscript>`元素中的内容只有在下列情况下才会显示出来：

* 浏览器不支持脚本。
* 浏览器支持脚本，但脚本被禁用。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example HTML Page</title>
  <script defer="defer" src="example1.js"></script>
  <script defer="defer" src="example2.js"></script>
</head>
<body>
  <noscript>
    <p>本页面需要浏览器支持（启用）JavaScript</p>
  </noscript>
</body>
</html>
```

## 小结
* 在包含外部`JavaScript`文件时，必须将`src`属性设置为指向相应文件的`URL`。而这个文件既可以是与包含它的页面位于同一个服务器上的文件，也可以是其他任何域中的文件。
* 所有`<script>`元素都会按照它们在页面中出现的先后顺序依次被解析。在不使用`defer`和`async`属性的情况下，只有在解析完前面`<script>`元素中的代码之后，才会开始解析后面`<script>`元素中的代码。
* 由于浏览器会先解析完不使用`defer`属性的`<script>`元素中的代码，然后再解析后面的内容，所以一般应该把`<script>`元素放在页面最后，即主要内容后面，`<body>`标签前面。
* 使用`defer`属性可以让脚本在文档完全呈现之后再执行。延迟脚本总是按照指定它们的顺序执行。
* 使用`async`属性可以表示当前脚本不必等待其他脚本，也不必阻塞文档呈现。不能保证异步脚本按照它们在页面中出现的顺序执行。
* `<noscript>`元素中的内容只有在浏览器不支持脚本或脚本被禁用的情况下才会显示出来。

# 第三章：基本概念
## 语法
## 关键字和保留字
## 变量
* 在函数体内，不用`var`声明的变量会声明为全局变量。
* 严格模式下，不能定义名为`eval`或`arguments`的变量。

## 数据类型
* 对一个值使用`typeof`操作符，可能返回下列某个字符串："undefined"、"boolean"、"string"、"number"、"object"、"function"。
* 声明变量后默认取得`undefined`值，而没声明变量直接使用则会抛出一个错误。

```javascript
var message;

console.log(message); // undefined
console.log(age); // 报错
```

* `null`表示空对象指针，对`null`使用`typeof`会返回"object"
* `Boolean`类型的隐式转换
* 关于数值范围，最小的小数值为`Number.MIN_VALUE`，即`5e-324`，最大数值为`Number.MAX_VALUE`，即`1.7976931348623157e+308`。一旦运算结果得到了超过`Number.MAX_VALUE`的值，则这个数值自动被转换成`Infinity`值，具体来说，如果这个数值是负数，则会被转化为`-Infinity`。
* 判断一个数是否是有穷的，可以用`isFinite()`函数。这个函数在参数位于最小与最大数值之间时会返回`true`。
* `NaN`与任何值都不想等，包括自身。判断一个值是否非数值，或者是否能够被转化为数值。

```javascript
console.log(NaN == NaN); // false
console.log(isNaN(NaN)); // true
console.log(isNaN(10)); // false，因为10是一个数值
console.log(isNaN('10')); // false，因为字符串10可以转换成数值10
console.log(isNaN('blue')); // true，因为字符串blue不能被转换成数值
console.log(isNaN(true)); // false，因为true可以转化为数值1
```

* 数值转换有三个函数可以把非数值转换为数值：`Number()`、`parseInt()`、`parseFloat()`。其中`Number()`函数会忽略字符串前面的`0`、`0x`等，直接以十进制来转换。如果是`null`值，转为`0`。如果是`undefined`，转为`NaN`。

* `Number()`函数对进制不敏感，而`parseInt()`对进制敏感。并且`parseInt()`可以带第二个参数，比如带上`16`的时候，可以不指定`0x`。
* `parseFloat()`只解析十进制值，会忽略前面的`0`。

```javascript
console.log(parseInt('10', 2)); // 2
console.log(parseInt('10', 8)); // 8
console.log(parseInt('10', 10)); // 10
console.log(parseInt('10', 16)); // 16
console.log(parseInt('AF', 16)); // 175
console.log(parseInt('AF')); // NaN（因为这里按照十进制解析了）
console.log(parseFloat('0908.5')); // 908.5（忽略前面的0）
```

* 数值、布尔值、对象和字符串值都有`toString()`方法，但`null`和`undefined`没有`toString()`方法。对于数字来说，`toString()`，可以指定进制。

```javascript
var num = 10;

console.log(num.toString()); // "10"
console.log(num.toString(2)); // "1010"
console.log(num.toString(8)); // "12"
console.log(num.toString(10)); // "10"
console.log(num.toString(16)); // "a"
```

* 在不知道要转换的值是不是`null`或`undefined`的情况下，还可以使用转型函数`String()`，这个函数能够将任何类型的值转换为字符串。`String()`函数把`null`转化为字符串"null"，把`undefined`转化为字符串"undefined"，对于其它有`toString()`方法的类型则调用`toString()`方法。

## 操作符
* `>>>`三个`>`符号表示无符号的右移。

## 语句
* `label`语句，其基本语法为`label: statement`，其可以在代码中添加标签。一般可以在将来与`break`和`continue`语句联合使用：

```javascript
var num = 0;
outermost:
  for (var i = 0; i < 10; i++) {
    for (var j = 0; j < 10; j++) {
      if (i == 5 && j == 5) {
        break outermost; // 退出指定位置的循环
      }
      num++;
    }
  }

console.log(num); // 55
```

```javascript
var num = 0;
outermost:
  for (var i = 0; i < 10; i++) {
    for (var j = 0; j < 10; j++) {
      if (i == 5 && j == 5) {
        continue outermost; // 跳到指定位置的循环
      }
      num++;
    }
  }

console.log(num); // 95
```

* `with`语句的作用是把代码的作用域设置到一个特定的对象中。定义`with`语句的目的主要是为了简化多次编写同一个对象的工作。

```javascript
var qs = location.search.substring(1);
var hostName = location.hostName;
var url = location.href;
```

上面的代码都用了`location`对象，如果使用`with`语句，可以把上面的代码改下成以下形式：

```javascript
with(location) {
  var qs = search.substring(1);
  var hostName = hostName;
  var url = href;
}
```

> 大量使用with语句会导致性能下降，不建议使用with语句，严格模式下，不允许使用with语句。

## 函数
> 注意，JavaScript参数没有重载，不过可以通过检查传入函数参数的类型和数量并作出不同的反应，可以模仿方法的重载。

# 第二十章：JSON
## 语法
* 最简单的`JSON`数据形式就是简单值，如数字、字符串、布尔值、`null`都属于`JSON`。
* `JSON`对象属性名必须用双引号，字符串必须用双引号。
* `JSON.stringify()`可以过滤结果，还可以字符串缩进，分别为其传入的第二个参数和第三个参数。

```javascript
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  "edition": 3,
  "year": 2011
};

var jsonText = JSON.stringify(book, ["title", "edition"]);
console.log(jsonText); // "{"title":"Professional JavaScript","edition":3}"
```

第二个参数还可以是函数，这个函数作为过滤器：

```javascript
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  "edition": 3,
  "year": 2011
};

var jsonText = JSON.stringify(book, function(key, value) {
  switch(key) {
    case "authors":
      return value.join(",");
    case "year":
      return 5000;
    case "edition":
      return undefined;
    default:
      return value;
  }
});
console.log(jsonText); // {"title":"Professional JavaScript","authors":"Nicholas C. Zakas","year":5000}
```

> 注意其中的switch语句不要忘记default！

第三个参数指定字符串缩进：

```javascript
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  "edition": 3,
  "year": 2011
};

var jsonText = JSON.stringify(book, null, 4);
console.log(jsonText);

/* 结果如下，缩进四个空格：

{
    "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    "edition": 3,
    "year": 2011
}

*/
```

* 为对象显式定义`toJSON()`方法。

```javascript
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  "edition": 3,
  "year": 2011,
  toJSON: function() {
    return this.title;
  }
};

var jsonText = JSON.stringify(book);
console.log(jsonText);
```

一旦对象显式定义了`toJSON()`方法，调用`JSON.stringify()`方法则是调用该方法，拿到返回值。如果在这里的`JSON.stringify()`方法还定义了第二个参数的过滤器函数和第三个参数字符串缩进，过滤器过滤的其实是`toJSON()`方法的返回值。

* 解析选项：`JSON.parse()`也可以接受第二个参数，该参数是一个函数，称为还原函数。

```javascript
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  "edition": 3,
  "year": 2011,
  releaseDate: new Date(2011, 11, 1)
};

var jsonText = JSON.stringify(book);

var bookCopy = JSON.parse(jsonText, function(key, value) {
  if (key == "releaseDate") {
    return new Date(value);
  } else {
    return value;
  }
});

console.log(bookCopy.releaseDate.getFullYear());
```

> 如果没有还原函数，bookCopy中就无法还原Date对象，也就无法调用getFullYear()方法。还原函数的作用在此。