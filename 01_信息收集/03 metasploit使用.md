## metasploit

在kali中执行`msfconsole`即可使用。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210815145555.png)

一般通过前期的信息搜集，知道目标使用的端口、服务、版本及漏洞信息等，使用msf对其漏洞进行利用。具体可参考[ms17-010渗透测试操作步骤](https://www.cnblogs.com/liuhuan086/p/13068752.html)。



## msf模块

```bash
┌──(huan㉿kali)-[/usr/share/metasploit-framework/modules]
└─$ ll
total 28
drwxr-xr-x 22 root root 4096 Aug 15 01:51 auxiliary
drwxr-xr-x 12 root root 4096 Aug 15 01:51 encoders
drwxr-xr-x  3 root root 4096 Aug 15 01:51 evasion
drwxr-xr-x 22 root root 4096 Aug 15 01:51 exploits
drwxr-xr-x 11 root root 4096 Aug 15 01:51 nops
drwxr-xr-x  5 root root 4096 Aug 15 01:51 payloads
drwxr-xr-x 14 root root 4096 Aug 15 01:51 post
```

msf6中共有7个模块，其中

### auxiliary

负责信息收集、指纹识别、漏洞探测、口令破解等功能

### encoders

对payload进行加密，绕过杀软的功能模块。

### evasion

绕过杀软的功能模块。

### exploits

漏洞利用模块，对已经扫描和收集到的目标的漏洞进行利用（有可能会失败）的模块。

### nops

提高payload稳定性及维持大小，空指令相关。

### payloads

exploits执行成功后，在目标系统真正执行的代码或指令等操作。

### post

后期渗透模块，执行一些后渗透操作，比如内网扫描和攻击、登录其他服务器、数据库等资产。

## msf简单使用步骤

### 漏洞扫描

1. 使用search命令，搜索可以用的模块。这里可以看到Name前面分为`auxiliary`和`exploit`。

   ![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210815150226.png)

2. 使用`use`选择要使用的探测模块。

   ```
   msf6 > use auxiliary/scanner/smb/smb_ms17_010
   msf6 auxiliary(scanner/smb/smb_ms17_010) >
   ```

3. 使用`show options`查看并设置需要使用的参数，`Required`表示是必填参数，具体参数的描述查看Description。

   比如这里设置的`RHOSTS`表示目标IP地址

   ![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210815150622.png)

4. 其他还可以设置线程数等，`set threads 16`。

5. 使用`run`命令开始扫描。



### 漏洞利用

当我们发现目标存在相应的漏洞时，利用对应的模块即可进行攻击。

