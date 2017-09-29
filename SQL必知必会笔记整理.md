近期主要是复习了基础的`SQL`，之前在大学课堂上听的基本上都是基本的增删查改的语法和基本的`SQL`概念，刷完这本书查漏补缺。之后计划啃`高性能MySQL`。

## 检索数据
* 使用`distinct`关键字，只返回不同的值

```sql
select distinct name from table;
```

* 使用`limit`关键字限制结果，不仅传两个参，即简化版的`limit`，也可以使用`offset`

```sql
select name from table limit 4 offset 3;
```

或者

```sql
select name from table limit 3, 4;
```

## 排序检索数据
* 在`SQL`中，有些子句是必需的，有些则是可选的。在指定一条`order by`子句时，应该保证它是`select`语句的最后一条子句。如果不是最后的子句，将会出现错误信息

* `order by`子句使用的列将是为显示而选择的列。

* 多个列排序，传两个参：

```sql
select id, price, name from table order by price, name;
```

* 此外还可以按照列的位置来排序：

```sql
select id, price, name from table order by 2, 3;
```

* `desc`关键字反向排序（降序，从Z到A，从大到小）

```sql
select id, price, name from table order by price desc;
```

* 如果多个列排序，只需要部分降序，只需要在对应字段名后使用`desc`即可。

```sql
select id, price, name from table order by price desc, name;
```

* 其实默认的升序排序是`asc`，不过通常被省略

## 过滤数据
过滤数据通常是使用`where`，但铭记一个原则，尽量不要在应用层去过滤数据。

* 空值检查（`NULL`）

```sql
select name from table where price is null;
```

* 判断在某个值之间，用`between`

```sql
select name, price from table where price between 5 and 10;
```

## 高级数据过滤
* 注意`and`操作符比`or`操作符优先级更高。为避免歧义，建议都要用圆括号
* `in`操作符的运用及其优势

1. 在有很多合法选项时，`in`操作符的语法更清楚直观。
2. 在与其他`and`和`or`操作符组合使用`in`时，求值顺序更容易管理。
3. `in`操作符一般比一组`or`操作符执行得更快。
4. `in`的最大优点是可以包含其他`select`语句，能更动态建立`where`语句。

```sql
select name, price from table where id in ('DLL01', 'BRS01') order by name;
```

* `not`操作符否定其后跟的任何条件（如使用包含类似于`not in`的子句）

## 用通配符进行过滤
* 百分号（`%`）通配符，匹配任意次数(`0~n次`，特别注意可以匹配0个字符)

```sql
select id, name from table where name like 'Fish%';
```

* 下划线(`_`)通配符，它只能够匹配单个字符，即有且仅有一个字符

```sql
select id, name from table where name like '__ inch teddy bear';
```

* 方括号(`[]`)通配符用来指定一个字符集，它必须匹配指定位置（通配符的位置）的在方括号范围内的一个字符。

```sql
select contact from table where contact like '[JM]%';
```

> 但是注意，虽然在SQL中的通配符是很有用的，但是这种功能是有代价的，即通配符的搜索一般会耗费更长的时间，因此不要过度使用通配符，即使确实遇到了要使用通配符的时候，也尽量不要把它们用在搜索模式的开始处。

## 创建计算字段



```sql

```