# XSS攻击

**[XSS攻击参考链接](https://www.cnblogs.com/liuhuan086/p/14741974.html)**

XSS攻击主要是针对用户进行攻击，产生的原因是网站应用程序输入验证存在缺陷，产生原因是服务端未对用户输入数据做过滤。



## 危害

*  盗用用户身份
* 未授权操作：未授权读取、篡改数据
* 按键记录和钓鱼
* 开启摄像头等



## 类型

* 反射型：非持久型，需要欺骗用户自己去点击链接才能触发漏洞。
* 存储型：持久型，攻击者把代码保存到数据库中，当用户访问存在漏洞的网页就会触发恶意代码。
* DOM型：DOM—based XSS漏洞是基于文档对象模型（Document Objeet Model，DOM）的一种漏洞，DOM型是通过URL传入参数去控制触发的。



## 攻击示例

### 反射型

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820221000.png)

实施攻击

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820220925.png)

或者这样

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820221526.png)



### 存储型

在留言板中添加恶意代码，用来获取用户cookie，这样，黑客可以不用知道你的用户名和密码，也能够以你的账号信息做相关操作，比如给好友发送虚假信息，以你的名义转账或购买商品等。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820222734.png)

然后点击留言之后，会立即弹出该提示，并且每次访问该页面都会弹出来。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820222806.png)

### DOM型

DOM型XSS其实是一种特殊类型的反射型XSS，它是基于DOM文档对象模型的一种漏洞。

在网站页面中有许多页面的元素，当页面到达浏览器时浏览器会为页面创建一个顶级的Document object文档对象，接着生成各个子文档对象，每个页面元素对应一个文档对象，每个文档对象包含属性、方法和事件。可以通过**JS脚本**对文档对象进行编辑从而修改页面的元素。也就是说，客户端的脚本程序可以通过DOM来动态修改页面内容，从客户端获取DOM中的数据并在本地执行。基于这个特性，就可以利用**JS脚本**来实现XSS漏洞的利用。

