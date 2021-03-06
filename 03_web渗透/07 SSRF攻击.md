# SSRF攻击

在计算机安全中，**服务器端请求伪造**（英语：Server-side Request Forgery，简称SSRF）是攻击者滥用服务器功能来访问或操作无法被直接访问的信息的[方式](https://zh.wikipedia.org/wiki/漏洞利用)之一。

服务器端请求伪造攻击将域中的不安全服务器作为[代理](https://zh.wikipedia.org/wiki/代理服务器)使用，这与利用[网页客户端](https://zh.wikipedia.org/wiki/网页浏览器)的[跨站请求伪造攻击](https://zh.wikipedia.org/wiki/跨站请求伪造)类似（如处在域中的浏览器可作为攻击者的代理）。

它是一种由攻击者构造，形成**由服务端发起请求**的一个安全漏洞。一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）。

SSRF 形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。

## 漏洞挖掘

- 数据层面需要关注的关键字段是URL、IP地址、链接等，关键字有：share、wap、url、link、src、source、target、u、3g、display、sourceURl、imageURL、domain……
- 业务层面需关注任何通过URL进行资源调用（入）或向外发起网络请求（出）的功能，如通过url文件上传下载处是存在SSRF最多的场景。其他具体业务场景包括：内容展示、社交分享、在线翻译、收藏功能、WebMail邮箱、各种处理工具（FFpmg）等

## 漏洞探测

- 请求包中将参数更改为不同的IP/DNS或TCP端口，观察返回包长度、返回码、返回信息及响应时间，不同则可能存在SSRF漏洞；
- 请求自己的公网服务器（或CEYE），使用nc –lvp监听请求。

## 漏洞利用

### 读取内部文件

然后我们尝试将路径进行替换，可以看到，这里将URL的值替换成服务器中配置文件的地址的时候，也被加载出来了。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210824083958.png)

如果这台机器还能访问内网其他服务器，那么就可以通过这台机器作为跳板，去对内网其他机器做渗透。

或者我们在某个网站存放一个病毒文件，将URL指向该文件，一旦访问之后，自动下载该文件到服务器中，然后可以通过该文件拿到shell执行任意指令。如果是挖矿病毒文件，那么服务器就会成为别人的免费劳动力。

> PS：每次新开一个漏洞都要配很久的靶机环境，早上先到这里，去上班了。

### 访问远程URL

访问一个存在加载远程URL漏洞的页面。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210824084456.png)

### 常用协议

```
http://：探测内网主机存活、端口开放情况
gopher://：发送GET或POST请求；攻击内网应用，如MySQL，redis
dict://：泄露安装软件版本信息，查看端口
file://：读取系统本地文件
php://：访问各个输入/输出流（I/O streams）
```

### 常见攻击行为

* 在回环接口上访问服务

* 使用`FILE://`等协议读取服务器上的本地文件

* 使用 AWS Rest 接口（http://bit.ly/2ELv5zZ）

* 横向移动到内部环境中

* 扫描内部网络和与这些服务的潜在交互方式（GET/POST/HEAD）

  使用burpsuite的Intruder进行抓包并重放，通过**dict协议**对端口进行尝试，查看返回包的长度不同，可以猜解哪些端口是开放的。

  ![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210824220915.png)

* 读取内部文件

  ```
  ?url=file:///etc/passwd
  ```

* 发现内部应用并攻击（redis、MySQL、zabbix......）

  当我们扫描出内网的端口和服务时，就能对对应的服务发起针对性攻击。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210830180524.png)

点击`Preview link`并抓包，通过抓包看到这里访问了我们填入的链接地址，那么，我们尝试填写回环地址，测试不同端口来看看，可以看到这里发现了28017端口的返回数据包长度和其他的不太相同，而28017是mongodb的端口。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210830181310.png)

通过浏览器打开页面，发现我们已经能够进入mongdb到后台。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210830181954.png)

> 参考链接：[SSRF漏洞(原理&绕过姿势)](https://www.t00ls.net/articles-41070.html)

## 思维导图

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huanSSRF%E6%BC%8F%E6%B4%9E.png)

