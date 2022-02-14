# SQLMap

`sqlmap -hh`可以用来查看详细参数列表。

## 一些技巧

* 需要定义攻击什么类型的数据库，如果认为存在注入点，而SQLMap却没有找到，就试着设置`--dbms=[database type]`参数。
* 如果需要测试存在认证继指的SQL注入操作，通过浏览器登录网站来获取Cookie，然后再使用`--cookie=[Cookie]`来设置。

* 最后如果确实不知道怎么操作，可以使用`sqlmap -wizard`来辅助进行测试。

    ```
    ┌──(huan㉿kali)-[/root]
    └─$ sqlmap -wizard                                                                     
            ___
           __H__
     ___ ___[.]_____ ___ ___  {1.5.2#stable}
    |_ -| . [,]     | .'| . |
    |___|_  ["]_|_|_|__,|  _|
          |_|V...       |_|   http://sqlmap.org
    
    [!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program
    
    [*] starting @ 02:51:21 /2021-08-19/
    
    [02:51:21] [INFO] starting wizard interface
    Please enter full target URL (-u): 10.4.7.5
    POST data (--data) [Enter for None]: "id=1"
    Injection difficulty (--level/--risk). Please choose:
    [1] Normal (default)
    [2] Medium
    [3] Hard
    > 1
    Enumeration (--banner/--current-user/etc). Please choose:
    [1] Basic (default)
    [2] Intermediate
    [3] All
    > 1
    
    sqlmap is running, please wait..
    ```

## POST参数示例

POST参数传递不是在URL中， 通过POST发送数据参数，sqlmap会像检测GET参数一样检测POST的参数。POST常常用于用域名和密码传递，网站服务器通常记录GET参数，且GET方法也有大小限制，因此对于大型应用程序传递较打数据时，通常使用POST参数。

```
--data=DATA  --data="id=1"
```

常规用法

```
sqlmap -u "http://10.4.7.5/mutillidae/index.php?page=login.php?username=xxx&password=xxx&login-php-submit-button=Login" -b
```







