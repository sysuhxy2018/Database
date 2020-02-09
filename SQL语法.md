# SQL语法

### MySQL数据类型

* int[(m)] [unsigned] [zerofill]。其中unsigned表示无符号（默认有符号）；m必须和zerofill搭配使用，规定显示的宽度，如果宽度小于m，则在左边补0，否则按原数值显示。
* float[(m, d)] [zerofill]。其中m规定总宽度（小数点不占位），d规定小数部分的位数。如果小数部分位数不够，则右边补0，超出则按四舍五入截取。m - d表示整数部分最多占有的位数，不能超过该宽度，否则会报错。m还可以搭配zerofill使用，如果整数位数不够的话会在左边补0（但测试发现总是会少补一个0）。
  * 注意浮点类型没有unsigned。
* decimal[(m, d)] [zerofill]。和浮点类型基本一致，但是整数部分补0数目是正确的，不会少一个。
  * 定点类型和浮点类型的区别是它存放和计算都用的精确值，而浮点类型用的近似值。
  * 无论是定点还是浮点类型，实际存放的小数部分都是按照处理后的d位来，而不仅仅是显示宽度。
* char和varchar的区别：
  * char(n)存储定长字符串，<= n个字符。如果不够，则存放时在末尾补空格，但查询时会删去末尾的空格。所以一般char(n)字符串末尾不能有空格；varchar(n)存储变长字符串，<= n个字符，不够就按实际长度算，但是需要额外字节数来记录长度(n <= 255需1B，> 255需2B)。
    * 注意这里n表示的是字符数，而不是字节数，实际占用字节数还与字符编码、文字（字母/汉字）有关。
  * char检索时性能优于varchar，varchar优势在于空间消耗会更少。
* timestamp和datetime的区别：
  * timestamp占4个字节，datetime8个。显示格式都是YYYY-MM-dd HH:mm:ss。
  * timestamp插入时默认值为当前时间戳，而datetime为null。不过都可以手动设置自动更新和初始化。
  * timestamp按utc时区存储并自动按照设置的时区转换；datetime按照插入/更新时的时区显示，如果查询时时区变了，不会自动显示成查询的时区。
  * 由于timestamp只有4个字节，所以支持范围仅为1970 ~ 2038年；而datetime为1000 ~ 9999年。

参考链接：

> https://www.iteye.com/blog/daizj-2248012
>
> https://www.cnblogs.com/-xlp/p/8617760.html
>
> https://blog.csdn.net/weixin_42570248/article/details/89786882
>
> https://www.cnblogs.com/liuxs13/p/9760812.html
>
> https://www.cnblogs.com/zhaoyanghoo/p/5581710.html

### 创建/删除数据库

``` mysql
create database mydb;
use mydb;	# 切换不同db使用
drop database mydb;
```

### 创建/修改/删除表

``` mysql
create table mytable (
	id int not null auto_increment,
    col1 int not null default 1,
    col2 varchar(45) null,
    col3 date null,
    primary key (id)
);

alter table mytable add col4 char(20);
alter table mytable drop column col2;	# drop可能是列或者表，所以需要名词来区分

drop table mytable; 
```

这里属于偷懒的写法，如果字段名字带空格就不能这样写了。需要反引号``(esc下面那个键）用来包围字段，另外单引号''，以及双引号""的意义一样，用来包围字符串。

### 增删改查(CRUD)

``` mysql
insert into mytable(col1, col2) values(5, "hello");
# 要求col_a，col_b数据类型必须和col1, col2兼容。如col1是varchar(5), col_a就不能是varchar(6)
insert into mytable(col1, col2) select col_a, col_b from mytable2;	
create table mytable3 as select col1, col2 from mytable;

update mytable set col1 = 10 where id = 1;

delete from mytable where id = 1; # 删除是一行行删的，所以不需要指定列
truncate table mytable;	# 清空所有记录

select distinct col1, col2 from mytable; # 不会出现相同的col1，col2组合
select * from mytable limit 1, 2; # 在所有满足条件的记录中返回第2 ~ 3行
```

* MySQL默认在安全模式下delete的where限定中必须出现主键（或者外键），即不能where col1 = 10。
* limit a, b表示取(a, a + b]区间记录。

### order

``` mysql
select * from mytable order by col1 desc, col2 asc; # 按col1降序排列，如果col1值相同，则按col2升序排列
```

* 如果不指定顺序，则默认按升序。

### where

``` mysql
select * from mytable where col2 <> "hello";
select * from mytable where col1 between 2 and 4; # between ... and表示一个区间，包括端点
select * from mytable where col2 is not null; # null判断用is/is not
select col2 from mytable where col1 in (select distinct col_a from mytable2 where col_b = "hello7");
select * from mytable where col2 like "h_%"; # 可匹配"hello"
select * from mytable where col2 regexp "^[^hw].*"; #匹配不以h和w开头的
```

* 相等用 = ，不等用 <>。对于字符串等类型比较也适用。
* in的范围内出现null不影响；但是not int的范围内不能出现null，否则会让查询结果为空集。
* and优先级比or高，但可以用括号限定条件优先级。
* like可用于字符串类型的匹配，但只能用到%和_两种特殊字符。一般更高级的匹配用regexp正则。

### as

``` mysql
select col1 * id as res from mytable;
select concat("print: ", trim(col2)) as res from mytable;
select length(left(col2, 2)) + 5 as res from mytable; # 函数可以嵌套，可以参与运算
```

* as表示别名，多用于表示计算/运行函数生成的中间结果字段。也可以给列名、表名取别名。

### group

``` mysql
select col1, count(*) as num from mytable group by col1 order by num;
select col1, col2, count(*) as num from mytable group by col1, col2;
select col1, count(*) as num from mytable group by col1 having mod(sum(id), 2) = 1;
```

* group by可以汇总相同数据的行到同一个组中，对每个分组可以应用聚合函数统计信息，如count, avg, sum等。group by分组的依据可以是多个字段，表示相同的col1, col2组合。
* having相当于过滤group by产生的分组，也可以用聚合函数。
* 执行顺序：where -> group by -> having -> order by -> limit。
  * 除了汇总字段，select中的字段必须都出现在group by字段中。MySQL中，如果有一个不在，比如col3，那么就会取分组中某行的col3数据来代表整个分组的col3。
  * where，having选择的字段和前面select和group by的字段都没关系，它们只是表示过滤。
  * order by排序的字段一般从select字段里选择，表示对最终结果集的排序。

### 子查询

``` mysql
select * from emp where dep_id in (select id from dept);
select * from emp where salary < any (select salary from emp where id in(2,5));
select * from emp where salary < all (select salary from emp where id in(2,5));
select * from emp where exists (select id from dept where dept.id = emp.dep_id);

select cust_name, (select count(*) from Orders where Orders.cust_id = Customers.cust_id) as orders_num from Customers order by cust_name;

select * from (select goods_id,cat_id,goods_name from goods order by cat_id asc,goods_id desc) as tmp group by cat_id;
```

子查询只返回一个字段的数据，即可以作为一列或者一个具体的数值。应用场景：

* where，可以和in, any, all, exists等组合。
  * in表示一个集合；如果是用 = 这样的比较运算符，= 另一边应该是一个具体的数值而不是集合。
  * < any只要满足小于集合中任意一个值即可，即小于最大值
  * < all要满足小于集合中所有值，即小于最小值
  * exist相当于用for遍历每一行，用每一行的数据代入子查询检查结果，如果结果不为空，则保留该记录。
* select，作为计算字段
* from，相当于把子查询的结果集当成一张表（视图）

### 连接

``` mysql
select * from emp as a inner join dept as b on a.dept_id = b.id;
select * from emp a left join dept b on a.dept_id = b.id; 
select * from emp a right join dept b on a.dept_id = b.id;

select * from emp as a left join dept as b on a.dept_id = b.id
union select * from emp as a right join dept as b on a.dept_id = b.id;
```

连接用于多表联合查询，可以替代子查询且效率更高，更简洁。

种类：

* 内连接，一个表和自身的内连接又叫自连接。
* 外连接，有左外连接，右外连接，全外连接。
  * 左外连接包含左表的全部记录（自然也包含自连接），对于不在左表中的右表字段，用null代替。右外连接同理。
  * MySQL中没有全外连接的关键字，可以通过左外连接和右外连接union/union all实现；union会自动去重，union all保留重复记录。union也可以单独用于合并两个查询，但要求两个查询select出完全相同的字段。
* 其他还有自然连接(natural join)和交叉连接(cross join)等。

内外连接通常是基于等值的比较关系(=)，其实也可以是其他比较关系。所有的连接都是基于cross join再筛选，也就是左表记录和右表记录的笛卡尔积，如果左表M条，右表N条，则cross join结果为M * N条。join只是表示连接两个表，后面我们还是可以用where，group by，having等继续过滤、分组。

实际上一些连接可以用where + 子查询代替。

### 补充

后续可以补充一些关于视图，trigger，函数和事务等方面的内容。



参考链接

> https://blog.csdn.net/ddupd/article/details/15377981
>
> https://blog.csdn.net/Mrqiang9001/article/details/100074408
>
> https://www.cnblogs.com/chywx/p/10154990.html
>
> https://www.cnblogs.com/luxd/p/9916677.html (常用函数表)
>
> https://www.cnblogs.com/niyl/p/9650183.html
>
> https://blog.csdn.net/sxlzs_/article/details/79396979
>
> https://blog.csdn.net/m0_38061639/article/details/82872705
>
> https://www.cnblogs.com/mryangbo/p/10904197.html
>
> https://www.jianshu.com/p/620eef505bd8
>
> https://www.cnblogs.com/xiaozhaoboke/p/11077781.html
>
> https://blog.csdn.net/zjt980452483/article/details/82945663





