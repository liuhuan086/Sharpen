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

#### ARP扫描示例

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210816161513.png)

#### 查点工具

在Metasploit的Scanner辅助模块中，有很多用于服务扫描和查点的工具，这些工具通常以`[serivce_name]_version`和`[service_nam]_login`命名。

* `[serivce_name]_version`可用于遍历网络中包含了某种服务的主机，并进一步确定服务的版本。
* `[service_nam]_login`可对某种服务进行口令探测攻击。

> 并非所有的模块都按照上述规范命名。

#### 代理工具

metasploit提供了`open_proxy`模块，能够更加方便地获取免费的HTTP代理服务器地址，用于隐藏真实的IP地址。

> 由于许多代理服务器安全性无法得到保障，请确保没有重要的私密信息通过代理发送。



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

当我们发现目标存在相应的漏洞时，利用对应exploit模块即可进行攻击。



## Nmap与Metasploit

Metasploit可以将渗透测试信息存入数据库中并可以共享。

每次msfconsole启动时，会自动连接到数据库，使用`db_status`可以查看与数据库连接状态。

```
msf6 > db_status
[*] postgresql selected, no connection
```

> emmm，这里还没有设置，因此显示是`no connection`

可以使用`db_connect`来连接数据库，因此也可以连接其他数据库查看漏洞信息。

```
msf6 > db_connect
[-] A URL or saved data service name is required.

   USAGE:
      * Postgres Data Service:
          db_connect <user:[pass]>@<host:[port]>/<database>
        Examples:
          db_connect user@metasploit3
          db_connect user:pass@192.168.0.2/metasploit3
          db_connect user:pass@192.168.0.2:1500/metasploit3
          db_connect -y [path/to/database.yml]
 
      * HTTP Data Service:
          db_connect [options] <http|https>://<host:[port]>
        Examples:
          db_connect http://localhost:8080
          db_connect http://my-super-msf-data.service.com
          db_connect -c ~/cert.pem -t 6a7a74c1a5003802c955ead1bbddd4ab1b05a7f2940b4732d34bfc555bc6e1c5d7611a497b29e8f0 https://localhost:8080
        NOTE: You must be connected to a Postgres data service in order to successfully connect to a HTTP data service.
 
      Persisting Connections:
        db_connect --name <name to save connection as> [options] <address>
      Examples:
        Saving:     db_connect --name LA-server http://123.123.123.45:1234
        Connecting: db_connect LA-server
 
   OPTIONS:
       -l,--list-services List the available data services that have been previously saved.
       -y,--yaml          Connect to the data service specified in the provided database.yml file.
       -n,--name          Name used to store the connection. Providing an existing name will overwrite the settings for that connection.
       -c,--cert          Certificate file matching the remote data server's certificate. Needed when using self-signed SSL cert.
       -t,--token         The API token used to authenticate to the remote data service.
       --skip-verify      Skip validating authenticity of server's certificate (NOT RECOMMENDED).
```

