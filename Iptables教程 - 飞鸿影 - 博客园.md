# Iptables教程 - 飞鸿影 - 博客园
`iptables` 是一个配置 Linux 内核 防火墙 的命令行工具。

初学者刚看到iptables，会感到很复杂，原因是 iptables 功能实在是太强大了。本文会从基本概念、使用上做介绍，读者看完后再去看 iptables 命令就能理解其含义了。

> 本文环境：

PC: Ubuntu18

iptables操作均在docker上进行。docker上的系统为centos6.9。

更新时间：2020.6.26

工作原理[#](#工作原理)
--------------

### 基本组成[#](#基本组成)

想要掌握 iptables ，需要了解其原理。

iptables 组成：

```null
Tables -> Chains -> Rules

```

**表 (tables)**:

```null
filter
nat
mangle
raw

```

每个 tables 有多个**链 (Chains)**：

[![](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626165835050-1714638159.png)
](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626165835050-1714638159.png)  
_(图片来自: [https://phoenixnap.com/](https://phoenixnap.com/))_

> 其中链是可以自定义新建的，后文会介绍。

最终我们会对某个链添加 **规则(rules)** 。下面是一条由具体的指令，包含了 tables、chains、rules：

```null

iptables -t filter -A INPUT -s "172.18.0.3" -j DROP

```

> 注: `-t filter`指的是指定对`filter`表操作，可以省略，默认就是 `filter` 表。本条指令看不懂也没有关系，后文还会出现。

### 功能介绍[#](#功能介绍)

下面介绍各个表、链的基本功能。我们实际使用最多的是Filter表、NAT表，所以本文主要也是介绍这2个表。

#### Filter表[#](#filter表)

Filter表示iptables的默认表，如果没有自定义表，那么就默认使用filter表，它具有以下三种内建链：

*   **INPUT链** – 处理来自外部的数据。
*   **OUTPUT链** – 处理向外发送的数据。
*   **FORWARD链** – 将数据转发到本机的其他网卡设备上。

#### NAT表[#](#nat表)

NAT表有三种内建链：

*   **PREROUTING链** – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标IP地址（destination ip address），通常用于DNAT(destination NAT)。
*   **POSTROUTING链** – 处理即将离开本机的数据包。它会转换数据包中的源IP地址（source ip address），通常用于SNAT（source NAT）。
*   **OUTPUT链** – 处理本机产生的数据包。

#### Mangle表[#](#mangle表)

Mangle表用于指定如何处理数据包。它能改变TCP头中的QoS位。Mangle表具有5个内建链：

*   **PREROUTING**
*   **OUTPUT**
*   **FORWARD**
*   **INPUT**
*   **POSTROUTING**

#### Raw表[#](#raw表)

Raw表用于处理异常，它具有2个内建链：

*   **PREROUTING chain**
*   **OUTPUT chain**

### 数据包流向[#](#数据包流向)

下图简要描述了网络数据包通过 **iptables** 的过程：

[![](https://img2020.cnblogs.com/blog/663847/202006/663847-20200625160808682-1494987004.png)
](https://img2020.cnblogs.com/blog/663847/202006/663847-20200625160808682-1494987004.png)

_(来自https://wiki.archlinux.org/)_

1、数据经由互联网到达 **nat 表的 PREROUTING**；

2、数据到达 **filter表的INPUT**；

3、数据到达 **本机**；

4、数据经由 **filter表的OUTPUT**；

5、数据经由 **nat 表的 POSTROUTING**；

6、数据到达互联网。

7、我们也可以在数据到达 **nat 表的 PREROUTING**后，使用 **filter表的FORWARD**进行转发。

理解数据包流向很重要，这样我们就能很清楚的知道在哪个表哪个链增加对应的规则。

安装iptables[#](#安装iptables)
--------------------------

iptables在大多数Linux系统上默认安装，可以在命令行输入`iptables --version`查看：

```null
iptables --version
iptables v1.6.1

```

如果没有该命令，可以自行安装：

```null

sudo apt-get install iptables
sudo apt-get install iptables-persistent


sudo yum –y install iptables-services




sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl mask firewalld

sudo yum –y install iptables-services
sudo systemctl enable iptables
sudo systemctl start iptables

```

如果需要在 docker 上体验 iptables ：

```null

docker pull daocloud.io/library/centos:centos6.9
docker tag daocloud.io/library/centos:centos6.9 centos6.9

docker run -it --privileged --name centos-1  centos6.9 /bin/bash

yum install initscripts

```

配置命令[#](#配置命令)
--------------

iptables命令参数非常多。我们分为：配置命令、匹配条件、动作选项、模块选项，这样便于记忆。

*   **配置命令** 指的`ACDIF`等命令，都是大写开头的简称，且一般不可省略，例如`-A`新增规则、`-D`删除规则。
    
*   **匹配条件** 一般是小写，例如`-t`指定table，`-i`指定网卡接口，这些条件一般是可选的。
    
*   **动作选项** 指的是`-j`指定的动作，可选值有：DROP、ACCEPT等。
    
*   **模块选项** 指的是`-m`指定的参数。
    

本节主要介绍配置命令。

iptables配置命令如下所示：

```null
iptables [option] CHAIN_rule [-j target]

```

以下是一些常用iptables选项的列表：

*   **`–A, ––append`** 将规则添加到链中（最后）。
*   **`–I, ––insert`** 将规则添加到给定位置的链中。
*   **`–C, ––check`** 寻找符合链条要求的规则。
*   **`–D, ––delete`** 从链中删除指定的规则。
*   **`–F, ––flush`** 删除对应表的所有规则，**慎重使用**。
*   **`–L, ––list`** 连锁显示所有规则。
*   **`–v, ––verbose`** 使用列表选项时显示更多信息。
*   **`-P, --policy`** 设置链的默认策略(policy)
*   **`-N, --new`** 创建用户自定义链
*   **`-X, --delete-chain`** 删除用户自定义链
*   **`-E, --rename-chain`** 重命名用户自定义链

iptables区分大小写，因此请确保使用正确的选项。更多参数可以输入`iptables --help`查看。

### 查看规则(-L)[#](#查看规则-l)

```null

iptables –L



iptables -t filter –L

```

上述命令执行后，系统显示链条的状态。输出将列出三个链：

```null
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       all  
DROP       all  

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

```

FORWARD、OUTPUT为空，没有定义规则；INPUT有三条规则：禁止来自 172.18.0.4 -172.18.0.5 任何协议的请求，相当于加了黑名单。

```null




iptables -vnL --line-numbers

```

输出示例：

```null
Chain INPUT (policy ACCEPT 9 packets, 621 bytes)
num   pkts bytes target     prot opt in     out     source               destination       
1        0     0 DROP       all  
2        0     0 DROP       all  

...

```

### 新增规则(-A)[#](#新增规则-a)

```null
-A <链名>
    APPEND，追加一条规则（放到最后）

```

示例：

```null



iptables -A INPUT -i eth0 -s 172.18.0.3 -j DROP

```

上面的命令会在 filter 表的 INPUT 链里追加一条规则：对于来自`172.18.0.3`经由 `eth0`接口的数据，直接丢弃。

我们可以在另一台机器 `172.18.0.3`访问当前机器：

```null
$ ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
...

```

发现访问被拒绝了。

```null

iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT

```

### 删除规则(-D)[#](#删除规则-d)

可以按规则号或者规则内容匹配。默认`filter`表。

```null
-D <链名> <规则号码 | 具体规则内容>
    DELETE，删除一条规则

```

先查看规则：

```null
$ iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       all  --  172.18.0.4           anywhere            
2    DROP       all  --  172.18.0.5           anywhere            
3    DROP       all  --  172.18.0.6           anywhere
4    DROP       all  --  172.18.0.3           anywhere
...

```

**1、按规则号(num)删除规则3：** 

将删除 filter 表 INPUT 链中的第三条规则（不管它的内容是什么）

```null
$ iptables -D INPUT 3

$ iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       all  --  172.18.0.4           anywhere            
2    DROP       all  --  172.18.0.5           anywhere            
3    DROP       all  --  172.18.0.3           anywhere
...

```

**2、按内容匹配删除：** 

​ 删除 filter 表 INPUT 链中内容为`-s 172.18.0.5 -i eth0 -j DROP`的规则 （不管其位置在哪里）

```null
$ iptables -D INPUT -s 172.18.0.5 -i eth0 -j DROP

$ iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       all  --  172.18.0.4           anywhere            
2    DROP       all  --  172.18.0.3           anywhere
...

```

建议还是指定规则号删除，以免误删。

### 插入规则(-I)[#](#插入规则-i)

最终也是实现规则的新增。需要指定规则号，默认的规则号是1，也就是插入到第1条。默认`filter`表。

```null
-I <链名> [规则号码]
    INSERT，插入一条规则

```

```null
$ iptables -I INPUT 3 -s 172.18.0.6  -j DROP

$ iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       all  --  172.18.0.4           anywhere            
2    DROP       all  --  172.18.0.3           anywhere            
3    DROP       all  --  172.18.0.6           anywhere
...

```

注意：

*   规则号码必须紧跟着 `-I INPUT`。
*   规则号码如果缺省，那么就是1，插入的规则成为第1条规则。
*   规则号码最大值是已有规则数+1。

小结：`-A`是在所有的规则的最后面增加规则，而`-I`可以在任意位置增加规则。

### 替换规则(-R)[#](#替换规则-r)

```null
-R <链名> <规则号码> <具体规则内容>
    REPLACE，替换一条规则

```

命令格式和`-I`类似。

```null
$ iptables -R INPUT 3 -s 172.18.0.6 -j ACCEPT
$ iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       all  --  172.18.0.4           anywhere            
2    DROP       all  --  172.18.0.3           anywhere            
3    ACCEPT     all  --  172.18.0.6           anywhere
... 

```

### 设置默认规则(-P)[#](#设置默认规则-p)

用于某个链的默认规则。当数据包没有被规则列表里的任何规则匹配到时，按此默认规则处理。可选值： DROP、ACCEPT。

```null
-P <链名> <动作>
    POLICY，设置某个链的默认规则

```

两种默认策略导致的不同结果：

*   默认策略DROP，需要逐条允许。一般服务器上都是这么设置的，仅开放部分端口或者白名单，其他全部拒绝。
*   默认策略ACCEPT，需要逐条拒绝。

配置默认链策略为拒绝：

```null
iptables -P INPUT DROP 
iptables -P FORWARD DROP 
iptables -P OUTPUT DROP 

```

如果防火墙之前没有任何配置，当上面的命令执行后，如果你当前是使用`ssh`连接的服务器，你会发现你被踢出来了，然后连不上了。。。

所以：**远程设置时不要一开始就写这几句默认策略，会把自己踢出服务器，应该最后设定**。

### 清除规则(-F)[#](#清除规则-f)

清除所有规则。可以指定表、链。默认`filter`表。

```null
iptables -F

```

将清除`fliter`表链中所有规则，但并不影响 `-P` 设置的默认规则。其它示例：

```null

iptables -t filter -F INPUT

```

*   `-P` 设置了 `DROP` 后，使用 `-F` 一定要小心。因为相当于把所有ACCEPT的都清除了，会导致ssh连接不上。
*   如果不写链名，默认清空某表里所有链里的所有规则。

### 保存规则[#](#保存规则)

用iptables命令写的规则都是立即生效的，但只是临时的。

我们可以使用`iptables-save`查看系统现在有哪些规则。该命令不同于`iptables -L`，可以打印出所有的规则，而且与我们输入的是一样的：

```null
$ iptables-save


*nat
:PREROUTING ACCEPT [3817:370531]
:INPUT ACCEPT [2:168]
:OUTPUT ACCEPT [56:3864]
:POSTROUTING ACCEPT [112:7728]
:DOCKER_OUTPUT - [0:0]
:DOCKER_POSTROUTING - [0:0]
-A PREROUTING -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.18.0.3:80 
-A PREROUTING -p tcp -m tcp --dport 81 -j DNAT --to-destination 172.18.0.3:80 
... 
*filter
:INPUT ACCEPT [3278:275037]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [3278:275037]
...

```

该命令执行后并不会保存，我们可以直观的看到各表有哪些链、哪些规则。需要注意的是打印出的规则省略了`iptables -t xxx`命令。

CentOS6.9可以使用`service iptables save`命令保存临时防火墙规则：

```null
$ service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]

```

该命令保存的内容就是我们使用`iptables-save`打印出来的内容。

如果系统使用的systemd，上述的操作可能无效。

其他系统参考：

*   [ubuntu18](https://www.cnblogs.com/xwgcxk/p/10820518.html)
*   [archlinux](https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

匹配条件选项[#](#匹配条件选项)
------------------

### \-p 协议类型 (--protocol)[#](#-p-协议类型----protocol)

用于指定规则的协议，如tcp, udp, icmp等，可以使用`all`来指定所有协议。如果不指定`-p`参数，则默认是`all`值。

也可以使用协议名(如tcp)，或者是协议值（比如6代表tcp）来指定协议。映射关系请查看`/etc/protocols`。

```null

iptables -A INPUT -s 172.18.0.3 -p icmp  -j DROP

```

> 注意：因为禁用了从172.18.0.3(简称B机器)来的icmp协议，那么当前机器也不能ping通机器B，原因是收不到响应数据。但是机器B确实能收到请求，可以在机器B上执行抓包`tcpdump -i eth0 icmp`。

### \-s 源地址 (--source)[#](#-s-源地址---source)

用于指定数据包的源地址。参数可以使IP地址、网络地址、主机名。例如：

*   `-s 192.168.0.1` 匹配来自 192.168.0.1 的数据包
*   `-s 192.168.1.0/24` 匹配来自 192.168.1.0/24 网络的数据包
*   `-s 192.168.0.0/16` 匹配来自 192.168.0.0/16 网络的数据包
*   `! -s 192.168.1.101`除这个IP外

如果不指定`-s`参数，就代表所有地址。

> `192.168.1.10/24`代表192.168.1.0-192.168.1.255网段，24表示网络号占用24位二进制，剩余8位二进制可以表示 2^8位主机。可以使用https://www.sojson.com/convert/subnetmask.html 工具计算网段。

### \-d 目的地址 (--destination)[#](#-d-目的地址---destination)

用于指定目的地址。参数和`-s`相同。

例如：

*   `-d 202.106.0.20` 匹配去往 202.106.0.20 的数据包
*   `-d 202.106.0.0/16` 匹配去往 202.106.0.0/16 网络的数据包
*   `-d www.abc.com` 匹配去往域名 www.abc.com 的数据包

### \-j 执行目标 (--jump)[#](#-j-执行目标---jump)

用于指定target。可能的值是ACCEPT, DROP, QUEUE, RETURN。也可以指定链(Chain)作为目标。后文会细讲。

### \-i 输入接口 (--in-interface)[#](#-i-输入接口---in-interface)

用于指定要处理来自哪个接口的数据包，这些数据包即将进入INPUT, FORWARD, PREROUTE链。

例如：

*   `-i eth0`指定了要处理经由`eth0`进入的数据包。
*   如果不指定`-i`参数，那么将处理进入所有接口的数据包。
*   如果出现`! -i eth0`，那么将处理所有经由`eth0`以外的接口进入的数据包。
*   如果出现`-i eth +`，那么将处理所有经由`eth`开头的接口进入的数据包。

### \-o 输出 (--out-interface)[#](#-o-输出---out-interface)

用于指定数据包由哪个接口输出，这些数据包即将进入FORWARD, OUTPUT, POSTROUTING链。

*   如果不指定`-o`选项，那么系统上的所有接口都可以作为输出接口。
*   如果出现`! -o eth0`，那么将从`eth0`以外的接口输出。
*   如果出现`-i eth +`，那么将仅从`eth`开头的接口输出。

### \--sport 源端口 (--source-port)[#](#--sport-源端口---source-port)

针对`-p tcp` 或者 `-p udp`，不可单独使用。

可以指定端口号或者端口名称，例如`–sport 22`与`–sport ssh`，`/etc/services`文件描述了上述映射关系。但从性能上讲，使用端口号更好。

使用冒号可以匹配端口范围，如`–sport 22:100`。

例如：

```null
--sport 1000       匹配源端口是 1000 的数据包
--sport 1000:3000  匹配源端口是 1000-3000 的数据包（含1000、3000）
--sport :3000      匹配源端口是 3000 以下的数据包（含 3000）
--sport 1000:      匹配源端口是 1000 以上的数据包（含 1000）

```

```null

iptables -A INPUT -p tcp --sport 80 -j ACCEPT

```

### \--dport 目的端口 (--destination-port)[#](#--dport-目的端口---destination-port)

针对`-p tcp` 或者 `-p udp`。参数和`–sport`类似。

```null

iptables -A INPUT -p tcp --dport 80 -j ACCEPT

```

### \--tcp-flags TCP标志[#](#--tcp-flags-tcp标志)

针对`-p tcp` 。可以指定由逗号分隔的多个参数。有效值可以是：SYN, ACK, FIN, RST, URG, PSH。可以使用`ALL`或者`NONE`。

### \-–icmp-type ICMP类型[#](#-icmp-type-icmp类型)

针对`-p icmp`。

*   `–icmp-type 0` 表示 pong
*   `–icmp-type 8` 表示 ping

动作选项[#](#动作选项)
--------------

`-j`参数支持下列动作选项：

*   ACCEPT
*   DROP
*   DNAT
*   SNAT
*   MASQUERADE
*   RETURN

### ACCEPT[#](#accept)

允许数据包通过本链而不拦截它。

例如：

```null

iptables -A INPUT -s 172.18.0.3 -j ACCEPT

```

### DROP[#](#drop)

阻止数据包通过本链，直接丢弃它。

例如：

```null

iptables -A INPUT -s 172.18.0.3 -j DROP

```

### DNAT[#](#dnat)

目的地址转换。

1、原理：在路由前（PREROUTING）将来自外网访问网关公网ip及对应端口的目的ip及端口修改为内部服务器的ip及端口，实现发布内部服务器。

2、应用场景：发布内部主机服务。

3、设置DNAT：网关主机上设置。

DNAT 支持转换为单 IP，也支持转换到 IP 地址池（一组连续的 IP 地址）。

编写防火墙规则：

```null
-j DNAT --to-destination 内网服务ip:端口

```

> `--to-destination`等同于`--to`。

例如：

```null

iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.18.0.3


iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 81 -j DNAT --to 172.18.0.3:80
    

iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.18.0.3-172.18.0.10

```

`DNAT`作用于`PREROUTING`链，如果写`POSRTOUTING`，会报错。

### SNAT[#](#snat)

源地址转换。

1、原理：在路由器后（POSTROUTING）将内网的ip地址修改为外网网卡的ip地址。

2、应用场景：共享内部主机上网。

3、设置SNAT：网关主机进行设置。

源地址转换即内网地址向外访问时，发起访问的内网ip地址转换为指定的ip地址（可指定具体的服务以及相应的端口或端口范围），这可以使内网中使用保留ip地址的主机访问外部网络，即内网的多部主机可以通过一个有效的公网ip地址访问外部网络。

SNAT 支持转换为单 IP，也支持转换到 IP 地址池（一组连续的 IP 地址）。

```null
-j SNAT --to IP[-IP][:端口-端口]（nat 表的 POSTROUTING 链）

```

例如：

```null

iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j SNAT  --to 172.18.0.2 

iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j MASQUERADE 


iptables -t nat -A POSTROUTING -d 172.18.0.1/24  -j SNAT --to 1.1.1.1


iptables -t nat -A POSTROUTING -d 172.18.0.1/24  -j SNAT --to 1.1.1.1-1.1.1.10

```

### MASQUERADE[#](#masquerade)

动态源地址转换（动态 IP 的情况下使用）。

上面的小节里，配置SNAT的时候：

```null
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j SNAT  --to 172.18.0.2

```

需要指定映射的目标地址`-to 172.18.0.2`。也可以使用`MASQUERADE`动态获取：

```null
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j MASQUERADE

```

如果IP地址不是固定的，使用这种方式会比较方便。

### RETURN[#](#return)

返回主链继续匹配。

比如：

```null

iptables -t nat -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER


iptables -t nat -A DOCKER -i docker0 -j RETURN

```

这两条规则将改变原有的数据包流向，会在 PREROUTING 前增加一个 DOCKER 链：

[![](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626162644001-1090144804.png)
](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626162644001-1090144804.png)

> 上图详见: [理解 Docker 网络(一) -- Docker 对 宿主机网络环境的影响](https://blog.csdn.net/qq_17004327/article/details/88630194)

模块选项[#](#模块选项)
--------------

`-m`参数支持模块选项。

*   state: 按包状态匹配
*   mac: 按来源 MAC 匹配
*   limit: 按包速率匹配
*   multiport: 多端口匹配
*   iprange: 按IP范围匹配
*   string 按字符串限定

### state[#](#state)

按包状态匹配。

格式：

```null
-m state --state 状态

```

状态：NEW、RELATED、ESTABLISHED、INVALID

*   NEW：新建连接请求的数据包，且该数据包没有和任何已有连接相关联。
*   ESTABLISHED：该连接是某NEW状态连接的回包，也就是完成了连接的双向关联。
*   RELATED：衍生态，与 conntrack 关联。简而言之，A连接已经是ESTABLISHED，而B连接如果与A连接相关，那么B连接就是RELATED。
*   INVALID：匹配那些无法识别或没有任何状态的数据包。这可能是由于系统内存不足或收到不属于任何已知连接的ICMP错误消息，也就是垃圾包，一般情况下我们都会DROP此类状态的包。

例如：

```null

iptables -A INPUT -p tcp -m state --state NEW -j DROP
iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT

```

### conntrack[#](#conntrack)

按包状态匹配。conntrack是state的扩展版本（内核版本>=2.5开始支持），包括状态参数也是基本相同。

状态：NEW、RELATED、ESTABLISHED、INVALID、**UNTRACKED** 。前面的几个上一节介绍了，这里说一下UNTRACKED。

*   UNTRACKED ： 这是一种特殊状态，或者说并不是状态。它是管理员在raw表中，为连接设置NOTRACK规则后的状态。这样做，便于提高包过滤效率以及降低负载。

```null
-m conntrack --ctstate 状态

```

示例：

```null

iptables -A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

```

### mac[#](#mac)

匹配某个 MAC 地址。

格式：

```null
-m mac --mac-source MAC

```

例如：

```null

iptables -A FORWARD -m --mac-source xx:xx:xx:xx:xx:xx -j DROP

```

注意：MAC 地址不过路由，不要试图去匹配路由后面的某个 MAC 地址。

### limit[#](#limit)

启用limit扩展，限制速度。用一定速率去匹配数据包。

格式：

```null
-m limit --limit 匹配速率 [--burst 缓冲数量]
-m limit --limit 25/minute: 允许最多每分钟25个连接
-m limit --limit-burst 100: 当达到100个连接后，才启用上述25/minute限制

```

例如：

```null
iptables -A FORWARD -d 192.168.0.1 -m limit --limit 50/s -j ACCEPT
iptables -A FORWARD -d 192.168.0.1 -j DROP

```

注意：`limit` 仅仅是用一定的速率去匹配数据包，并非 `限制`。

### multiport[#](#multiport)

一次性匹配多个端口，可以区分源端口，目的端口或不指定端口。

格式：

```null
-m multiport <--sports|

```

例如：

```null
iptables -A INPUT -p tcp -m multiports --ports 21,22,25,80,110 -j ACCEPT

```

注意：必须与 `-p` 参数一起使用。

### iprange[#](#iprange)

指定IP范围。

格式：

```null
-m iprange --src-range
-m iprange --dst-range

```

示例：

```null

iptables -A INPUT -m iprange --src-range 192.168.1.2-192.168.1.7 -j DROP


iptables -A INPUT -m iprange --dst-range 192.168.1.2-192.168.1.7 -j DROP

```

### string[#](#string)

按字符串限定。格式：

```null
-m string --string "STRING" --algo kmp 指定字符串本身

```

其中`--algo` 指定匹配算法：bm或kmp。

### addrtype[#](#addrtype)

按地址类型匹配。格式：

```null
-m addrtype --dst-type 地址类型

```

支持的地址类型：

*   UNSPEC
*   UNICAST
*   LOCAL 匹配本机地址
*   BROADCAST 匹配广播地址
*   ANYCAST
*   MULTICAST
*   BLACKHOLE
*   UNREACHABLE
*   PROHIBIT
*   THROW
*   NAT
*   XRESOLVE

> 使用`iptables -m addrtype --help`可以查看类型列表。地址类型博主也没有完全明白都是什么作用，后面遇到了再补。

示例：

```null

iptables -t nat -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER

```

自定义链[#](#自定义链)
--------------

iptables的默认链就已经能够满足我们了，为什么还需要自定义链呢？原因如下：

当默认链中的规则非常多时，不方便我们管理。

例如，如果INPUT链中存放了几十条规则，这几十条规则有针对http服务的，有针对sshd服务的，有针对私网IP的，有针对公网IP的，假如，我们突然想要修改针对http服务的相关规则，难道我们还要从头看一遍这几十条规则，找出哪些规则是针对http的吗？这显然不合理。

所以，iptables中，可以自定义链，通过自定义链即可解决上述问题。

现在，我们自定义一个名为 WEB 的链，实现：

*   转发当前机器80端口流量到172.18.0.3:80
*   80端口流量交由WEB管理；且禁止IP: 192.168.2.194 访问80端口

为了实现上面的规则，需要分别在filter表和nat表增加 自定义链 WEB：

```null


iptables -t filter -N WEB

iptables -t filter -A WEB -s 192.168.2.194 -j DROP

iptables -t filter -A INPUT -p tcp --dport 80 -j WEB



iptables -t nat -N WEB

iptables -t nat -A WEB -p tcp --dport 80 -j DNAT --to 172.18.0.3:80

iptables -t nat -A PREROUTING -p tcp --dport 80 -j WEB


iptables -t nat -A POSTROUTING -p tcp --dport 80 -j SNAT -d 172.18.0.3 --to 172.18.0.2 

```

查看生成的规则：

```null
$ iptables-save

*nat
:PREROUTING ACCEPT [1:73]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [1:72]
:POSTROUTING ACCEPT [1:72]
:WEB - [0:0]
-A PREROUTING -p tcp -m tcp --dport 80 -j WEB 
-A POSTROUTING -d 172.18.0.3/32 -p tcp -m tcp --dport 80 -j SNAT --to-source 172.18.0.2 
-A WEB -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.18.0.3:80 
COMMIT

*filter
:INPUT ACCEPT [1:97]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [1:72]
:WEB - [0:0]
-A INPUT -i lo -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 80 -j WEB 
-A WEB -s 192.168.2.194/32 -j DROP

```

常见规则示例[#](#常见规则示例)
------------------

### 启用环回流量[#](#启用环回流量)

允许来自您自己的系统（本地主机）的流量：

```null
iptables -A INPUT -i lo -j ACCEPT

```

此命令将防火墙配置为接受localhost（**`lo`**）接口（**`-i`**）的通信**。** 现在，源自系统的所有内容都将通过防火墙。需要设置此规则，以允许应用程序与localhost接口通信。

### 简易防火墙[#](#简易防火墙)

该规则默认拒绝所有INPUT链，允许：

*   loop回环网卡流量
*   连接状态为RELATED,ESTABLISHED的连接
*   80, 22, 21 端口

```null
iptables -F
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED  -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP

```

注：如果设置了默认规则`iptables -P OUTPUT DROP`，那么对应的ACCEPT需要增加`-A OUTPUT`的指令：

```null
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED  -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED  -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT DROP

```

### IP黑名单[#](#ip黑名单)

禁止172.18.0.3访问。

```null
iptables -A INPUT -i eth0 -s 172.18.0.3 -j DROP

```

### 端口转发[#](#端口转发)

172.18.0.2 机器80端口流量转发到 172.18.0.3 机器80端口。这样很简单的实现了流量转发。需要在  
172.18.0.2 机器配置规则：

```null


iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to 172.18.0.3:80


iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j SNAT --to 172.18.0.2 



```

### docker防火墙配置[#](#docker防火墙配置)

安装docker后会在宿主机增加规则，实现端口映射、docker内部连接外网。

NAT 表改动的解析如下：

```null

:DOCKER - [0:0]


-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER


-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER


-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE


-A DOCKER -i docker0 -j RETURN

```

FILTER 表改动的解析如下:

```null

:DOCKER - [0:0]


:DOCKER-ISOLATION-STAGE-1 - [0:0]


:DOCKER-ISOLATION-STAGE-2 - [0:0]


:DOCKER-USER - [0:0]


-A FORWARD -j DOCKER-USER


-A FORWARD -j DOCKER-ISOLATION-STAGE-1


-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT


-A FORWARD -o docker0 -j DOCKER


-A FORWARD -i docker0 ! -o docker0 -j ACCEPT


-A FORWARD -i docker0 -o docker0 -j ACCEPT


-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2


-A DOCKER-ISOLATION-STAGE-1 -j RETURN


-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP


-A DOCKER-ISOLATION-STAGE-2 -j RETURN


-A DOCKER-USER -j RETURN

```

可以结合理解的 iptables 知识，对着注释看一遍,，可加深理解。

安装 Docker 后 Docker 添加了 DOCKER， DOCKER-USER， DOCKER-ISOLATION-STAGE-1， DOCKER-ISOLATION-STAGE-2 四条链，如图：

[![](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626164941601-586501223.png)
](https://img2020.cnblogs.com/blog/663847/202006/663847-20200626164941601-586501223.png)

> 本小节来源：[理解 Docker 网络(一) -- Docker 对 宿主机网络环境的影响](https://blog.csdn.net/qq_17004327/article/details/88630194)，作者 [若即](https://blog.csdn.net/qq_17004327/article/details/88630194)，感谢作者的分享。

总结[#](#总结)
----------

一直都想整理 iptables ，但是每次都没有静下心研究。之前也看过这个，但是时间一长就忘记了。这次趁着假期把 iptables 相关的知识都整理了，也拓展了之前未曾接触过的：例如模块选项、DNAT、SNAT等等，回头再去看 [25个iptables常用示例](https://www.linuxprobe.com/25-iptables-common-examples.html) ，不会再像看天书那样了。

iptables 功能真是太强大了，实现了常用的IP屏蔽、数据包转发、端口转发等功能。docker就是基于iptables实现端口转发的。

本文涉及知识点特别多，建议初学者先全文先看一遍，然后再细看，这样可以加深理解。文中给了很多示例，读者最好可以在电脑上实践一下。

参考[#](#参考)
----------

1、iptables (简体中文) - ArchWiki  
[https://wiki.archlinux.org/index.php/Iptables\_(简体中文)](https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

2、iptables详细教程：基础、架构、清空规则、追加规则、应用实例 - Lesca 技术宅  
[https://lesca.me/archives/iptables-tutorial-structures-configuratios-examples.html](https://lesca.me/archives/iptables-tutorial-structures-configuratios-examples.html)

3、25个iptables常用示例 | 《Linux就该这么学》  
[https://www.linuxprobe.com/25-iptables-common-examples.html](https://www.linuxprobe.com/25-iptables-common-examples.html)

4、两小时玩转iptables.ppt

[https://kdocs.cn/l/sQ2wks924](https://kdocs.cn/l/sQ2wks924)

5、看了那么多iptables的教程，这篇教程还是比较全面易懂的 - 91云(91yun.co)  
[https://www.91yun.co/archives/1690](https://www.91yun.co/archives/1690)

6、iptables详解（10）：iptables自定义链 - wanstack - 博客园  
[https://www.cnblogs.com/wanstack/p/8393282.html](https://www.cnblogs.com/wanstack/p/8393282.html)

7、同局域网下的 Iptables DNAT  
[http://wsfdl.com/踩坑杂记/2017/01/12/iptables\_snat.html](http://wsfdl.com/%E8%B8%A9%E5%9D%91%E6%9D%82%E8%AE%B0/2017/01/12/iptables_snat.html)

8、Linux通过iptables端口转发访问内网服务器上的内网服务 - 看天博客  
[http://hi.ktsee.com/635.html](http://hi.ktsee.com/635.html)

9、理解 Docker 网络(一) -- Docker 对 宿主机网络环境的影响\_若即的专栏-CSDN博客\_docker访问宿主机网络  
[https://blog.csdn.net/qq\_17004327/article/details/88630194](https://blog.csdn.net/qq_17004327/article/details/88630194)