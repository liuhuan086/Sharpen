# SSH隧道

SSH隧道即SSH端口转发，在SSH客户端与SSH服务端之间建立一个隧道，将网络数据通过该隧道转发至指定端口，从而进行网络通信。SSH隧道自动提供了相应的加密及解密服务，保证了数据传输的安全性。

SSH隧道有三种端口转发模式：

* 本地端口转发（Local Port Forwarding）
* 远程端口转发（Remote Port Forwarding）
* 动态端口转发（Dynamic Port Forwarding）



## 原理

ssh在登录时，可以指定一些参数，来实现一些操作，详见[ssh参数详解](https://github.com/liuhuan086/Sharpen/blob/main/04_%E7%90%86%E8%AE%BA%E7%9F%A5%E8%AF%86/02%20SSH%E5%91%BD%E4%BB%A4%E5%8F%82%E6%95%B0.md)。



## 安全风险

现在有三台机器分别是A、B、C三台主机

```
10.4.7.A
10.4.7.B
10.4.7.C
```

其中B可以访问A和C，A和C互相不能访问，B对外提供Web服务，这时候，我们可以通过ssh隧道进行端口转发，利用B主机作为跳板机，访问A主机或C主机，这样便能利用B主机实现内网渗透。



## 演示

现在有三台靶机，IP分别为：

```
10.4.7.10
10.4.7.11
10.4.7.12
```

扫描一下这三台机器：

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210902153850.png)

可以看到三台都有22端口对外暴露，且10机器提供8080的web服务，接下来就尝试进行ssh隧道演示。

### ssh参数

```
ssh -L [LOCAL_IP]:LOCAL_PORT:JUMP_SERVER:JUMP_PORT [USER]@TARGET_SERVER
```

或者使用`-g`表示本地LOCAL_IP，不需要再手动填入IP地址

```
ssh -g -L LOCAL_PORT:JUMP_SERVER:JUMP_PORT [USER]@TARGET_SERVER
```

执行命令

```
ssh -g -L 2222:10.4.7.10:8080 hacker@10.4.7.11
```

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210902154933.png)

居然没有安装curl命令，那就直接访问页面吧。

![](https://borinboy.oss-cn-shanghai.aliyuncs.com/xntz/20210902155212.png)

可以看到这里直接通过我们监听在kali上的IP和端口访问到了`10`机器的`8080`端口的服务。

