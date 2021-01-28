## 就是要让你搞懂Nginx，这篇就够了！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/ow6przZuPIENb0m5iawutIf90N2Ub3dcPuP2KXHJvaR1Fv2FnicTuOy3KcHuIEJbd9lUyOibeXqW8tEhoJGL98qOw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/sz_mmbiz_jpg/knmrNHnmCLHVyDRF2DkD2tqrLqyJ7zqfkrnM0lnfURX8lyojEbqVGV0212PW4VSN6HvrDg6Apt1COriaxPsIS4Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

------

> 作者：
>
> https://blog.csdn.net/yujing1314/article/details/107000737

### 目录

- 1.Nginx 知识网结构图

- 

- - 1.1 反向代理
  - 1.2 负载均衡
  - 1.3 动静分离

- \2. nginx 如何在 linux 安装

- \3. nginx 常用命令

- 4.nginx 的配置文件

- 

- - 4.1 反向代理实战
  - 4.2 反向代理小结
  - 4.3 负载均衡实战
  - 4.4 动静分离实战

- 5.nginx 高可用

- 

- - 5.1 安装 keepalived

- 6. 原理解析

- 小结

# 1.Nginx 知识网结构图

![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t7UeibfWosX57mz5WEEK1lJLUBpRhX9ghwLOicQbZwM0rVwpvGJk1OAhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
Nginx 是一个高性能的 HTTP 和反向代理服务器，特点是占用内存少，并发能力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好

nginx 专为性能优化而开发，性能是其最重要的要求，十分注重效率，有报告 nginx 能支持高达 50000 个并发连接数

## 1.1 反向代理

**正向代理**
正向代理：局域网中的电脑用户想要直接访问网络是不可行的，只能通过代理服务器来访问，这种代理服务就被称为正向代理。
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tEdhOKBFz2ESqO32ZeybtlbsUK2NhlQLAvEeIl46gR9Osa3lo0kSMbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**反向代理**
反向代理：客户端无法感知代理，因为客户端访问网络不需要配置，只要把请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据，然后再返回到客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t1UibsaktKxYp2XjvKQfF7HjTXJ0S5kUZdzxzqE4LtC58kzw7EdGia4lg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1.2 负载均衡

客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行狡猾，服务器处理完毕之后，再将结果返回给客户端

普通请求和响应过程
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t5j9tkx68yhxb7jGJu075QYlA76rQNe1bD0xLp8R4UpyztQg0ZYibyyA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
但是随着信息数量增长，访问量和数据量飞速增长，普通架构无法满足现在的需求

我们首先想到的是升级服务器配置，可以由于摩尔定律的日益失效，单纯从硬件提升性能已经逐渐不可取了，怎么解决这种需求呢？

我们可以增加服务器的数量，构建集群，将请求分发到各个服务器上，将原来请求集中到单个服务器的情况改为请求分发到多个服务器，也就是我们说的负载均衡

**图解负载均衡**
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tPxTXXfX4MZhDPvk1MJ6Oq6EC5ibjCRWwyW9UlWmTS7HPCBOpPMKwquQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
假设有 15 个请求发送到代理服务器，那么由代理服务器根据服务器数量，平均分配，每个服务器处理 5 个请求，这个过程就叫做负载均衡

## 1.3 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面交给不同的服务器来解析，加快解析的速度，降低由单个服务器的压力

动静分离之前的状态
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tm7ePofUUOPNFAx7ZhDlX182xpNSZoY5NYyibNCrSBrwGBBSZP5BHiayw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
动静分离之后
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tEOyz8X02LsQibd1GXiaywSSRR1KczbOEHBUMbFhQa6d5ZSZmhPPvFDcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 2. nginx 如何在 linux 安装

https://blog.csdn.net/yujing1314/article/details/97267369

# 3. nginx 常用命令

查看版本

```
./nginx -v
```

启动

```
./nginx
```

关闭（有两种方式，推荐使用 ./nginx -s quit）

```
 ./nginx -s stop ./nginx -s quit
```

重新加载 nginx 配置

```
./nginx -s reload
```

# 4.nginx 的配置文件

配置文件分三部分组成

全局块
从配置文件开始到 events 块之间，主要是设置一些影响 nginx 服务器整体运行的配置指令

并发处理服务的配置，值越大，可以支持的并发处理量越多，但是会受到硬件、软件等设备的制约
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tJICv6796m3raxOqrpusibicmPSibnEkpFHAVzZf2CjA4776yEwM4HfIZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

events 块
影响 nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 workprocess 下的网络连接进行序列化，是否允许同时接收多个网络连接等等

支持的最大连接数
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tYhXNNb6RwP8zFIYrRqDnK9yjrzAvC1Q8fSQKuRUEibYVxztJicLfH7WQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
http 块
诸如反向代理和负载均衡都在此配置

**location 指令说明**

- 该语法用来匹配 url，语法如下

```
location\[ = | ~ | ~\* | ^~\] url{}
```

1. =: 用于不含正则表达式的 url 前，要求字符串与 url 严格匹配，匹配成功就停止向下搜索并处理请求
2. ~：用于表示 url 包含正则表达式，并且区分大小写。
3. ~*：用于表示 url 包含正则表达式，并且不区分大瞎写
4. ^~：用于不含正则表达式的 url 前，要求 ngin 服务器找到表示 url 和字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再匹配
5. 如果有 url 包含正则表达式，不需要有~ 开头标识

## 4.1 反向代理实战

**配置反向代理**
目的：在浏览器地址栏输入地址 www.123.com 跳转 linux 系统 tomcat 主页面

具体实现
先配置 tomcat：因为比较简单，此处不再赘叙
并在 windows 访问
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tubfMls9CpDdg2tFnITvnI9NnIuQFF9twphU54tQPQtQ4yn5vgKvUng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
具体流程
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tNd5rNd3T2RJzwECkmtmN5mlMjEhCoibibHNDo0icmcq7VWMQgfc3kxqUg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
修改之前
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tRAIOenkMVR9PrR0HD0icHQTyW2P4zmhwGoqibibFicibSmXQmsib3rmQHXBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tWzmpCeuOMaWabZ1K1s7jcbJEUEn6oxruklibYsFKqrvddS5bPYQl9JQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
再次访问
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tmTf2dreFJGCJX08IMaLW5SmDpZ9AwNQI5wDck04J5fvbS1fXqggzgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
**反向代理 2**

\1. 目标
访问 http://192.168.25.132:9001/edu/ 直接跳转到 192.168.25.132:8080
访问 http://192.168.25.132:9001/vod/ 直接跳转到 192.168.25.132:8081

\2. 准备
配置两个 tomcat，端口分别为 8080 和 8081，都可以访问，端口修改配置文件即可。
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tU3JFTGCdhicf8m0ewDibLsgibOXHyS0nClsuAwgl3k2K6iaHxWSnic0UbbA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tuLgck4qpHLMMB3HUZdwWZHnPzy4fic0zVP86UA4TTfOdGFPmAZcWXsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

新建文件内容分别添加 8080！！！和 8081！！！
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tyjxUFialaGBaibOmMpTia8dicg3UOnoHn0fyvrtottudO5C4vAaKUZ2GDA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tTjjcibhnIQcR6cjJJE1cHCT5ufSWyxTDdk2e1ZqmfSFqnxoZMN01KSw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
响应如下
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tXyHp3v7LDoNPicdagthG4H0Qj0VIDJq74QR2uKhrGyWkj4V8xLicH4CA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tibkVLqLwgMRgtJs66qynrv1Zwm06ibHbwNz3tXMmRWHX4bSzpJMX8E6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
\3. 具体配置
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tib5iaD7V9u8DoQaSWVhzD7Bib1sJVEiculxafK5BpHjm8hdHm3ficrZE9Kg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
重新加载 nginx

```
./nginx -s reload
```

访问
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tib3icibsVS9riby5tpEUyKhuNeA23Yb3e2t6xs0cUt4BadmBCTEoymlSCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tib3icibsVS9riby5tpEUyKhuNeA23Yb3e2t6xs0cUt4BadmBCTEoymlSCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
实现了同一个端口代理，通过 edu 和 vod 路径的切换显示不同的页面

## 4.2 反向代理小结

第一个例子：浏览器访问 www.123.com，由 host 文件解析
出服务器 ip 地址

192.168.25.132 www.123.com
然后默认访问 80 端口，而通过 nginx 监听 80 端口代理到本地的 8080 端口上，从而实现了访问 www.123.com，最终转发到 tomcat 8080 上去

第二个例子：
访问 http://192.168.25.132:9001/edu/ 直接跳转到 192.168.25.132:8080
访问 http://192.168.25.132:9001/vod/ 直接跳转到 192.168.25.132:8081

实际上就是通过 nginx 监听 9001 端口，然后通过正则表达式选择转发到 8080 还是 8081 的 tomcat 上去

## 4.3 负载均衡实战

\1. 修改 nginx.conf
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tBs2ot3OYI8YI7HQG22QYibfaYgolXl3J3oQjXia890njfiaz30NnTlL1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t0IOOLGD1xpMUDpKBRBKVwVvFtEOZjrFKN5iaiaRNGk9jT89VmYdnNoPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
\2. 重启 nginx

```
./nginx -s reload
```

\3. 在 8081 的 tomcat 的 webapps 文件夹下新建 edu 文件夹和 a.html 文件，填写内容为 8081！！！！

\4. 在地址栏回车，就会分发到不同的 tomcat 服务器上
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tPbaYnIvyYN4Be60U6vk57NBIWRsM644uChKrMCCcqPEibmXPKWLo6iaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tsYPdoVM7KMkjmEIl4nwkTiaicctDRnvicQTdjv9dILQSzhgIDXmVBKBwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
\3. 负载均衡方式

- 轮询（默认）
- weight，代表权，权越高优先级越高
  ![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tch70NLZkf8qtubY2pkwBqtmR4H7gfvqj4unic8libudnjRbzxdv2ysNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
- fair，按后端服务器的响应时间来分配请求，相应时间短的优先分配
  ![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tUWXfYCaQm3YulEibzatNpiceqJEOy40PjdCNicD9RLK08GDLjYhiaSVc4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
- ip_hash, 每个请求按照访问 ip 的 hash 结果分配，这样每一个访客固定的访问一个后端服务器，可以解决 session 的问题
  ![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tev1ibmcZIKD8E5ibK8AQGnibNFExEbT0zNmrJ9m4SrD5zUQFjJfMVLmZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 4.4 动静分离实战

**什么是动静分离**
把动态请求和静态请求分开，不是讲动态页面和静态页面物理分离，可以理解为 nginx 处理静态页面，tomcat 处理动态页面

动静分离大致分为两种：一、纯粹将静态文件独立成单独域名放在独立的服务器上，也是目前主流方案；二、将动态跟静态文件混合在一起发布，通过 nginx 分开

**动静分离图析**
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tE4LU7FSn61vfwUFEPGeelI2lIQ2p8NNtrdjCCiclvmzPWmZrgyKbmLw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
**实战准备**
准备静态文件

![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tdDkdwlqgQsLmP7O7YFqbWUTHUia6Wr2eicLnoTM6wCV4kicNVXlA0UjeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tLslycwYSYjdl03enXkMtCcu3jbUYYGVqH0ErIfQVIrqUqZtsfedtWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
配置 nginx
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tcicPibFv9ibVnqenxgnRO4SyHWJnp8dEZxBc4xD7RXWBUfRv4YwdUxauA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 5.nginx 高可用

如果 nginx 出现问题
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t8OxB1Qf85X4JYCno1lMJJcojybZs4U052uqEEEqCpDI6YjEZb1gMCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
解决办法
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tPbprzeQ92u37HUfaqUdIpx4JV9SsCn4xvfoIBEqiav16TxWemiaBeibtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
前期准备

1. 两台 nginx 服务器
2. 安装 keepalived
3. 虚拟 ip

## 5.1 安装 keepalived

```
\[root@192 usr\]# yum install keepalived -y\[root@192 usr\]# rpm -q -a keepalivedkeepalived-1.3.5-16.el7.x86\_64
```

修改配置文件

```
\[root@192 keepalived\]# cd /etc/keepalived\[root@192 keepalived\]# vi keepalived.conf
```

分别将如下配置文件复制粘贴，覆盖掉 keepalived.conf
虚拟 ip 为 192.168.25.50

> 对应主机 ip 需要修改的是
> smtp_server 192.168.25.147（主）smtp_server 192.168.25.147（备）
> state MASTER（主） state BACKUP（备）

```
global\_defs {   notification\_email {     acassen@firewall.loc     failover@firewall.loc     sysadmin@firewall.loc   }   notification\_email\_from Alexandre.Cassen@firewall.loc   smtp\_server 192.168.25.147   smtp\_connect\_timeout 30   router\_id LVS\_DEVEL # 访问的主机地址}vrrp\_script chk\_nginx {  script "/usr/local/src/nginx\_check.sh"  # 检测文件的地址  interval 2   # 检测脚本执行的间隔  weight 2   # 权重}vrrp\_instance VI\_1 {    state BACKUP    # 主机MASTER、备机BACKUP    interface ens33   # 网卡    virtual\_router\_id 51 # 同一组需一致    priority 90  # 访问优先级，主机值较大，备机较小    advert\_int 1    authentication {        auth\_type PASS        auth\_pass 1111    }    virtual\_ipaddress {        192.168.25.50  # 虚拟ip    }}
```

启动

```
\[root@192 sbin\]# systemctl start keepalived.service
```

![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tvktibJ20xq8MqGZyZN920ZS70jnHkLm6ecUwxfIzbbb5JtqvKzzUmVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
访问虚拟 ip 成功
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tEVFKH8aibjlKSDo5licKSuiaubeR3avbFOjTXib3mZjSQt2pGk4DnCppyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
关闭主机 147 的 nginx 和 keepalived，发现仍然可以访问

# 6. 原理解析

![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9t486Te3lmxcnMPNSFibYpYXK1RDuKLAakiaC6Yx09emYHs6FaysEeuK5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
如下图，就是启动了一个 master，一个 worker，master 是管理员，worker 是具体工作的进程
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tDhUJHjAZgCEZPVfuzEicCCce91XMqR0uM1yF7WqHDic2un8AEO0rYkKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
worker 如何工作
![img](https://mmbiz.qpic.cn/mmbiz_png/9Eibnmwqk0AhicJmwqPCbs0FjJLxCGBG9tXIWDJ8fcVrd4eFqtM729vK3EdwuxKUMicemZA3pMSDSTE2pg0jcVEEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 小结

- worker 数应该和 CPU 数相等
- 一个 master 多个 worker 可以使用热部署，同时 worker 是独立的，一个挂了不会影响其他的

