# metasploit

在kali中执行`msfconsole`即可使用。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210815145555.png)

一般通过前期的信息搜集，知道目标使用的端口、服务、版本及漏洞信息等，使用msf对其漏洞进行利用。具体可参考[ms17-010渗透测试操作步骤](https://www.cnblogs.com/liuhuan086/p/13068752.html)。



# Nmap与Metasploit

Metasploit可以将渗透测试信息存入数据库中并可以共享。

每次msfconsole启动时，会自动连接到数据库，使用`db_status`可以查看与数据库连接状态。

```
msf6 > db_status 
[*] postgresql selected, no connection
```

在kali中新开终端查看postgresql服务状态，首次使用postgresql需要启动服务，并建议新建用户。

```bash
$ service postgresql status

● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: inactive (dead)

$ sudo service start postgresql

$ service postgresql status    
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: active (exited) since Mon 2021-08-16 05:49:15 EDT; 10min ago
    Process: 9017 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9017 (code=exited, status=0/SUCCESS)
        CPU: 1ms

# 登录
$ sudo -u postgres psql
psql (13.2 (Debian 13.2-1))
Type "help" for help.

postgres=# CREATE USER admin WITH PASSWORD '1122';
CREATE ROLE
postgres=# CREATE DATABASE msf OWNER admin;
CREATE DATABASE
postgres=# 
```

```
msf6 > db_connect admin:1122@127.0.0.1/msf

msf6 > db_status
[*] Connected to msf. Connection type: postgresql. Connection name: OrKJCtgy.
```

可以使用`db_connect`来连接数据库，因此也可以连接其他数据库查看漏洞信息。

```
msf6 > db_connect admin:1122@127.0.0.1/msf
Connected to Postgres data service: 127.0.0.1/msf
```

nmap很好的与metasploit集成在一起，可以方便的在Metasploit终端中使用`db_map`。

```
msf6 > db_nmap
[*] Usage: db_nmap [--save | [--help | -h]] [nmap options]
```



# msf模块

msf6中共有7个模块，在`/usr/share/metasploit-framework/modules`目录下

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

## auxiliary

负责信息收集、指纹识别、漏洞探测、口令破解等功能

```
┌──(huan?kali)-[/usr/…/modules/auxiliary/scanner/discovery]
└─$ ll
total 56
-rw-r--r-- 1 root root  3365 Feb 11  2021 arp_sweep.rb
-rw-r--r-- 1 root root  1247 Feb 11  2021 empty_udp.rb
-rw-r--r-- 1 root root  4071 Feb 11  2021 ipv6_multicast_ping.rb
-rw-r--r-- 1 root root  5735 Feb 11  2021 ipv6_neighbor.rb
-rw-r--r-- 1 root root  5710 Feb 11  2021 ipv6_neighbor_router_advertisement.rb
-rw-r--r-- 1 root root 12874 Feb 11  2021 udp_probe.rb
-rw-r--r-- 1 root root 11958 Feb 11  2021 udp_sweep.rb
```

其中两个常用模块

* arp_sweep：使用ARP请求枚举本地局域网络中的所有活跃主机。
* udp_sweep：通过发送UDP数据包探查指定主机是否活跃，并发现主机上的UDP服务。

在TCP/IP网络环境中，一台主机在发送数据帧前需要使用ARP（Address Resolution Protocol，地址解析协议）将目标IP地址转换成MAC地址，这个转换过程是通过发送一个ARP请求完成的。

### ARP扫描示例

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210816161513.png)

### 查点工具

在Metasploit的Scanner辅助模块中，有很多用于服务扫描和查点的工具，这些工具通常以`[serivce_name]_version`和`[service_nam]_login`命名。

* `[serivce_name]_version`可用于遍历网络中包含了某种服务的主机，并进一步确定服务的版本。
* `[service_nam]_login`可对某种服务进行口令探测攻击。

> 并非所有的模块都按照上述规范命名。

### 代理工具

metasploit提供了`open_proxy`模块，能够更加方便地获取免费的HTTP代理服务器地址，用于隐藏真实的IP地址。

> 由于许多代理服务器安全性无法得到保障，请确保没有重要的私密信息通过代理发送。

### wmap

Metasploit中内置的Web扫描器，允许用户使用和配置Metasploit中的辅助模块，对网站进行集中扫描。

```
msf6 > load wmap

.-.-.-..-.-.-..---..---.
| | | || | | || | || |-'
`-----'`-'-'-'`-^-'`-'
[WMAP 1.5.1] ===  et [  ] metasploit.com 2012
[*] Successfully loaded plugin: wmap

msf6 > help

wmap Commands
=============

    Command       Description
    -------       -----------
    wmap_modules  Manage wmap modules
    wmap_nodes    Manage nodes
    wmap_run      Test targets
    wmap_sites    Manage sites
    wmap_targets  Manage targets
    wmap_vulns    Display web vulns
    ..........
```

#### 扫描步骤

1. 通过命令添加要扫描的网站
2. 将添加的网站作为扫描目标
3. 查看哪些模块将会在扫描中使用

具体用法请参考[wmap官方链接](https://www.offensive-security.com/metasploit-unleashed/wmap-web-scanner/)。

> Web漏洞扫描器极大的提高渗透测试人员的效率，但是，由于对某些漏洞比如信息泄露、加密机制缺陷等漏洞无法探测，且还存在误报和漏报的问题（假阳性、假阴性），对探测到的漏洞，仍然需要手工探测来真正确定。

## encoders

对payload进行加密，绕过杀软的功能模块。

## evasion

绕过杀软的功能模块。

## exploits

漏洞利用模块，对已经扫描和收集到的目标的漏洞进行利用（有可能会失败）的模块。

## nops

提高payload稳定性及维持大小，空指令相关。

## payloads

exploits执行成功后，在目标系统真正执行的代码或指令等操作。

## post

后期渗透模块，执行一些后渗透操作，比如内网扫描和攻击、登录其他服务器、数据库等资产。







# msf简单使用步骤

## 漏洞扫描

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



## 漏洞利用

当我们发现目标存在相应的漏洞时，利用对应exploit模块即可进行攻击。



