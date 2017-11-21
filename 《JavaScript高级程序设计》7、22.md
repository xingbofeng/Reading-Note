# 第七章：函数表达式
* 函数声明有函数声明提升，函数表达式则只提升变量。
* 匿名函数又叫拉姆达函数，匿名函数的`name`属性是空字符串。

> 注意，由于函数声明提升的存在，并且JavaScript没有块级作用域，因此不要使用下列语法。

```javascript
if (condition) {
  function sayHi() {
    console.log('Hi!');
  }
} else {
  function sayHi() {
    console.log('Yo!');
  }
}
```

上述代码会忽略`condition`的判断条件，产生不可预知的结果。

## 递归
## 闭包
闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。

当某个函数被调用时，会创建一个执行环境（`execution context`）及相应的作用域链。然后，使用`arguments`和其他命名参数的值来初始化函数的活动对象（`activation object`）。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位，……直至作为作用域链终点的全局执行环境。

```javascript
function compare(value1, value2) {
  if (value1 < value2) {
    return -1;
  } else if (value1 > value2) {
    return 1;
  } else {
    return 0;
  }
}
var result = compare(5, 10);
```

当调用`compare()`时，会创建一个包含`arguments`、`value1`和`value2`的活动对象。全局执行环境的变量对象（包含`result`和`compare`）在`compare()`执行环境的作用域链中则处于第二位。下图展示了包含上述关系的
`compare()`函数执行时的作用域链。

![https://sinacloud.net/pro-js/7-1.jpg](https://sinacloud.net/pro-js/7-1.jpg)

* 作用域链被保存在`[[Scope]]`属性中。

> 注意：由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过度使用闭包可能会导致内存占用过多。

### 闭包与变量
* 作用域链的这种配置机制引出了一个值得注意的副作用，即闭包只能取得包含函数中任何变量的最后一个值。

```javascript
function createFunctions() {
  var result = new Array();
  for (var i = 0; i < 10; i++) {
    result[i] = function() {
      return i;
    };
  }
  return result; // 这些函数取得的变量i都是同一个变量
}
```

通过创建另一个匿名函数强制让闭包的行为符合预期，如下所示。

```javascript
function createFunctions() {
  var result = new Array();
  for (var i = 0; i < 10; i++) {
    // 函数立即执行
    result[i] = function(num) {
      return function() { // 这里的形参num都不一样，因为参数按值传递
        return num;
      };
    }(i);
  }
  return result;
}
```

如上，定义了一个匿名函数，并将立即执行该匿名函数的结果赋给数组。这里的匿名函数有一个参数`num`，也就是最终的函数要返回的值。在调用每个匿名函数时，我们传入了变量`i`。由于函数参数是**按值传递**的，所以就会将变量`i`的当前值复制给参数`num`。

### 关于this对象
* 全局环境中的`this`：指向`window`。
* 作为单纯的函数调用：指向`window`，严格模式下是`undefined`。
* 作为对象的方法调用：指向该对象。
* 作为构造函数调用：指向新创建的对象。
* 闭包中：指向`window`。

### 内存泄漏
* 在不需要时，把闭包中的引用对象手动置为`null`可以防止内存泄漏。

```javascript
function assignHandler() {
  var element = document.getElementById("someElement");
  var id = element.id;
  element.onclick = function() {
    console.log(id);
  };
  element = null;
}
```

把`element.id`的一个副本保存在一个变量中，并且在闭包中引用该变量消除了循环引用，再把`element`变量设置为`null`。这样就能够解除对`DOM`对象的引用，顺利地减少其引用数，确保正常回收其占用的内存。

## 模仿块级作用域
* 通过自执行匿名函数。

## 私有变量
* 可以通过闭包创建私有变量。

```javascript
function Person(name) {
  this.getName = function() {
    return name;
  };
  this.setName = function(value) {
    name = value;
  };
}
var person = new Person("Nicholas");
console.log(person.getName()); // "Nicholas"
person.setName("Greg");
console.log(person.getName()); // "Greg"
```

### 静态私有变量
通过在私有作用域中定义私有变量或函数，同样也可以创建特权方法，其基本模式如下所示。

特权方法是在原型上定义的，因此所有实例都使用同一个函数。而这个特权方法，作为一个闭包，总是保存着对包含作用域的引用。来看一看下面的代码。

```javascript
(function() {
  var name = "";
  Person = function(value) {
    name = value;
  };
  Person.prototype.getName = function() {
    return name;
  };
  Person.prototype.setName = function(value) {
    name = value;
  };
})();
var person1 = new Person("Nicholas");
console.log(person1.getName()); // "Nicholas"
person1.setName("Greg");
console.log(person1.getName()); // "Greg"
var person2 = new Person("Michael");
console.log(person1.getName()); // "Michael"
console.log(person2.getName()); // "Michael"
```

在这种模式下，变量`name`就变成了一个静态的、由所有实例共享的属性。而调用`setName()`或新建一个`Person`实例都会赋予`name`属性一个新值。结果就是所有实例都会返回相同的值。

> 多查找作用域链中的一个层次，就会在一定程度上影响查找速度。而这正是使用闭包和私有变量的一个明显的不足之处。

### 模块模式
关于单例模式，可见暑期写的设计模式的学习笔记。

通过返回一个对象，实现模块模式。

```javascript
var application = function() {
  // 私有变量和函数
  var components = new Array();
  // 初始化
  components.push(new BaseComponent());
  // 公共
  return {
    getComponentCount: function() {
      return components.length;
    },
    registerComponent: function(component) {
      if (typeof component == "object") {
        components.push(component);
      }
    }
  };
}();
```

### 增强的模块模式
有人进一步改进了模块模式，即在返回对象之前加入对其增强的代码。这种增强的模块模式适合那些单例必须是某种类型的实例，同时还必须添加某些属性和（或）方法对其加以增强的情况。

```javascript
var application = function() {
  // 私有变量和函数
  var components = new Array();
  // 初始化
  components.push(new BaseComponent());
  // 创建 application 的一个局部副本
  var app = new BaseComponent();
  // 公共接口
  app.getComponentCount = function() {
    return components.length;
  };
  app.registerComponent = function(component) {
    if (typeof component == "object") {
      components.push(component);
    }
  };
  // 返回这个副本
  return app;
}();
```

在这个重写后的应用程序（`application`）单例中，首先也是像前面例子中一样定义了私有变量。主要的不同之处在于命名变量`app`的创建过程，因为它必须是`BaseComponent`的实例。这个实例实际上是`application`对象的局部变量版。此后，我们又为`app`对象添加了能够访问私有变量的公有方法。最后一步是返回`app`对象，结果仍然是将它赋值给全局变量`application`。

# 第二十二章：高级技巧
## 高级函数
### 安全的类型检测
* `typeof`不可靠，对于`null`返回`object`，检测正则对象可能返回`function`。
* `instanceof`不可靠，对于多个全局作用域情况下产生问题。
* 建议使用`Object.prototype.toString.call()`。
* 检测函数合集：

```javascript
var isType = {
  isString: function(value) {
    return typeof value === 'string';
  },
  isBoolean: function(value) {
    return typeof value === 'boolean';
  },
  isNumber: function(value) {
    return typeof value === 'number';
  },
  isUndefined: function(value) {
    return typeof value === undefined;
  },
  isNull: function(value) {
    return value === null;
  },
  isObject: function(value) {
    if (value === null) {
      return false;
    } else if (typeof value === 'function' || typeof value === 'object') {
      return true;
    }
  },
  isArray: function(value) {
    return Object.prototype.toString.call(value) === '[object Array]';
  },
  isFunction: function(value) {
    return Object.prototype.toString.call(value) === '[object Function]';
  },
  isRegExp: function(value) {
    return Object.prototype.toString.call(value) === '[object RegExp]';
  },
  isNativeJSON: function() {
    return window.JSON && Object.prototype.toString.call(JSON) === '[object JSON]';
  }
};
```

### 作用域安全的构造函数
> 为了防止不使用new调用构造函数时对全局对象window或者global的属性进行修改，故需要使用作用域安全的构造函数。

```javascript
function Person(name, age, job) {
  // 首先检测确认 this 是否为正确类型的实例
  if (this instanceof Person) {
    this.name = name;
    this.age = age;
    this.job = job;
  } else {
    return new Person(name, age, job)
  }
}
```

上述模式在继承链中会受到影响，看如下例子。

```javascript
function Polygon(sides) {
  if (this instanceof Polygon) {
    this.sides = sides;
    this.getArea = function() {
      return this.sides;
    };
  } else {
    return new Polygon(sides);
  }
}

function Rectangle(width, height) {
  Polygon.call(this, 2);
  this.width = width;
  this.height = height;
  this.getArea = function() {
    return this.width * tghis.height;
  };
}

var rect = new Rectangle(5, 10);
console.log(rect.sides); // undefined
```

在上面的代码中，`Polygon` 构造函数的作用域是安全的，然而 `Rectangle` 则不是，`Polygon.call(this, 2)` 中的 `this` 对象并不是 `Polygon` 的实例，所以最终导致 `Rectangle` 构造函数中的 `this` -- 即新建的 `Rectangle` 实例没有得到增长，`Polygon.call()` 返回的对象又没有用到。

> 解决方法：构造函数窃取 + 原型链继承（寄生组合继承）

> 关键代码：Rectangle.prototype = new Polygon(); ，使得能通过 if (this instanceof Polygon) 判断

```javascript
function Polygon(sides) {
  if (this instanceof Polygon) {
    this.sides = sides;
    this.getArea = function() {
      return this.sides;
    };
  } else {
    return new Polygon(sides);
  }
}

function Rectangle(width, height) {
  Polygon.call(this, 2);
  this.width = width;
  this.height = height;
  this.getArea = function() {
    return this.width * tghis.height;
  };
}

Rectangle.prototype = new Polygon(); // 使得能通过 if (this instanceof Polygon) 判断

var rect = new Rectangle(5, 10);
console.log(rect.sides); // undefined
```

### 惰性载入函数
需要惰性载入函数的原因？

> 因为浏览器的差异，我们常常需要包含大量的if语句做兼容，如果每次运行一个函数，都需要检测if中的条件，这无疑对资源造成了浪费。惰性载入函数让我们不必每次都运行if语句，只需要第一次运行后记下分支的正确走向，之后就可以避开判断。

实现方式：

* 函数在第一次调用时，内部再处理（重写）函数 -- 第一次调用惰性判断

```javascript
function process() {
  if (...) {
    process = function() {
      ... // 重写 process 函数
    }
  } else {
    process = function() {
      ... // 重写 process 函数
    }
  }
}
```

* 在声明函数的时候就根据分支条件指定适当的函数 -- 代码首次加载时惰性判断

```javascript
var process = (function() {
  if (...) {
    return function() {...
    }
  } else {
    return function() {...
    }
  }
})();
```

优势：只在第一次执行分支代码时牺牲一点儿性能，之后便可以避免执行不必要的代码。

### 函数绑定
函数绑定是创建一个函数，可以在特定的`this`环境中指定参数调用另一个函数。函数绑定的技巧常常和回调函数与事件处理程序一起使用，用以确保函数执行时的执行环境；

* 使用`call()`、 `apply()`和`bind()`实现函数绑定。

模拟的`bind`实现：

* [从一道面试题的进阶，到“我可能看了假源码”](https://zhuanlan.zhihu.com/p/25379434)
* [Javascript中bind()方法的使用与实现](https://segmentfault.com/a/1190000002662251)

```javascript
Function.prototype.bind = Function.prototype.bind || function(context) {
  var me = this;
  var args = Array.prototype.slice.call(arguments, 1);
  var F = function() {};
  F.prototype = this.prototype;
  var bound = function() {
    var innerArgs = Array.prototype.slice.call(arguments);
    var finalArgs = args.contact(innerArgs);
    return me.apply(this instanceof F ? this : context || this, finalArgs);
  }
  bound.prototype = new F();
  return bound;
}
```

### 函数柯里化（currying）
`JavaScript`中函数柯里化常用于实现延迟计算、参数复用、动态生成函数等功能。

> 注意需要在柯里化的同时为函数绑定好执行环境

```javascript
function curry(fn, context) {
  var args = Array.prototype.slice.call(arguments, 2);

  return function() {
    var innerArgs = Array.prototype.slice.call(arguments);
    var finalArgs = args.concat(innerArgs);
    return fn.apply(context, finalArgs)
  }
}

// 使用实例
function add(num1, num2) {
  return num1 + num2;
}

var curriedAdd = curry(add, null, 5); // 传入待柯里化的函数、执行环境以及默认参数
curriedAdd(3); // 8
```

* [闭包和高阶函数](https://github.com/xingbofeng/JavaScript-design-patterns/blob/master/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0.md#currying)
* [聊聊柯里化](https://segmentfault.com/a/1190000011825142)
* [JavaScript专题之函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42)

## 防篡改对象
### 不可拓展对象
* 使用`Object.preventExtensions()`使得对象不可拓展，不能再次给对象添加属性和方法。
* 使用`Object.isExtensible()`判断对象是否可拓展。

### 密封的对象
* 使用`Object.seal()`使得对象被密封，不能给对象添加或删除方法。
* 使用`Object.isSealed()`确定对象是否被密封了。

### 冻结的对象
* 使用`Object.freeze()`使得对象被冻结，冻结的对象不可拓展并且密封，并且对象的`[[Writable]]`特性被设置为`false`。如果定义`[[Set]]`函数，访问器属性仍然是可写的。

> 对于JavaScript库作者而言，冻结对象是很有用的。因为JavaScript库最怕有人意外修改库中的核心对象。冻结（或密封）主要的库对象能够防止这些问题的发生。

## 高级定时器
### 重复的定时器
使用 `setInterval()` 间歇调用的优缺点：

* 优点
  * 确保了定时器代码规则地插入到队列中
* 问题
  * 某些间隔会被跳过
  * 多个定时器代码之间的间隔可能比预期的小（由于队列机制）

* 某些间隔会被跳过的原因：定时器代码可能在代码再次被添加到队列之前还没有完成执行，结果导致定时器代码发生叠加的情况是不会发生的，因为 `JavaScript` 引擎已经给我们做好了优化。
    
> 当使用setInterval()时，仅当没有该定时器的任何其他代码实例时，才将定时器代码添加到队列中。

解决方案：使用递归`setTimeout`模式链式调用：

```javascript
setTimeout(function() {
  // ...处理中
  doSomething();
  setTimeout(arguments.callee, interval);
}, interval);
```

链式超时调用代替间歇调用的好处：

* 在前一个定时器代码执行完之前，不会向队列插入新的定时器代码，确保不会有任何缺失的间隔
* 可以保证在下一次定时器代码执行之前，至少要等待指定的间隔，避免了连续的执行

### Yielding Processes
* 由于浏览器的内存配额限制，运行在浏览器的`JavaScript`不同于桌面应用（能随意控制他们要的内存大小和处理器时间），一般内存分配与处理器时间的分配都有限制，这一措施是为了恶意程序导致系统崩溃。当代码运行超过特定时间、超过特定的语句数量、超过特定的内存限制时浏览器会有中断机制，以及询问用户确认是否继续运行的机制。
* 脚本长时间运行的问题通常有两个原因：过长的、过深嵌套的函数调用或进行大量处理的循环
* 使用 `Yielding Processes` 的三个条件：`一个循环占用了大量的时间`、`循环内的处理不需要同步完成，即循环中的处理可以推迟一些处理，而不会对循环之后的代码执行造成影响`、`数据不需要按顺序完成`。
* 使用数组分块技术：

```javascript
setTimeout(function() {
  // 取出下一个条目并处理
  var item = arr.shift();
  process(item);

  // 若还有条目，再设置另一个定时器
  if (arr.length > 0) {
    setTimeout(arguments.callee, 100);
  }
}, 100);
```

可以`chunk`化数组分块封装函数：

```javascript
function chunk(arr, process, context) {
  setTimeout(function() {
    var item = arr.shift();
    process.call(context, item);

    if (arr.length > 0) {
      setTimeout(arguments.callee, 100);
    }
  })
}
```

> 一旦某个函数需要花50ms以上的时间完成，那么最好看看能否将任务分别为一系列可以使用定时器的小任务。

### 函数节流
* 目的：防止连续进行过多的 DOM 操作（需要很多的内存和 CPU 时间）会导致浏览器挂起，`onreize` 事件在部分浏览器下会随着页面的拉伸连续不断的出发，`onscroll`、`onmousemove` 也有同样的问题。

* 书上的模式：

```javascript
function throttle(method, context) {
  clearTimeout(method.tId);
  timer = setTimeout(function() {
    method.call(context);
  }, 100)
}

// 使用方法：以 resize 事件处理程序为例
function resizeDiv() {
  var div = document.getElementById('myDiv');
  div.style.height = div.offsetWidth + 'px';
}

window.onresize = function() {
  throttle(resizeDiv);
}
```

* 我常用的模式：

```javascript
var flag = false;

function throttle(method, context) {
  if (flag) {
    return;
  }
  setTimeout(function() {
    method.call(context);
    flag = false;
  }, 100);
}

// 使用方法：以 resize 事件处理程序为例
function resizeDiv() {
  var div = document.getElementById('myDiv');
  div.style.height = div.offsetWidth + 'px';
}

window.onresize = function() {
  throttle(resizeDiv);
}
```

* [闭包和高阶函数之函数节流](https://github.com/xingbofeng/JavaScript-design-patterns/blob/master/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0.md#函数节流)

* `underscore`源码之`throttle`

```javascript
/**
 * 频率控制 返回函数连续调用时，func 执行频率限定为 次 / wait
 * 
 * @param  {function}   func      传入函数
 * @param  {number}     wait      表示时间窗口的间隔
 * @param  {object}     options   如果想忽略开始边界上的调用，传入{leading: false}。
 *                                如果想忽略结尾边界上的调用，传入{trailing: false}
 * @return {function}             返回客户调用函数   
 */
_.throttle = function(func, wait, options) {
  var context, args, result;
  var timeout = null;
  // 上次执行时间点
  var previous = 0;
  if (!options) options = {};
  // 延迟执行函数
  var later = function() {
    // 若设定了开始边界不执行选项，上次执行时间始终为0
    previous = options.leading === false ? 0 : _.now();
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };
  return function() {
    var now = _.now();
    // 首次执行时，如果设定了开始边界不执行选项，将上次执行时间设定为当前时间。
    // 否则previous这里是0，如果是0，remaining肯定小于0，则会执行func.apply(context, args);这条语句，执行业务逻辑
    if (!previous && options.leading === false) previous = now;
    // 延迟执行时间间隔
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    // 延迟时间间隔remaining小于等于0，表示上次执行至此所间隔时间已经超过一个时间窗口
    // remaining大于时间窗口wait，表示客户端系统时间被调整过
    if (remaining <= 0 || remaining > wait) {
      // 清除定时器防止内存泄露
      // clearTimeout(timeout)后，timeout的值并不会清空，如果不设置为null，就不能根据timeout设置下次的timeout
      clearTimeout(timeout);
      timeout = null;
      // 把previous置为现在的时间戳(下次判断用)
      previous = now;
      // 调用func
      result = func.apply(context, args);
      if (!timeout) context = args = null;
      //如果延迟执行不存在，且没有设定结尾边界不执行选项
    } else if (!timeout && options.trailing !== false) {
      // 设定把timeout定时器，设定为remaining毫秒后推入定时器回调队列执行later回调函数
      timeout = setTimeout(later, remaining);
    }
    return result;
  };
};
```

* `underscore`源码之`debounce`

```javascript
/**
 * 空闲控制 返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
 *
 * @param  {function} func        传入函数
 * @param  {number}   wait        表示时间窗口的间隔
 * @param  {boolean}  immediate   设置为ture时，调用触发于开始边界而不是结束边界
 * @return {function}             返回客户调用函数
 */
_.debounce = function(func, wait, immediate) {
  var timeout, args, context, timestamp, result;

  var later = function() {
    // 据上一次触发时间间隔
    var last = _.now() - timestamp;

    // 上次被包装函数被调用时间间隔last小于设定时间间隔wait
    // 重新设定定时器，不过时间更新了！
    if (last < wait && last > 0) {
      timeout = setTimeout(later, wait - last);
    } else {
      timeout = null;
      // 如果设定为immediate===true，因为开始边界已经调用过了此处无需调用
      if (!immediate) {
        // 执行业务逻辑
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      }
    }
  };

  return function() {
    context = this;
    args = arguments;
    timestamp = _.now();
    // 立即触发需要满足两个条件：immediate为true，且timeout不是null
    // 根据 timeout 是否为空可以判断是否是首次触发
    var callNow = immediate && !timeout;
    // 如果延时不存在，重新设定延时
    if (!timeout) timeout = setTimeout(later, wait);
    if (callNow) {
      // 达到立即触发的条件（第一次调用），调用业务逻辑函数
      result = func.apply(context, args);
      context = args = null;
    }

    return result;
  };
};
```

## 自定义事件
* 基于观察者模式实现自定义事件。
* 基本模式：`EventTarget`类

```javascript
function EventTarget() {
  this.handlers = {};
}

EventTarget.prototype = {
  constructor: EventTarget,
  addHandler: function(type, handler) {
    if (typeof this.handlers[type] == undefined) {
      this.handlers[type] = []; // 如果 ET 的实例上还没有该事件类型，便建立该事件类型
    }
    this.handlers[type].push(handler); // 如果已有该事件类型，便将事件处理程序推入该事件的处理程序列表中
  },
  fire: function(event) {
    if (!event.target) {
      event.target = this;
    }
    if (this.handlers[event.type] instanceof Array) { // 检查事件类型是否已注册
      var handlers = this.handlers[event.type]; // handlers 为所有事件处理程序的数组
      for (var i = 0, len = handlers.length; i < len; i++) {
        handlers[i](event); // 触发数组中的每一个事件处理程序
      }
    }
  },
  removeHandler: function(type, handler) {
    if (this.handlers[type] instanceof Array) { // 检查事件类型是否已注册
      var handlers = this.handlers[type];
      for (var i = 0, len = handlers.length; i < len; i++) { // 遍历寻找事件处理程序员
        if (handlers[i] === handler) {
          break;
        }
        handlers.splice(i, 1); // 删除传入的事件处理程序的对应项
      }
    }
  }
};
```

* 使用方法

```javascript
function handlerMessage(event) {
  console.log('Message received: ' + event.message);
}

// 创建一个新对象
var traget = new EventTarget();

// 添加一个事件处理程序
target.addHandler('message', handleMessage);

// 触发事件，以事件对象的形式传入事件的相关信息
target.fire({
  type: 'message',
  message: 'Hi!'
});

// 删除事件处理程序
target.removeHandler('message', handleMessage);

// 删除后无法再触发
target.fire({
  type: 'message',
  message: 'Hi!'
});
```

* 使用寄生组合继承使得其他对象可以继承 `EventTarget`

```javascript
function Person(name, age) {
  EventTarget.call(this);
  this.name = name;
  this.age = age;
}

function inheritPrototype(subType, superType) {
  var prototype = Object(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}

inheritPrototype(Person, EventTarget);

Person.prototype.say = function(message) {
  this.fire({
    type: 'message',
    message: message
  });
}

// 调用方法
function handleMessage(event) {
  console.log(event.target.name + "say: " + event.message);
}

var person = new Person('yangfch3', 21);

person.addHandler('message', handleMessage);

person.say('Hi!');
```

## 拖放
* 使用`mousedown`、`mousemove`、`mouseup`事件。
* 可以使用`EventTarget`类基于发布订阅模式封装拖放事件。

```javascript
function EventTarget() {
  this.handlers = {};
}

EventTarget.prototype = {
  constructor: EventTarget,
  addHandler: function(type, handler) {
    if (typeof this.handlers[type] == undefined) {
      this.handlers[type] = [];
    }
    this.handlers[type].push(handler);
  },
  fire: function(event) {
    if (!event.target) {
      event.target = this;
    }
    if (this.handlers[event.type] instanceof Array) {
      var handlers = this.handlers[event.type];
      for (var i = 0, len = handlers.length; i < len; i++) {
        handlers[i](event);
      }
    }
  },
  removeHandler: function(type, handler) {
    if (this.handlers[type] instanceof Array) {
      var handlers = this.handlers[type];
      for (var i = 0, len = handlers.length; i < len; i++;) {
        if (handlers[i] === handler) {
          break;
        }
        handlers.splice(i, 1);
      }
    }
  }
};

var DragDrop = function() {
  var dragdrop = new EventTarget(),
    dragging = null,
    diffX = 0,
    diffY = 0;

  function handleEvent(event) {
    // 获取事件和对象
    event = eventUtil.getEvent(event);
    var target = eventUtil.getTarget(event);

    // 确定事件类型
    switch (event.type) {
      case 'mousedown':
        if (target.className.indexOf('draggable') > -1) {
          dragging = traget;
          diffX = event.clientX - target.offsetLeft;
          diffY = event.clientY - traget.offsetTop;

          dragdrop.fire({
            type: 'dragstart',
            target: dragging,
            x: event.clientX,
            y: event.clientY
          });
        }
        break;

      case 'mousemove':
        if (dragging !== null) {
          // 指定位置
          dragging.style.left = (event.clientX - diffX) + 'px';
          dragging.style.top = (event.clientY - diffY) + 'px';

          // 触发自定义事件
          dragdrop.fire({
            type: 'drag',
            target: dragging,
            x: event.clientX,
            y: event.clientY
          });
        }
        break;

      case 'mouseup':
        dragdrop.fire({
          type: 'dragend',
          traget: dragging,
          x: event.clientX,
          y: event.clientY
        });
        dragging = null;
        break;
    }
  }

  dragdrop.enable = function() {
    eventUtil.addEventHandler(document, 'mousedown', handleEvent);
    eventUtil.addEventHandler(document, 'mousemove', handleEvent);
    eventUtil.addEventHandler(document, 'mouseup', handleEvent);
  };

  dragdrop.disable = function() {
    eventUtil.removeEventHandler(document, 'mousedown', handleEvent);
    eventUtil.removeEventHandler(document, 'mousemove', handleEvent);
    eventUtil.removeEventHandler(document, 'mouseup', handleEvent);
  };

  return dragdrop;

}();
```

上述例子中，我们创建了三个事件：`dragstart` `drag` `dragend`，受到启发，我们很容易地能够使用浏览器原生事件进行再加工指定出一系列自定义事件；

三个自定义事件调用的方法：

```javascript
DragDrop.enable();

DragDrop.addHandler("dragstart", function(event) {
  var status = document.getElementById("status");
  status.innerHTML = "Started dragging " + event.target.id;
});

DragDrop.addHandler("drag", function(event) {
  var status = document.getElementById("status");
  status.innerHTML += "<br>Dragged " + event.target.id + " to (" + event.x + "," + event.y + ")";
});

DragDrop.addHandler("dragend", function(event) {
  var status = document.getElementById("status");
  status.innerHTML += "<br>Dropped " + event.target.id + " at (" + event.x + "," + event.y + ")";
});
```

运行结果：![](http://oczira72b.bkt.clouddn.com/17-11-21/80824560.jpg)

参考资料：

* [从一道面试题的进阶，到“我可能看了假源码”](https://zhuanlan.zhihu.com/p/25379434)
* [Javascript中bind()方法的使用与实现](https://segmentfault.com/a/1190000002662251)
* [闭包和高阶函数](https://github.com/xingbofeng/JavaScript-design-patterns/blob/master/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0/ch3-%E9%97%AD%E5%8C%85%E5%92%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0.md)
* [聊聊柯里化](https://segmentfault.com/a/1190000011825142)
* [JavaScript专题之函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42)
* [发布-订阅模式(观察者模式)](https://github.com/xingbofeng/JavaScript-design-patterns/blob/master/ch8-%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F/ch8-%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F.md)