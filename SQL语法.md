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

### 语法

> 以下语句均在MySQL8.0下测试通过

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

* as表示别名，多用于表示计算/运行函数生成的中间结果字段。

### group





参考链接

> https://blog.csdn.net/ddupd/article/details/15377981
>
> https://blog.csdn.net/Mrqiang9001/article/details/100074408
>
> https://www.cnblogs.com/chywx/p/10154990.html
>
> https://www.cnblogs.com/luxd/p/9916677.html (常用函数表)





