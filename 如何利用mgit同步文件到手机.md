# 如何利用mgit同步文件到手机
最近公司系统部同事找到我，说电脑上有个程序一直在读取windows目录下的某个文件，还给了我截图看，说是云服务之类的。

我猜是syncthing在同步文件。

syncthing一个开源的分布式文件同步工具，只要在同一个网络下，安装好了syncthing，无论你是手机，平板，台式机，笔记本都可以实时同步。

公司不允许，文件同步的梦想泡汤了，只能另找出路。

经过多日调研和尝试，找到了一个更好的方案。

采用github作为远程中转的集散地，nas，笔记本，台式机，手机在各自的内网去和github交互就可以了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyKzwvJrMamUibkrD8h2RZU2ooVroI4UYgBJEw0Q4am85JBO9sF7SOicmzA/640?wx_fmt=png&from=appmsg)

使用github有众多好处，

1.  免费备份，5GB的容量做知识管理够用了
    
2.  暴露在公网，其它客户端都可以直连同步文件
    
3.  git做版本控制，多端冲突易解决
    
4.  比较稳定，几乎不会挂掉
    

有盆友可能会问为什么不用NAS自建git服务，这块维护成本有点高，需要搭建数据库服务，配置SSL等，在使用频率较高的情况下，稍有不慎，服务就不能用了，最后还是选了github，我们使用NAS专门做镜像备份，还可以触发webhook自动化部署到hugo。

方案定好了，接下来面临的第一大难题是，如何让各个设备都装上git客户端。

桌面端可以下载git bash，NAS自带git client，唯有移动端是个大坑。

找来找去，最终在github上找到了MGIT，开源免费，完美解决了这个问题，不得不大赞github。

关于MGIT的安装和使用，网上资料比较少，这也是本篇的重点，笔者花了些时间，记录了一下。

> https://github.com/maks/MGit

根据官方的说明，MGIT可以在google play上下载，但是国内厂商的google play是阉割版本，用起来比较麻烦。

官方还提供了第二种下载方式fdroid，这种方式相对简单.

fdroid就像一个应用市场，只需要映射到清华大学镜像源，就可以安装国外开发者开发的应用。

接下来和我一起操作吧。

### 下载fdroid应用

首先我们去github这个地址下载fdroid。

> https://github.com/f-droid/fdroidclient/releases

![](https://mmbiz.qpic.cn/sz_mmbiz_png/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyK13rVGWibX7IKyFq78AYHIUaSjwBQTa9DHFYBgRmYTjeyY5cPiblfpV3w/640?wx_fmt=png&from=appmsg)

选择相对稳定的版本，下载好了，在手机上找到该文件，点击安装。

### 更换镜像源

安装好了fdriod后，点击”设置“，找到”存储库“，把自带的镜像源关掉，新增清华大学镜像源，后面安装包都来自这个地址。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyKcktn2dibACyE1OvpWv3bLL17sicOhCyxyMpzrc3N2ksDlobeEFd34gVw/640?wx_fmt=jpeg&from=appmsg)

清华大学镜像源站点fdriod的使用说明在下面这个地址

> https://mirrors.tuna.tsinghua.edu.cn/help/fdroid/

![](https://mmbiz.qpic.cn/sz_mmbiz_png/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyK7ca8j3uZdVfoIRBnHfiaPxduLsJgUzLjXulhHItXU7Rbqj7zKFxZLCA/640?wx_fmt=png&from=appmsg)

> https://mirrors.tuna.tsinghua.edu.cn/fdroid/repo/?fingerprint=43238D512C1E5EB2D6569F4A3AFBF5523418B82E0A3ED1552770ABB9A9C9CCAB

只需要添加这个镜像存储库就可以了。

### 安装mgit

添加镜像源后，点击搜索，找到mgit，查看mgit的最新版本是不是v1.7.0。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyKxuJn5omwkEicWgrAbZ9A3Thy2Ocia5p0pc9NRibicghzKyibCjeZ3YOW8dw/640?wx_fmt=jpeg&from=appmsg)

如果是低版本的apk，可能会有些bug，比如用其他工具新增文件后，mgit没有权限会检测不到新增的文件，因此无法纳入版本库。

点击安装，授权mgit文件所有权权限，接下来就可以拉取仓库了。

### 拉取仓库

打开mgit，点击右上角，找到克隆仓库，可以看到如下界面，

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyKn1UhyM4SpwC8rlYwajdnEN41nlXgyn2zdzicffM8W914rTAg5jV57Wg/640?wx_fmt=jpeg&from=appmsg)

这里可以填写远程仓库地址和要映射的本地地址，远程仓库地址我们采用https协议，把用户名和密码统一写在url里，格式是这种的，

> https://username:password@github.com/xxx.git

clone完毕后就可以正式使用了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/oO3eAEXfZTUiaJpsV19cNUwTv7JTQuxyKiaR5Ow2dvQfgL1TfQElqlQNYwflJKgicTID8yTSmUqjthoTwd9CQeXKA/640?wx_fmt=jpeg&from=appmsg)

这个界面是不是很清爽，该有的功能都有了。

每次编辑前，先用mgit拉取最新版本，编辑结束后，再用mgit推送到远程仓库，完美解决同步文件到移动端的问题。

还在等什么，赶快行动起来吧。