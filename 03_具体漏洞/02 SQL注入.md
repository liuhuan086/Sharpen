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

## 判断注入点

