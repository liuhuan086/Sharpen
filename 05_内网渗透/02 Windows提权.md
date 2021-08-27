# 组策略首选项

Windows 2008 Server引入了一项称为组策略首选项的新功能，该功能使管理员可以部署影响域中计算机/用户的特定配置。通过在组策略管理控制台中配置的组策略首选项，管理员可以推出多种策略，例如，当用户登录其计算机时自动映射网络驱动器，更新内置管理员帐户的用户名或对注册表进行更改。



# 错误权限配置

系统服务文件在操作系统启动时加载执行，并且在后台调用可执行文件。

比如，对于Java升级程序，每次重启操作系统，Java升级程序会检测是否有最新版本Java程序，系统服务程序始终加载，通常运行在高权限本地账户。如果低权限用户对于服务调用的可执行文件具有可写权限，那么可以将其替换成我们创建的文件，这样创建的文件每次都随着操作系统启动服务而获得执行权限。



## Windows

Windows下的权限查看命令是icacls，具体相关用法和参数查看[**icacls命令官方文档**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)。

```\
C:\Users\huan\Desktop\LayerDomainFinder-3>icacls README.md
README.md Everyone:(F)
          NT AUTHORITY\SYSTEM:(I)(F)
          BUILTIN\Administrators:(I)(F)
          DESKTOP-7UETL55\huan:(I)(F)
```

* **F**：Full Control，类似于Linux中的777
* **M** - Modify，修改访问权限
* **RX** - 读取和执行访问
* **R** - 只读访问
* **W** - 只写访问

可以看到这里`Everyone`也是**F**权限，即所有用户都具有全部权限来对这个文件进行增删改查，那么如果我将该文件替换成其他可执行文件，就能悄悄地提升为系统权限。

### 示例

首先查看当前所有用户名

```
C:\Users\huan>net user
\\DESKTOP-7UETL55 的用户帐户
-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
huan                     WDAGUtilityAccount
```

然后我添加一个脚本

```
net user test 1122 /add
net localgroup administrators test /add
```

因为目前没有靶机有类似于Java需要管理员权限在重启后自动加载的服务，为了演示方便，这里是用Windows计划任务来实现，当重启时自动触发。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824175652.png)

当我关机重启后，可以看到这里成功添加了用户。



### 本地利用模块

Metasploit下的一个模块——Windows Service Trusted Path Privilege Escalation本地利用模块。

这个漏洞利用原理是查找系统服务文件中存在非引用路径。比如一个服务调用某个可执行文件，因为没有使用绝对路径，导致程序在执行时产生歧义，因此产生漏洞。

比如这里我应该执行的是`C:\test\target path`下的`demo.exe`，但是因为在`C:\test`下还存在一个`target.exe`的恶意程序。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824183225.png)

下面是系统调用伪代码

```
system(C:\test\target path\demo.exe)
```

在Windows系统下的终端里面，可执行程序加不加`.exe`是等效的，下图以powershell做演示。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824183915.png)

也就是说`target`和`target.exe`是等效的，因此，如果存在漏洞，这里就会将`C:\test\target`进行调用。

可以看到我这里在终端执行时提示了报错，可能是因为现在使用的Windows 10操作系统已经将漏洞修复了，但是验证了我们前面的说法，这里是将`target.exe`进行了解析，只是因为漏洞被修复没有执行成功。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824184826.png)

漏洞修复以前，`target.exe`能够被成功利用，且是添加管理员账户或者其他敏感操作，就会产生严重的后果。

