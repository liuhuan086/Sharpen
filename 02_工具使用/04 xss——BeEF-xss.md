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

