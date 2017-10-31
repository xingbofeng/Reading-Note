# Schema与数据类型优化
## 选择优化的数据类型

`MySQL`支持的数据类型非常多，选择正确的数据类型对于获得高性能至关重要。不管存储哪种类型的数据，下面几个简单的原则都有助于做出更好的选择：

* 更小的通常更好，尽量使用可以正确存储数据的最小数据类型，更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期也更少

* 简单就好，简单数据类型的操作通常需要更少的CPU周期。例如，整型比字符操作代价更低，因为字符集和校对规则（排序规则）使字符比较比整型比较更复杂。使用`MySQL`内建的类型（`data`、`time`、`datatime`）而不是字符串来存储日期和时间，用整型存储IP地址
* 尽量避免`null`，最好指定列为`not null`，除非真的需要存储`null`值，通常数值型默认为`0`，字符型默认为`Empty String`。如果查询中包含可为`null`的列，对`MySQL`来说更难优化，因为可为`null`的列使得索引、索引统计和值比较都更复杂。可为`null`的列会使用更多的存储空间，在`MySQL`中也需要特殊处理。当可为`null`的列被索引时，每个索引记录需要一个额外的字节。通常把可为`null`的列改为`not null`带来的性能提升比较小，所以调优时没有必要首先在现有`schema`中查找并修改，除非确定这会导致问题，但是，如果计划在列上建索引时，就应该尽量避免设计成可为`null`的列
  * 使用count()统计某列记录数时，会忽略掉`null`值的记录，而默认值为`Empty String`的记录会被统计
  * 判断是否为`null`用`is null`或者`is not null`，判断是否为空字符串用`= ''`或者`<> ''`

```sql
select count(userphone) from tb_user; //不含字段值为null
select count(*) from tb_user where userphone = ''; //不含字段值为null
select count(userphone) from tb_user where userphone = ''; //不含字段值为null
select count(*) from tb_user where userphone <> ''; //不含字段值为null
select count(userphone) from tb_user where userphone <> ''; //不含字段值为null
select count(*) from tb_user where userphone is null; //仅含字段值为null
select count(userphone) from tb_user where userphone is null; //不含字段值为null、Empty String
select count(*) from tb_user where userphone is not null; //含字段值为Empty String
select count(userphone) from tb_user where userphone is not null; //含字段值为Empty String
```

当为列选择数据类型时，第一步需要选择合适的大类型：数字、字符串、时间等；第二部步是选择具体类型，很多`MySQL`的数据类型可以存储相同类型的数据，只是存储的长度和范围不一样、允许的精度不同，或者需要的物理空间（磁盘和内存空间）不同。
### 整数类型
有两种类型的数字：`整数`和`实数`。如果存储`整数`，可以使用这几种`整数`类型：`tinyint`、`smallint`、`mediumint`、`int`、`bigint`，分别使用：`8`、`16`、`24`、`32`、`64`位存储空间，它们可以存储值的范围从`-2^(N-1)`到`(2^(N-1))-1`，其中`N`是存储空间的位数。

`整数`类型有可选的`unsigned`属性，表示不允许负值，这大致可以使正数的上限提高一倍，例如无符号型`tinyint`可以存储的范围是0~255，而`tinyint`的存储范围是-128~127。有符号和无符号类型使用相同的存储空间，并具有相同的性能，因此可以根据实际情况选择合适的类型。

`MySQL`可以为`整数`类型提供宽度，如`int(11)`，对大多数应用这是没有意义的：它不会限制值的合法范围，只是规定了`MySQL`的一些交互工具用来显示字符的个数。对于存储和计算来说，`int(1)`和`int(20)`相同。

### 实数类型

`实数`是带有小数部分的数字，然而，它们不只是为了存储小数部分，也可以使用`decimal`存储比`bigint`还大的`整数`。`MySQL`既支持精确类型，也支持不精确类型：
* `float`和`double`类型支持使用标准的浮点运算进行近似计算
* `decimal`类型用于存储精确的小数

`浮点`和`decimal`类型都可以指定精度，对于`decimal`列，可以指定小数点前后所允许的最大位数，这会影响列的空间消耗。`MySQL` 5.0和更高版本将数字打包保存到一个二进制字符串中（每4个字节存9个数字）。例如，`decimal(18,9)`小数点两边将各存储9个数字，一共使用9个字节：小数点前的数字用4个字节，小数点后的数字用4个字节，小数点本身占1个字节。`MySQL` 5.0和更高版本中的`decimal`类型允许最多65个数字。

`浮点`类型在存储同样范围的值时，通常比`decimal`使用更少的空间。`float`使用4个字节存储，`double`占用8个字节，相比`float`有更高的精度和更大的范围。和`整数`类型一样，能选择的只是存储类型；`MySQL`使用`double`作为内部`浮点`计算的类型。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用`decimal`——例如存储财务数据。但在数据量比较大的时候，可以考虑使用`bigint`代替`decimal`，将需要存储的货币单位根据小数的位数乘以相应的倍数即可，这样可以同时避免`浮点`存储计算不精确和`decimal`精确计算代价高的问题。

`MySQL`允许使用非标准语法：`float(m,d)`、`real(m,d)`或`double(m,d)`，这里`(m,d)`表示该值一共保存`m`位数字，其中`d`位数字在小数点后面。例如，定义为`float(7,4)`的列保存值的范围：`-999.9999`~`999.9999`，在实际保存值时会四舍五入，如果在`float(7,4)`列内插入`999.00009`，实际保存值`999.0001`。

`decimal`和`numeric`在`MySQL`中视为相同的类型，它们用于保存精确值，例如财务数据。当定义列为该类型时，可以指定精度和标度，例如，`decimal(5,2)`中`5`是精度，`2`是标度，精度表示可以保存数字的总位数，标度表示小数点后可以保存数字的位数。

### 字符串类型
`MySQL`支持多种字符串类型，每种类型还有很多变种。

#### varchar和char
`varchar`和`char`是两种最主要的字符串类型。

`varchar`类型用于存储可变长字符串，是最常见的字符串数据类型，它比定长类型更节省空间，因为它仅使用必要的空间（例如，越短的字符串使用越少的空间）。在`MySQL` 5.0或更高版本，存储和检索`varchar`时会保留末尾空格。`varchar`需要使用`1`或`2`个额外字节记录字符串的长度：如果列的最大长度小于或等于`255`字节，需额外使用`1`个字节，否则使用`2`个字节。

下面这些情况下使用`varchar`是合适的：
* 字符串列的最大长度比平均长度大很多
* 列的更新很少，所以碎片不是问题
* 使用了像`utf-8`这样复杂的字符集，每个字符都使用不同的字节数进行存储

`char`类型是定长的：`MySQL`总是根据定义的字符串长度分配足够的空间。当存储`char`值时，`MySQL`会删除所有的末尾空格。`char`适合存储很短的字符串，或者所有值都接近同一个长度：
* `char`非常适合存储密码的md5值，因为这是一个定长的值
* 对于经常变更的数据，`char`也比`varchar`更好，因为定长的`char`类型不容易产生碎片
* 对于非常短的列，`char`也比`varchar`在存储空间上也更有效率

数据如何存储取决于存储引擎，并非所有的存储引擎都会按照相同的方式处理定长和变长的字符串。

#### varbinary和binary

`varbinary`和`binary`类型存储的是二进制字符串。二进制字符串和常规字符串非常相似，但是二进制字符串存储的是字节码而不是字符，填充也不一样：`MySQL`填充`binary`采用的是`\0`（零字节）而不是空格，在检索时也不会去掉填充值。

当需要存储二进制数据，并且希望`MySQL`使用字节码而不是字符进行比较时，这些类型是非常有用的。二进制比较的优势并不仅仅体现在大小写敏感上。`MySQL`比较`binary`字符串时，每次按一个字节，并且根据该字节的数值进行比较。因此，二进制比较比字符比较简单很多，所以也就更快。

#### blob和text

`blob`和`text`都是为存储很大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储。实际上，它们分别属于两组不同的数据类型家族：字符类型是`tinytext`、`text`、`mediumtext`、`longtext`；对应的二进制类型是`tinyblob`、`blob`、`mediumblob`、`longblob`。

与其它类型不同，`MySQL`把每个`blob`和`text`值当作一个独立的对象处理。存储引擎在存储时通常会做特殊处理。当`blob`和`text`值太大时，InnoDB会使用专门的“外部”存储区域来进行存储，此时每个值在行内需要`1`~`4`个字节存储一个指针，然后在外部存储区域存储实际的值。

`blob`和`text`家族之间仅有的不同是`blob`类型存储的是二进制数据，没有排序规则或字符集，而`text`类型有字符集和排序规则。

`MySQL`对`blob`和`text`列进行排序与其他类型是不同的：它只对每个列的最前`max_sort_length`字节而不是整个字符串做排序。

`MySQL`不能将`blob`和`text`列全部长度的字符串进行索引，也不能使用这些索引消除排序。

### 日期和时间类型

`MySQL`能存储的最小时间粒度为秒，提供两种相似的日期时间类型：`datetime`和`timestamp`，提供日期类型：`date`，提供时间类型：`time`。

`datetime`类型能保存大范围的值，从`1001`年到`9999`年，精度为妙，使用`8`个字节的存储空间。

`timestamp`类型保存了从`1970年1月1日午夜（格林尼治标准时间）以来的秒数`，它和UNIX时间戳相同，使用`4`个字节的存储空间，因此它的范围比`datetime`小的多：只能表示从`1970`年到`2038`年。`timestamp`显示的值也依赖于时区，`MySQL`服务器、操作系统，以及客户端连接都有时区设置。

### 选择标识符

为`标识列（identifier column）`选择合适的数据类型非常重要。一旦选定了一种类型，要确保在所有关联表中都使用同样的类型，类型之间需要精确匹配。在可以满足值的范围的需求，并且预留未来增长空间的前提下，应该选择最小的数据类型:
* `整数`通常是标识列最好的选择，因为它们很快并且可以使用`AUTO_INCREMENT`
* 尽量避免使用字符串类型作为`标识符`，因为它们很消耗空间，并且通常比数字类型慢。对于完全随机的字符串也需要多加注意，随机产生的值会任意分布在很大的空间内，这会导致`insert`以及一些`select`语句变的很慢

### 特殊类型数据

经常使用`varchar(15)`列来存储`IP`地址，然而，它们实际上是`32`位`无符号整数`，不是字符串，用小数点将地址分成四段的表示方法只是为了阅读。所以应该用`无符号整数`存储`IP`地址，`MySQL`提供`INET_ATON()`、`INET_NTOA()`函数在这两种表示方法之间转换，示例如下。

```sql
`ip` int(10) unsigned default '0';

update tb_test set ip = INET_ATON('192.168.1.1');

select INET_NTOA(ip) from tb_test;
```

## schema设计中的陷阱

`schema`等价于数据库，如下所示：

```sql
mysql> select SCHEMA_NAME from information_schema.SCHEMATA;
+-----------------------+
| SCHEMA_NAME           |
+-----------------------+
| information_schema    |
| blogstation           |
| cloudhospital         |
| cncounter             |
| freshmommybbqjl       |
| freshmommyhhzce       |
| freshmommywy          |
| hibernate             |
| hospital              |
| inforbase             |
| jeesite               |
| mobilemedicalplatform |
| mybatis               |
| mydb                  |
| MySQL                 |
| pciplatform           |
| performance_schema    |
| questiondatabase      |
| redis                 |
| remotecooperate       |
| sharelife             |
| sonar                 |
| sourceshareplat       |
| springsession         |
| test                  |
| videoplatform         |
| wonders_bi            |
+-----------------------+
27 rows in set (0.00 sec)
```

在`MySQL`特定实现下，设计`schema`时需要避免的错误：

* 太多的列，`MySQL`的存储引擎API工作时需要在`服务器层`和`存储引擎层`之间通过`行缓冲`格式拷贝数据，然后在`服务器层`将缓冲内容解码成各个列。从`行缓冲`中将编码过的列转换成`行数据结构`的操作代价是非常高的。InnoDB的行结构总是需要转换，转换的代价依赖于列的数量，如果计划使用数千个字段，必须意识到服务器的性能运行特征会有一些不同
* 太多的关联，`MySQL`限制了每个关联操作最多只能有`61`张表。一个粗略的经验法则，如果希望查询执行的快速且并发性好，单个查询最好在`12`个表以内做关联
* 全能的枚举，注意防止过度使用`ENUM`
* 变相的枚举
* 非此发明的`null`

## 范式和反范式

对于任何给定的数据通常都有很多种表示方法，从完全的`范式化`到完全的`反范式化`，以及两者的折中。在`范式化`的数据库中，每个事实数据会出现并且只出现一次。相反，在`反范式化`的数据库中，信息是冗余的，可能会存储在多个地方。

### 范式的优缺点

因为性能问题而寻求帮助时，经常会被建议对`schema`进行`范式化`设计，尤其是写密集的场景：

* `范式化`的更新操作通常比`反范式化`要快
* 当数据较好的`范式化`时，就只有很少或者没有重复数据，所以只需要修改更少的数据
* `范式化`的表通常更小，可以更好的放在内存里，所以执行操作会更快
* 很少有多余的数据意味着检索列表数据时更少需要`DISTINCT`或者`GROUP BY`语句

`范式化`设计的`schema`的缺点是通常需要关联。稍微复杂一些的查询语句在符合`范式化`的`schema`上都可能需要至少一次关联，也许更多，这不但代价昂贵，也可能使一些索引策略无效。

### 反范式的优缺点

`反范式化`的`schema`因为所有数据都在一张表中，可以很好的避免关联。如果不需要关联表，则对大部分查询最差的情况——即使表没有使用索引——是全表扫描。当数据比内存大时这可能比关联要快的多，因为这样避免了随机I/O。

### 混用范式化和反范式化

事实是，完全的`范式化`和完全的`反范式化`的`schema`都是实验室里才有的东西，在实际应用中经常需要混用，可能使用部分`范式化`的`schema`、缓存表，以及其他技巧。

最常见的`反范式化`数据的方法是复制或者缓存，在不同的表中存储相同的特定列。在`MySQL` 5.0和更新版本中，可以使用`触发器`更新缓存值，这使得实现这样的方案变的更简单。

## 缓存表和汇总表
有时提升性能最好的方法是在同一张表中保存衍生的冗余数据；有时也需要创建一张完全独立的汇总表或缓存表。

缓存表表示存储那些可以比较简单地从`Schema`其他表获取数据的表。
汇总表表示保存的是使用`group by`语句聚合数据的表。

一个有用的技巧是对缓存表使用不同的存储引擎。例如：主表用`InnoDB`，使用`MyISAM` 作为缓存表的引擎将会得到更小的索引占用空间，并且可以做全文检索。

> 注意：全文检索还是使用专门的工具，比如 ElasticSearch 更好。

在使用缓存表和汇总表时，必须决定是实时维护数据还是定时重建。看需求。定时重建不仅节省资源，还保持表不会有很多碎片，以及完全顺序组织的索引（这会更加高效）。

当重建汇总表和缓存表时，使用**影子表**来保证数据在操作时依然可用。

```sql
drop table if exists my_summary_new, my_summary_old;
create table my_summary_new like my_summary;
rename table my_summary to my_summary_old, my_summary_new to my_summary;
```

### 物化视图
物化视图是预先计算并且存储在磁盘上的表，可以通过各种各样的策略刷新和更新。

`MySQL`并不原生支持物化视图。

`Justin Swanhart`的开源工具`Flexviews`，[Swanhart Toolkit](https://github.com/greenlion/swanhart-tools)。

### 计数器表
假设有一个计数器表，只有一行数据，记录网站的点击次数：

```sql
create table hit_counter(
  cnt int unsigned not null
) ENGINE = InnoDB;
```

问题在于，对于任何想要更新这一行的事务来说，这条记录上都有一个全局的互斥锁。这会使得这些事务只能串行执行。要获得更高的并发更新性能，也可以将计数器保存在多行中，每次随机选择一行进行更新。

```sql
create table hit_counter(
  slot tinyint unsigned not null primary key,
  cnt int unsigned not null
) ENGINE = InnoDB;
```

然后预先在这张表增加`100`行数据。现在选择一个随机的槽(`slot`)进行更新：

```sql
update hit_counter set cnt + 1 where slot = rand() * 100;
```

要获得统计结果，需要使用下面这样的聚合查询：

```sql
select sum(cnt) from hit_counter;
```

一个常见的需求是每隔一段时间开始一个新的计数器（例如，每天一个）。如果需要这么做，则可以再简单地修改一下表设计：

```sql
create table daily_hit_counter(
  day date not null,
  slot tinyint unsigned not null,
  cnt int unsigned not null,
  primary key(day, slot)
) ENGINE = InnoDB;
```

在这个场景中，可以不像前面的例子那样预先生成行，而用`on duplicate key update`（当插入重复值的时候更新该原行）代替：

```sql
insert into daily_hit_counter(day, slot, cnt)
  values(CURRENT_DATE, rand() * 100, 1)
  on duplicate key update cnt = cnt + 1;
```

如果希望减少表的行数，以避免表变得太大，可以写一个周期执行的任务，合并所有结果到0号槽，并且删除其它所有的槽：

```sql
update daily_hit_counter as c
  inner join(
    select day, sum(cnt) as cnt, min(slot) as mslot
    from daily_hit_counter
    group by day
  ) as x using(day)
set c.cnt = if(c.slot = x.mslot, x.cnt, 0),
    c.slot = if(c.slot = x.mslot, 0, c.slot);
```

## 加快ALTER TABLE操作的速度
`MySQL`的`alter table`操作的性能对于大表来说是个大问题。`MySQL`执行大部分修改表结构操作的方法是用新的结构创建一个空表，从旧表中查出所有数据插入新表，然后删除旧表。

一般而言，大部分`alter table`操作将导致`MySQL`服务中断。有两个技巧可以避免：

* 先在一台不提供服务的机器上执行`alter table`操作，然后和提供服务的主库进行切换；
* 影子拷贝：用要求的表结构创建一张和源表无关的新表，然后通过重命名和删表的操作交换两张表。还有一些第三方工具可以完成：
  * [online schema change](https://launchpad.net/mysqlatfacebook)
  * [openark toolkit](http://code.openark.org/blog/)
  * [Percona Toolkit](https://www.percona.com/software)

不是所有的`alter table`操作都会引起表重建。

```sql
-- 很慢，多次读和多次插入操作
alter table film
  modify column rental_duration tinyint(3) not null default 5;
-- 直接修改 _.frm_ 文件而不设计表数据。操作非常快。
alter table film
  alter column rental_duration set default 5;
```

> alter table 允许使用 alter column, modify column 和 change column 语句修改列。这三种操作都是不一样的。

* `modify column`会做大量的读写操作，慢
* `alter column`则不会，这个操作比较快

### 只修改 .frm 文件
下面的这些操作有可能不需要重建表：

* 移除一个列的`AUTO_INCREMENT`属性；
* 增加、移除，或更改 ENUM 和 SET 常量。

基本的技术是为想要的表结构创建一个新的`.frm`文件，然后用它替换掉已经存在的那张表的`.frm`文件。步骤如下：

1. 创建一张有相同结构的空表，并进行所需要的修改；
2. 执行 FLUSH TABLES WITH READ LOCK。这将会关闭所有正在使用的表，并且禁止任何表被打开；
3. 交换 .frm 文件；
4. 执行 UNLOCK TABLES 来释放第2步的读锁。

### 快速创建 MyISAM 索引
为了高效地载入数据到`MyISAM`表中，有一个常用的技巧是先禁用索引、载入数据，然后重新启用索引。

```sql
alter table load_data disable keys;
alter table load_data enable keys;
```

不过，这个办法对唯一索引无效，因为`disable keys`只对非唯一索引有效。

现代版本的`InnoDB`中有类似的技巧。

## 总结
* 尽量避免过度设计；
* 使用小而简单的合适数据类型，除非真的需要，否则应尽可能避免使用`NULL`；
* 尽量使用相同的数据类型存储相似或相关的值，尤其是要在关联条件中使用的列；
* 注意可变长字符串，其在临时表和排序时可能导致悲观的按最大长度分配内存；
* 尽量使用整型定义标识列；
* 避免使用`MySQL`已经遗弃的特性，例如指定浮点数的精度，或者整型的显示宽度；
* 小心使用`ENUM`和`SET`；
* 最好避免使用`BIT`。
