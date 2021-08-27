NoSQL数据库由通常不遵循表格/关系模型的数据组成，这些称为“非结构化数据”，如图片，视频，社交媒体的数据。

在传统的 SQL 注入中，攻击者会尝试破坏SQL查询语句并在服务器端修改查询语句。使用NoSQL注入，攻击可以在应用程序的其他区域中执行，而不是在传统的SQL注入中执行。此外，在传统的SQL注入中，攻击者会使用一个标记来发起攻击。在NoSQL注入中，在NoSQL注入中，通常存在将字符串解析或评估为NoSQL调用的漏洞。

NoSQL注入中的漏洞产生：

* 端点接受的JSON数据是从NoSQL数据库中请求的
* 我们能够使用NoSQL比较运算符操作查询来更改NoSQL查询

NoSQL注入的一个常见例子是： [{"$gt":""}] 。这个JSON对象基本上是说运算符（$gt）大于NULL("")。由于逻辑上一切都大于NULL，因此JSON对象成为一个真正正确的语句，允许我们绕过或注入NoSQL 查询。这相当于SQL注入中的[' or 1=1—]。在MongoDB中，我们可以使用以下条件运算符之一：

* (>)大于 - $gt
* (<)小于 - $lt
* (>=)大于等于 - $gte
* (<=)小于等于 - $lte



## 示例

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210827190249.png)

通过抓包查看到用户名和密码，尝试对其修改再提交。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210827191501.png)

可以看到这里登录成功了。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210827191623.png)