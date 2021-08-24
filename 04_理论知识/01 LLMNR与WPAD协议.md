# 链路本地多播名称解析（LLMNR）

是一个基于域名系统（DNS）数据包格式的协议，IPv4和IPv6的主机可以通过此协议对同一本地链路上的主机执行名称解析。

在DNS 服务器不可用时，DNS 客户端计算机可以使用本地链路[多播](https://baike.baidu.com/item/多播)名称解析 (LLMNR—Link-Local Multicast Name Resolution)（也称为多播 DNS 或 mDNS）来解析本地网段上的名称。例如，如果[路由器](https://baike.baidu.com/item/路由器)出现故障，从网络上的所有 DNS 服务器切断了[子网](https://baike.baidu.com/item/子网)，则支持 LLMNR 的子网上的客户端可以继续在对等基础上解析名称，直到网络连接还原为止。

除了在网络出现故障的情况下提供名称解析以外，LLMNR 在建立临时对等网络（例如，机场候机区域）方面也非常有用。

## LLMNR工作过程

(1) 主机在自己的内部名称缓存中查询名称。如果在缓存中没有找到名称，那么主机就会向自己配置的主[DNS服务器](https://baike.baidu.com/item/DNS服务器/8079460)发送查询请求。如果主机没有收到回应或收到了错误信息，主机还会尝试搜索配置的备用DNS服务器。如果主机没有配置DNS服务器，或者如果在连接DNS服务器的时候没有遇到错误但失败了，那么名称解析会失败，并转为使用LLMNR。

(2) 主机通过[用户数据报协议](https://baike.baidu.com/item/用户数据报协议/8535496)(UDP)发送多播查询，查询主机名对应的IP地址，这个查询会被限制在本地子网(也就是所谓的链路局部)内。

(3) 链路局部范围内每台支持LLMNR，并且被配置为响应传入查询的主机在收到这个查询请求后，会将被查询的名称和自己的主机名进行比较。如果没有找到匹配的主机名，那么计算机就会丢弃这个查询。如果找到了匹配的主机名，这台计算机会传输一条包含了自己IP地址的单播信息给请求该查询的主机。



# WPAD（网络代理自动发现协议）

网络代理自动发现协议（Web Proxy Auto-Discovery Protocol，WPAD）是一种客户端使用[DHCP](https://baike.baidu.com/item/DHCP/218195)或[DNS](https://baike.baidu.com/item/DNS/427444)发现方法来定位一个配置文件[URL](https://baike.baidu.com/item/URL/110640)的方法。在检测和下载配置文件后，它可以执行配置文件以测定特定[URL](https://baike.baidu.com/item/URL/110640)应使用的代理。

网络代理自动发现协议(Web Proxy Autodiscovery Protocol)，通过让[浏览器](https://baike.baidu.com/item/浏览器/213911)自动发现代理服务器 ，定位代理配置文件，下载编译并运行，最终自动使用代理访问网络。

代理自动配置文件(Proxy Auto-Config，PAC)，定义了浏览器和其他用户代理如何自动选择适当的代理服务器来访问一个URL。要使用 PAC，我们应当在一个网页服务器上发布一个PAC文件，并且通过在浏览器的代理链接设置页面输入这个PAC文件的URL或者通过使用WPAD协议告知用户代理去使用这个文件。

## 实现原理简介

在获取第一个页面前，实现有此方法的网页浏览器给本地[DHCP服务器](https://baike.baidu.com/item/DHCP服务器/9956953)发送一个**DHCPINFORM**查询，并使用服务器的回复中的WPAD选项。如果DHCP服务器没有提供所需的信息，则再使用DNS。假设用户的计算机网络名称为pc.department.branch.example.com，浏览器将依次尝试下列[URL](https://baike.baidu.com/item/URL/110640)，成功在客户端的域中找到一个代理配置文件：

- http://wpad.department.branch.example.com/wpad.dat
- http://wpad.branch.example.com/wpad.dat
- http://wpad.example.com/wpad.dat
- http://wpad.com/wpad.dat

## PAC文件

一个PAC文件包含一个[JavaScript](https://baike.baidu.com/item/JavaScript)形式的函数“FindProxyForURL(url, host)”。这个函数返回一个包含一个或多个访问规则的字符串。用户代理根据这些规则适用一个特定的代理器或者直接访问。 当一个代理服务器无法响应的时候，多个访问规则提供了其他的后备访问方法。 浏览器在访问其他页面以前，首先访问这个PAC文件。

## 自动化技术

现代的浏览器实现了几个级别的自动化，用户可以选择最适合他们需要的级别。下面的这些方法被普遍的实现：

- 手动代理配置：为所有的URLs规定一个主机名和端口作为代理。大多数浏览器允许用户规定一个域名的列表(例如 localhost)，访问这个列表里面的域名的时候不通过代理服务器。

- 代理自动配置(PAC)：规定一个指向PAC文件的URL，这个文件中包括一个[JavaScript](https://baike.baidu.com/item/JavaScript)函数来确定访问每个URL时所选用的合适代理。这个方法更加适合需要几个不同代理配置的笔记本用户，或者有很多不同代理服务器的复杂的企业级设置。

- 网络代理自发现协议(WPAD)： 浏览器通过DHCP和DNS的查询来搜索PAC文件的位置。

    DHCP比DNS有着更高的优先级：如果DHCP提供了WPAD URL，则不会进行DNS查询。只能配合DHCPv4使用，WPAD-Option选项没有在[DHCPv6](https://baike.baidu.com/item/DHCPv6/2778674)中定义。

## 安全性

在极大简化组织中浏览器配置的同时，WPAD协议必须被小心使用，攻击者可以篡改用户浏览器中显示的内容：

- 网络内部的攻击者可以创建一个DHCP服务器，提供一个恶意PAC脚本的网址。
- 如果网络是“company.co.uk”并且http://wpad.company.co.uk/wpad.dat文件未提供，浏览器将请求http://wpad.co.uk/wpad.dat。浏览器不会测定这是否仍处在公司内部。
- 同样的方法已被http://wpad.org.uk使用。它提供一个wpad.dat文件从而将用户的所有流量重定向到一个互联网拍卖网站。
- 实施DNS劫持的ISP可能阻断WPAD协议的DNS查询，从而将用户引导至非代理服务器的主机。
- 泄露的WPAD查询可能导致与内部网络命名方案的域名冲突。如果攻击者注册一个域来回答WPAD查询并配置一个有效的代理，这有可能进行跨越互联网的中间人攻击 [2] 。

通过WPAD文件，攻击者可以引导用户的浏览器到他们自己的代理，拦截和修改所有WWW流量。尽管2005年对Windows WPAD的处理应用了一个简单的修复，但它只解决了.com域的问题。在Kiwicon的一次演讲表示，世界上的许多地方仍有极其脆弱的安全漏洞，在新西兰为测试目的注册的一个示例域名在几秒内收到了来自全国各地的代理请求。几个wpad.tld域名（包括COM, NET, ORG和US）当前指向客户端[环回地址](https://baike.baidu.com/item/环回地址/8443645)以帮助防范此安全漏洞，虽然还有一些名称仍是注册状态（如wpad.co.uk）。

因此，管理员应确保用户可以信任组织中的所有[DHCP服务器](https://baike.baidu.com/item/DHCP服务器/9956953)，并且组织内的所有可能的wpad域都在控制之下。此外，如果没有为组织配置wpad域，用户将转向域层次结构中的下一个（更高层级）wpad站点并使用其配置。这允许能在顶级域注册wpad子域名的人将自己的代理设为对所有流量或特定站点提供服务，从而施行对特定国家或区域的大范围[中间人攻击](https://baike.baidu.com/item/中间人攻击/1739730)。

除了上述“陷阱”，WPAD方法是获取一个JavaScript文件并在所有用户浏览器上执行，即使用户已禁用查看网页时使用的JavaScript。

> **——以上解释来源于百度百科**。

