近期主要是复习了基础的`SQL`，之前在大学课堂上听的基本上都是基本的增删查改的语法和基本的`SQL`概念，刷完这本书查漏补缺。之后计划啃`高性能MySQL`。

## 检索数据
* 使用`distinct`关键字，只返回不同的值

```sql
select distinct name from table1;
```

* 使用`limit`关键字限制结果，不仅传两个参，即简化版的`limit`，也可以使用`offset`

```sql
select name from table1 limit 4 offset 3;
```

或者

```sql
select name from table1 limit 3, 4;
```

## 排序检索数据
* 在`SQL`中，有些子句是必需的，有些则是可选的。在指定一条`order by`子句时，应该保证它是`select`语句的最后一条子句。如果不是最后的子句，将会出现错误信息

* `order by`子句使用的列将是为显示而选择的列。

* 多个列排序，传两个参：

```sql
select id, price, name from table1 order by price, name;
```

* 此外还可以按照列的位置来排序：

```sql
select id, price, name from table1 order by 2, 3;
```

* `desc`关键字反向排序（降序，从Z到A，从大到小）

```sql
select id, price, name from table1 order by price desc;
```

* 如果多个列排序，只需要部分降序，只需要在对应字段名后使用`desc`即可。

```sql
select id, price, name from table1 order by price desc, name;
```

* 其实默认的升序排序是`asc`，不过通常被省略

## 过滤数据
过滤数据通常是使用`where`，但铭记一个原则，尽量不要在应用层去过滤数据。

* 空值检查（`NULL`）

```sql
select name from table1 where price is null;
```

* 判断在某个值之间，用`between`

```sql
select name, price from table1 where price between 5 and 10;
```

## 高级数据过滤
* 注意`and`操作符比`or`操作符优先级更高。为避免歧义，建议都要用圆括号
* `in`操作符的运用及其优势

1. 在有很多合法选项时，`in`操作符的语法更清楚直观。
2. 在与其他`and`和`or`操作符组合使用`in`时，求值顺序更容易管理。
3. `in`操作符一般比一组`or`操作符执行得更快。
4. `in`的最大优点是可以包含其他`select`语句，能更动态建立`where`语句。

```sql
select name, price from table1 where id in ('DLL01', 'BRS01') order by name;
```

* `not`操作符否定其后跟的任何条件（如使用包含类似于`not in`的子句）

## 用通配符进行过滤
* 百分号（`%`）通配符，匹配任意次数(`0~n次`，特别注意可以匹配0个字符)

```sql
select id, name from table1 where name like 'Fish%';
```

* 下划线(`_`)通配符，它只能够匹配单个字符，即有且仅有一个字符

```sql
select id, name from table1 where name like '__ inch teddy bear';
```

* 方括号(`[]`)通配符用来指定一个字符集，它必须匹配指定位置（通配符的位置）的在方括号范围内的一个字符。

```sql
select contact from table1 where contact like '[JM]%';
```

> 但是注意，虽然在SQL中的通配符是很有用的，但是这种功能是有代价的，即通配符的搜索一般会耗费更长的时间，因此不要过度使用通配符，即使确实遇到了要使用通配符的时候，也尽量不要把它们用在搜索模式的开始处。

## 创建计算字段
* `MySQL`使用`Concat`拼接字段

```sql
select concat(name, ' (', country, ')') from table1 order by name;
```

* 使用`rtrim()`、`ltrim()`和`trim()`去掉右边、左边和左右两边的空格

```sql
select rtrim(name) from table1 order by name;
```

## 常用的函数
* `avg()`返回平均值
* `count()`返回表中行的数目或者符合特定条件的行的数目

```sql
select count(*) as num from table1;
```

* `max()`返回最大值
* `min()`返回最小值
* `sum()`用于求和

* 如果是对所有行执行计算，指定`all`参数或者不指定参数，如果需要聚合不同值，则指定`distinct`关键字，但注意：`distinct`关键字不能用于`count(*)`，必须指定列名

比如下面的代码，就是错误的：

```sql
select count(distinct *) as num from table1;
```

* 聚集函数还可以组合：

```sql
select count(*) as num, min(price) as price_min, max(price) as price_max, avg(price) as price_avg from table1;
```

## 分组数据
* 分组是使用`select`语句的`group by`子句建立的

```sql
select id, count(*) as num from table1 group by id;
```

上面的`select`语句指定了两个列,`group by`子句则会指示按照`id`来排序并分组数据。

* `having`用于过滤分组，`where`用于过滤行。特别注意的是，`where`在分组前进行过滤，而`having`在分组后进行过滤

```sql
select id, count(*) as orders from table1 group by id having count(*) >= 2;
```

## select子句顺序（优先级）
1. `select`
2. `from`
3. `where`
4. `group by`
5. `having`
6. `order by`

## 使用子查询
* 注意一点是子查询不适合太多，否则对性能会产生不好的影响

## 联结
联结常用于代替子查询的使用

* 内联结
* 等值联结
* 自联结
* 自然联结
* 外联结

使用联结的要点：

* 注意所使用的联结类型，一般情况使用内联结
* 保证使用正确的联结条件
* 总应提供联结条件，否则会出现笛卡尔积

## 组合查询
* 使用`union`关键字创建组合查询，`union`关键字创建组合查询时，默认去掉了重复的行，使用`union all`可以返回匹配的所有行。

```sql
select name, concat, email from table1 where state in ('IL', 'IN', 'MI');
  union all
select name, concat, email from table1 where name = 'Fun4All';
```

* 一般来说`union`几乎总是可以完成与多个`where`条件相同的工作。`union all`是`union`的一种形式，它完成`where`子句完成不了的工作。如果确实需要每个条件的匹配行全部出现（包括重复的行），就必须使用`union all`，而不是`where`

* 对组合查询的结果排序时，只需要在最后加`order by`，虽然`order by`似乎只是最后一条`select`语句的组成部分，但实际上它排序了`union`后的所有结果

## 插入数据
* 不指定字段直接插入数据并不安全，应该尽量避免使用。上面的`SQL`语句高度依赖于表中列的定义次序，还依赖于其容易获得的次序信息。即使可以得到这种次序信息，也不能保证各列在下一次表结构变动后保持完全相同的次序。
* `insert`语句可以省略数据，省略的列只能为指定了可以为空值的字段或者设定了默认值的字段
* 还可以插入检索出的数据

```sql
insert into table1 (id, contact, email, name) select id, contact, email, name from table2;
```

* 从一个表复制另外一个表的内容，对于非`MySQL`可以使用`select into`语句

```sql
select * into table1 from table2;
```

* 而对于`MySQL`，则只能够用如下方法：

```sql
create table table1 as select * from table2;
```

## 更新和删除数据
* 一定不能省略过滤条件，最好先写`where`子句
* 删除表中所有行，则不写`where`子句即可，但可以使用`truncate table`子句，这些方法都不能删除表，而`drop table`才是删除表

```sql
truncate table table1;
```

## 创建和操纵表
* 使用图形化界面或者`sql`语句来创建

```sql
create table table1
(
  id char(10) not null,
  name char(254) not null,
  explain varchar(1000) null,
)
```

* 创建表的时候，没有指定`not null`，就表示是`null`
* 更新表用`alter table`

```sql
alter table table1 add phone char(20);
```

```sql
alter table table1 drop column phone;
```

## 视图
视图是虚拟的表，视图只包含使用时动态检索数据的查询

关于为什么需要视图：

* 简化复杂的`sql`操作，在编写查询后，可以方便地重用它而不必知道其基本查询细节
* 使用表的一部分而不是整个表
* 保护数据，可以授予用户访问表的特定部分的权限，而不是整个表的访问权限
* 更改数据格式和表示。视图可以返回与底层表的表示和格式不同的数据

注意，虽然可以创建的视图数目没有限制，但是大量嵌套视图会影响性能，尽量不要嵌套太多层级。

* 使用视图可以简化复杂的联结：

```sql
create view myView as
  select name, contact, id
from table1, table2, table3
  where table1.id = table2.id
    and table2.num = table3.num;
```

如下调用视图：

```sql
select * from myView;
```

此外，还可以用视图做下列事情：

* 用视图重新格式化检索出的数据
* 用视图过滤不想要的数据
* 使用视图和计算字段

## 存储过程
其实可以在后端语言完成，也可以使用存储过程：

* 使用`execute`执行存储过程：

```sql
execute addNewProduct('JTS01', 'Stuffed Eiffel Tower', 6.49);
```

## 事务
一般来说，事务是必须满足4个条件：`Atomicity（原子性）`、`Consistency（稳定性）`、`Isolation（隔离性）`、`Durability（可靠性）`

* 事务的原子性：一组事务，要么成功；要么撤回。
* 稳定性 ：有非法数据（外键约束之类），事务撤回。
* 隔离性：事务独立运行。一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。事务的100%隔离，需要牺牲速度。
* 可靠性：软、硬件崩溃后，数据表驱动会利用日志文件重构修改。

使用`begin（开始事务）`，`rollback（回退）`，`commit（提交）`，`savepoint（保留点）`关键字实现事务。