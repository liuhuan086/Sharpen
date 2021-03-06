# CSRF攻击

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。这利用了web中用户身份验证的一个漏洞：

**简单的身份验证只能保证请求是发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的**。

## 浏览器cookie

### 什么是cookie

因为HTTP协议是无状态的，网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密），称之为cookie。Cookie保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。

* 内存Cookie：由浏览器维护，保存在内存中，浏览器关闭即消失，存在时间短暂。
* 硬盘Cookie保存在硬盘里，有过期时间，除非用户手动清理或到了过期时间，硬盘Cookie不会清除，存在时间较长。所以，按**存在时间**，可分为**非持久Cookie**和**持久Cookie**。

> 不同浏览器对于cookie的发送策略有所不同，比如FireFox可以发送第三方Cookie，而IE浏览器则不允许。
>
> 对于IE浏览器，攻击者需要精心构造攻击环境，比如诱使用户在当前浏览器中先访问目标站点，使得Session Cookie有效，再实现CSRF攻击。
>
> 但如果CSRF攻击的目标并不允许使用Cookie，则也不必考虑浏览器的Cookie策略了。

## 漏洞点

### P3P头

P3P Header是W3C制定的一项关于隐私的标准。

如果网站返回给浏览器的HTTP头中包含有P3P头，则在某种程度上来说，将允许浏览器发送第三方Cookie。在IE下，即使是`<iframe>`、`<script>`等标签，也将不再拦截第三方Cookie的发送。

在网站的业务中，P3P头主要用于类似广告等需要跨域访问呢的页面。但很遗憾的是，P3P头设置后，对于Cookie的影响将扩大到整个域中的所有页面，因为Cookie是以域和path为单位的，这并**不符合最小权限原则**。

`www.b.com`中的一个页面的标签中，包含了一个指向`www.a.com`的iframe。

```html
<iframe width=300 height=300 src="http://www.a.com/test.php"></iframe>
```

`www.a.com/test.php`是一个对`a.com`域设置的Cookie的页面，其内容为：

```php
<?php
header("Set-Cookie: test=axis; domain=.a.com; path=/")
?>
```

当请求`www.b.com/test.html`时，它的iframe会告诉浏览器去跨域请求`http://www.a.com/test.php`，test.php会尝试`Set-Cookie`，所以浏览器会收到一个Cookie。

如果`Set-Cookie`成功，再次请求该页面，浏览器应该会发送刚才收到的Cookie。可是由于跨域限制，在`a.com`上`Set-Cookie`是不会成功的，所以无法发送刚才收到的Cookie。这里无论临时Cookie还是本地Cookie都一样。

但如果加上P3P Header，P3P Header的介入改变了a.com的隐私策略，从而使得``<iframe>`、`<script>`等标签在IE中不再拦截第三方Cookie的发送。因此，上面的第二次请求就可以成功发出之前收到的Cookie。

很多时候，如果测试CSRF时发现`<iframe>`等标签在IE中居然能发送Cookie，而又找不到原因，那么很可能就是因为P3P Header在作怪。

### GET、POST

POST和GET一样，都可以产生CSRF攻击。

一些重要操作并未严格区分GET、POST请求。攻击者可以使用GET来请求表单的提交地址。比如在PHP中，如果使用的是`$_REQUEST`，而非`_POST`获取变量，则会存在这个问题。

如果服务器端已经区分了GET与POST，也有若干种方式来构造出一个POST请求。最简单的办法就是在一个页面中构造好一个form表单，然后使用JavaScript自动提交这个表单。

```html
<form action="http://www.a.com/register" id="register" method="post">
    <input type=text, name="username" value="" />
    <input type=text, name="password" value="" />
    <input type=submit, name="submit" value="submit" />
</form>
<script>
var f = document.getElementById("register");
f.input[0].value="test";
f.input[0].value="passwd";
</script>
```

攻击者甚至可以将这个页面隐藏在一个不可见的iframe窗口中，那么整个自动提交表单的过程，对于用户来说也是不可见的。

### Flash CSRF

Flash也有多种方式能够发起网络请求，包括POST。

## 危害

* 盗用用户身份
    * 转移受害者资产
    * 利用受害者身份发送消息欺骗其他用户
    * 获取受害者能访问的相关资源
* 自动进行某些操作，可以参考曾经发生过的[百度CSRF蠕虫攻击事件](https://blog.csdn.net/weixin_34409741/article/details/85166567)。

## 防御

==**安全防御的体系应该是相辅相成，缺一不可的。**==

### 使用验证码

验证码被认为是对抗CSRF攻击最简洁有效的防御方法。

因为CSRF攻击的过程，往往是在用户不知情的情况下构造了网络请求。而验证码，则强制用户必须与应用进行交互，才能完成最终请求。因此，在通常情况下，验证码能够很好的遏制CSRF攻击。

> 你将访问xxx网站，请输入验证码以验证本次操作，如不是你本人操作，无需理会。

### 使用Token

使用Token是业内防御CSRF攻击统一的做法。

CSRF为什么能够攻击成功？其本质原因是**重要操作的所有参数都是可以被攻击者猜到的**。攻击者只有预测出URL的所有参数与参数值， 才能成功的构造一个伪造的请求。反之，则不会成功。

出于这个原因，可以想到一个解决方案：把参数加密，或者使用一些随机数。从而让攻击者无法猜测到参数值。这是”**不可预测性原则**“的一种应用。但是把参数进行加密难以读懂，加随机数导致用户可能无法收藏URL（每次随机数都不一样），最后，普通的参数如果也被加密，将增加后端处理的工作量。因此，Token就出现了。

Token需要足够随机，必须使用足够安全的随机数生成算法，或者采用真随机数生成器（物理随机），比如CPU温度、硬盘的当前容量等。在实际应用中，Token可以放在用户的Session中，或者浏览器的Cookie中。

由于Token的存在，攻击者无法再构造出一个完整的URL实施CSRF攻击。

Token需要同时放在表单和Session中。在提交请求时，服务器只需验证表单中的Token，与用户Session（或Cookie）中的Token是否一致，如果一致，则认为是合法请求。

**Token使用原则**

这个Token的目的不是为了防止重复提交，所以为了使用方便，可以允许在同一个用户的有效生命周期内，在Token被消耗掉前（未过期，未被使用）都能使用该Token。但是如果用户已经提交过表单，那么就应该重新生一个新的Token（防止被窃取之后进行复用）。

如果Token是保存在Cookie中，而不是服务器端的Session中，则会带来一个新的问题。如果用户打开了多个相同的页面同时操作，那么按照上面的逻辑，就会出现需要重新生成的问题。在这种情况下，可以考虑生成多个有效的Token。

最后，使用Token应该注意Token的保密性，如果Token出现在URL中，则可能会通过Referer的方式泄露。**尽量把Token放在表单中，通过Post请求，以form表单或（AJAX方式）提交，可以有效避免Token泄露的问题？？？**

Token仅仅用于对抗CSRF攻击，如果网站存在其他漏洞，比如XSS漏洞，Token依然会被攻击者利用，比如攻击者获取到该Token请求页面之后，获取页面内容的Token，再构造一个合法的请求，该过程被称为XSRF攻击。

### Refer Check

> 该方法可以通过抓包来绕过

Refer Check在互联网中最常见的应用就是“防止图片盗链”，

> 在我们的服务器对抗压力的时候，他人的网站引用你的图片，进而会消耗你网络带宽资源的一种行为。
>
> **防盗链原理**
>
> http 协议中，如果从一个网页跳到另一个网页，http 头字段里面会带个 Referer。图片服务器通过检测 Referer 是否来自规定域名，来进行防盗链。

比如，一个“论坛发帖”的操作，正常情况下，需要先登录到用户后台，或者访问有发帖功能的页面。在提交表单时，Referer的值必然是发帖表单所在的页面。如果Referer的值不是这个页面，甚至不是发帖网站的域，则极有可能是CSRF攻击。

即使我们能够通过检查Referer是否合法来判断用户是否被CSRF攻击，也仅仅是满足了防御的充分条件。Referer Check的缺陷在于，服务器并非什么时候都能取到Referer。很多用户出于隐私保护的考虑，限制了Referer的发送。在某些情况下，浏览器也不会发送Referer，比如从HTTPS跳转到HTTP，处于安全考虑， 浏览器也不会发送Referer。

## 演示

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210823222326.png)

登录之后，打开burpsuite并抓包，点击修改密码查看抓包的内容。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210823225636.png)

然后使用burpsuite来构造PoC，emmm，需要专业版才能使用`构造CSRF PoC`的功能。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210823224231.png)

那就通过其他方式手动生成PoC吧。

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://www.webtester.com/csrf/csrf01.php?c=update" method="POST">
      <input type="hidden" name="password" value="a123456" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

将上面的内容保存成`.html`文件，然后通过浏览器打开，查看页面只有一个`Submit request`按钮。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210823230424.png)

点击按钮再查看结果，发现提示修改成功。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/huan20210823230752.png)

为什么该HTML文件就能执行成功呢？我们一条一条的查看它的代码

```
  <script>history.pushState('', '', '/')</script>
```

> h5中提供了不修改页面内容只修改地址栏的api，pushState(添加浏览历史)
>
> pushState方法接受三个参数，依次为：
>
> state：一个与指定网址相关的状态对象，popstate事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填null。可用它来传一些数据
>
> title：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填null。
>
> url：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。

再接着看`form`表单，这里就是往目标网站发送表单数据。

```
    <form action="http://www.webtester.com/csrf/csrf01.php?c=update" method="POST">
```

继续看下面的代码，是一个`input`标签，最关键的地方在于，这里定义了`type=hidden`，也就是说这个`input`输入框对于用户来说是隐藏的，看不见的，当用户点击提交之后，就会在不知情的情况下修改密码。

```
      <input type="hidden" name="password" value="a123456" />
```

