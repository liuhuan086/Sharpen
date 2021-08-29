# 简介

MySQL的内置函数虽然丰富，但毕竟不能满足所有人的需要，有时候我们需要对表中的数据进行一些处理而内置函数不能满足需要的时候，就需要对MySQL进行一些扩展。幸运的是，MySQL给使用者提供了添加新函数的机制，这种使用者自行添加的MySQL函数就称为**UDF(User Define Function)**。其实除了UDF外，使用者还可以将函数添加为MySQL的固有（内建）函数，固有函数被编译进mysqld服务器中，使其永久可用。

**查看组件安装位置**

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828220059.png)

> 题外话：[phpadmin忘记密码后重置密码](https://m.php.cn/tool/phpmyadmin/441404.html)，重置之后使用`net start mysql`命令重启MySQL。



# UDF的特性

* 可以自定义函数功能
* 可以自定义函数返回值



# 相关语法

**创建UDF**

```sql
CREATE FUNCTION function_name(parameter_nametype,[parameter_name type,...])
RETURNS {STRING|INTEGER|REAL};
```

上面的意思是创建一个聚合或者是非聚合的函数，sql中调用的名称是function_name，返回类型是{STRING|INTEGER|REAL}其中的一种。

```sql
CREATE [AGGREGATE] FUNCTION function_name RETURNS {STRING|INTEGER|REAL} SONAME 'shared_library_name';
```

上面除了创建函数外，还生成动态链接库，shared_library_name是生成的动态链接库的名称。

创建成功后会从MySQL服务器的mysql数据库的func表中找到对应的一条记录的添加。

**查看UDF**

```sql
SHOW CREATE FUNCTION function_name;
```

**删除UDF**

```sql
DROP FUNCTION function_name;
```

