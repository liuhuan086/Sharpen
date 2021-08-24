# ARP欺骗

ARP（地址解析协议）欺骗通常是渗透测试最后保留的手段，或者用于一个非常具体的（目标）渗透测试。

ARP欺骗是一种中间人（MITM）攻击方式，主要基于ARP的协议缺陷，具体就是在OSI二层（MAC地址）转换只OSI三层（IP地址）过程中存在缺陷。

官话：在以太网协议中规定，同一局域网中的一台主机要和另一台主机进行直接通信，必须要知道目标主机的MAC地址。而在TCP/IP协议中，网络层和传输层只关心目标主机的IP地址。这就导致在以太网中使用IP协议时，数据链路层的以太网协议接到上层IP协议提供的数据中，只包含目的主机的IP地址。于是需要一种方法，根据目的主机的IP地址，获得其MAC地址。这就是ARP协议要做的事情。所谓地址解析（address resolution）就是主机在发送帧前将目标IP地址转换成目标MAC地址的过程。-- Extracted from WikiPedia.

通俗点说，在局域网中通信时使用的是MAC地址，而不是常见的IP地址。所以在局域网的两台主机间通信时，必须要知道对方的MAC地址，这就是ARP协议要做的事：将IP地址转换为MAC地址。

> [原文链接：浅谈ARP欺骗的实现与防御](https://www.freebuf.com/articles/network/210852.html)



# ARP欺骗工具

## cain and Abel

暂时未找到下载地址，先空着。



## [Ettercap](https://github.com/Ettercap/ettercap.git)

### 安装

下载源码并进入目录

```
cd /opt
git clone https://github.com/Ettercap/ettercap.git
cd ettercap
```

安装依赖库

```
sudo apt-get install debhelper bison check cmake flex ghostscript libbsd-dev libcurl4-openssl-dev libgeoip-dev libltdl-dev libluajit-5.1-dev libncurses5-dev libnet1-dev libpcap-dev libpcre3-dev libssl-dev libgtk-3-dev libgtk2.0-dev -y
```

开始安装

```
mkdir build
cd build
sudo cmake ../
sudo make install
```

安装完成后执行命令查看

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824110305.png)

先清空受害者机器的ARP表，并查看IP地址。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824112153.png)

然后开始攻击，[攻击教程参考：ettercap详细使用教程](https://blog.csdn.net/smli_ng/article/details/106133685)。

```
ettercap -G 命令可以使用图形化界面
```

然后查看攻击机的IP和mac地址

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824112650.png)

再查看受害者机器的IP和mac地址，发现mac地址已经是攻击机的mac地址了。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824112754.png)

尝试抓包，在受害者主机上登录一个网页查看是否能够获取到数据。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824130021.png)

在ettercap图形化界面通过以下方式查看结果。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824130434.png)

可以看到我们输入的账号密码全部都能获取到。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824125817.png)

### 插件

ettercap中还有很多插件，比如`dns_spoof`模块，这个模块可以控制受害者主机访问互联网上的地址。这些模块需要花时间去研究到底是什么样的攻击方式，以及能够得到什么样的结果。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210824124329.png)



