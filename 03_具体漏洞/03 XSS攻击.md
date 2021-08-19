# XSS攻击

**[XSS攻击参考链接](https://www.cnblogs.com/liuhuan086/p/14741974.html)**

XSS攻击主要是针对用户进行攻击，产生的原因是网站应用程序输入验证存在缺陷，产生原因是服务端未对用户输入数据做过滤。

## 危害

*  窃取 Cookie
* 未授权操作：蠕虫攻击
* 按键记录和钓鱼
* 开启摄像头等



## BeEF漏洞工具

> 吐槽一刻：github拉不下来源码，难受

可以通过docker安装：

```
docker run --rm -p 3000:3000 janes/beef
```

