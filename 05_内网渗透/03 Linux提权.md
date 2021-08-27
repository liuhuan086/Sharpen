Linux中可以参考这篇文章，[Linux提权一文通](https://www.freebuf.com/articles/system/251884.html)。

在Linux中，也会存在`错误权限配置`的问题，通过查找所有用户具有写权限文件的SUID/GUID所有者是root以及不正确的配置文件，从而发现权限提升漏洞。



# 权限漏洞工具相关

## 权限漏洞查找工具

* [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check)
* [LinEnum](https://github.com/rebootuser/LinEnum)
* https://www.exploit-db.com/

## 权限提升漏洞工具列表

* [Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)





# 演示

## 查看系统版本

```
webper@webper-virtual-machine:~$ uname -a
Linux webper-virtual-machine 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

查找对应版本是否存在漏洞

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210827084740.png)

可以看到有对应的漏洞的存在，其中一个点进去之后可以看到有exp的脚本。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210827084928.png)

可以看到脚本开头是注释，介绍了该exp的使用方法，下面是C语言的代码。我们将完整代码复制到目标服务器上，或保存到本地再上传到目标服务器，按照使用方法去执行。

## 查找存放目录

在Linux下，`/var/tmp`目录是黑客最喜欢的目录，因为这里的目录权限比较高，任意用户都可以读写和执行。

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

## 提权

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

额，翻车了，先去上班吧，回头再看。