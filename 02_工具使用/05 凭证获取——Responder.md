# Responder

[**LLMNR与WPAD参考链接**](https://github.com/liuhuan086/Sharpen/blob/main/04_%E7%90%86%E8%AE%BA%E7%9F%A5%E8%AF%86/01%20LLMNR%E4%B8%8EWPAD%E5%8D%8F%E8%AE%AE.md)

Responder是实现监听LLMNR（本地链路多播名称解析，Linux Local Multicast Resolution）协议和NBT-NS（网络基本输入输出协议域名服务，NetNIOS over TCP/IP Name Service）协议工具之一。

Responder还利用了另外一个漏洞——WPAD（网络代理自动发现协议，Web Proxy Auto-Discovery Protocol）漏洞。基本工作原理是当浏览器（IE或者网络LAN设置）设置为自动检测配置时，受害者主机将试图从网络上获取配置文件。

## Windows默认配置

控制面板——网络和Internet——连接——局域网设置

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210823162412.png)

可以看到这里默认时自动检测设置。

## 使用

**Responder常用参数**

* -i：本地主机的IP地址
* -I：设置网卡名称
* -b：设置为off表示用于关闭NTLM鉴权
* -r：设置为off，否则会对网络流量造成影响

Responder开始运行后，程序自动识别请求并发送恶意响应。

```bash
python Responder.py -I eth0 -i 10.4.7.133 -b Off -r Off -w On
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210823180704.png)

可以看到，该脚本首先使用LLMNR协议对`10.4.7.133`进行欺骗，恶意的WPAD文件发送到目标主机。这意味着受害者的网页浏览数据都经过服务器作为代理，这同样意味着可以获取所有明文信息。

