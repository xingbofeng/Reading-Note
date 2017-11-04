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
