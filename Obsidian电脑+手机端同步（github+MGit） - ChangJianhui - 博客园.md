# Obsidian电脑+手机端同步（github+MGit） - ChangJianhui - 博客园
| 名称 | 版本 |
| --- | --- |
| Obsidian | 1.5.3 |
| 华为HarmonyOS | 4.0.0 |
| Git | 2.43.0-64-bit |
| MGit | 1.7.0(手机端) |
| F-Droid | apk(手机端) |

> 安装F-Droid是因为我的手机是HarmonyOS系统，没有谷歌框架，无法通过google play安装MGit，所以下载F-Droid来安装MGit.

百度网盘：[https://pan.baidu.com/s/1Y63MKoXdzk\_pRuAaiB8AYg?pwd=qevw](https://pan.baidu.com/s/1Y63MKoXdzk_pRuAaiB8AYg?pwd=qevw) 提取码：qevw  
123云盘（不限速）：[https://www.123pan.com/s/CyY6Vv-QBXJ.html](https://www.123pan.com/s/CyY6Vv-QBXJ.html%E6%8F%90%E5%8F%96%E7%A0%81:5yvN)提取码:5yvN

下载安装Obsidian
------------

到官网或者Obsidian的中文论坛下载安装包，安装完成在本地新建一个仓库，我这里的仓库名字是`obsidian`，后续以这个仓库为例进行说明。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203548030-267225076.png)

默认的插件市场是需要VPN的，如果有插件的需要可以用pkmer这个插件市场下载插件，这个不需要VPN

`pkmer`链接: [PKMer](https://pkmer.cn/)

安装git
-----

安装包可以去官网下载或者使用我上面给的链接

安装好后要找到这个应用，一会儿要用到

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203548859-1503935772.png)

使用github
--------

首先要注册一个github账号，记住自己的`用户名`和`注册时使用的邮箱`，这个注册的教程网上可以查阅，这里就不写了。

点击右上角的头像->再点击Settings，在这里可以看到和找到自己的用户名和邮箱。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203549527-771969494.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203550051-616584741.png)

使用git将本地仓库`obsidian`上传到github中
------------------------------

我这里只给出这个过程要用到的命令，如果需要了解git如何使用可以学习廖雪峰老师的教程

git教程（廖雪峰）：[Git教程 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/896043488029600)

✨开始之前：

将本地仓库上传到github中需要进行身份认证，过程如下：

*   指定用户名和Email地址
    
*   在本地用命令行生成一对密钥（包括公钥和私钥）
    
*   将生成的公钥用记事本打开，复制后粘贴到github中存放公钥的地方
    

指定用户名和Email地址：

首先打开git bash, 然后进行命令行操作

```null
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"

```

生成一对密钥：

```null
$ ssh-keygen -t rsa -C "youremail@example.com"

```

你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。

将公钥`id_rsa.pub`用记事本打开，复制后粘贴到github中存放公钥的地方：

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203550519-616881552.png)

点击`NEW SSH KEY`进行粘贴

✨现在在github上创建一个仓库，用来存放我们要上传的文件：

点击右上角的+号->选择`New repository`

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203551054-972504879.png)

我这里的仓库名字是`obsidian`和本地仓库名相同，属性选择私有，点击`Create repository`

注意：这里除了这两个地方需要改动，不要动其他的选项

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203551530-720185533.png)

✨初始化本地仓库：

通过`git init`命令把这个目录变成Git可以管理的仓库

进入到刚刚创建的本地仓库`obsidian`

```null
$ cd e:\obsidian

```

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203551846-1640833816.png)

一定要进入obsidian仓库里面，再执行初始化命令

初始化obsidian仓库

```null
$ git init

```

通过`git init`命令把这个目录变成Git可以管理的仓库.

✨将本地的obsidian仓库与github上的obsidian仓库进行关联：

```null
$ git remote add origin git@server-name:path/repo-name.git

```

`server-name`是自己的github用户名，`repo-name`是刚刚在github上创建的仓库名字。

✨将本地仓库的内容推送到GitHub仓库

在推送之前，我们需要先做一个提交申请，这是由于git的工作机理要求。具体可以参看git教程。

这里先创建一个txt格式文件，用记事本创建，并写入内容Here are my obsidian notes.

保存为叫readmetxt的txt格式文件。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203552137-668957195.png)

添加文件到Git仓库，分两步：（这里要在obsidian本地仓库中进行）

第一步

```null
$ git add readmetxt.txt

```

第二步

```null
$ git commit -m "add files."

```

"add files."这个代表一个提交说明，可自定义，但要用引号。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203552496-604557523.png)

将本地仓库的内容推送到GitHub仓库：

```null
$ git push -u origin main

```

由于远程库是空的，我们第一次推送`main`分支时，加上了`-u`参数，Git不但会把本地的`main`分支内容推送的远程新的`main`分支，还会把本地的`main`分支和远程的`main`分支关联起来，在以后的推送或者拉取时就可以简化命令。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203552888-355411266.png)

这里你可能会遇到下面说的问题：

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203553323-1736803583.png)

注意：Are you sure you want to continue connecting (yes/no)?这里一定要输入`yes`，不能直接回车，因为回车代表no

上面的过程可以看出，想使用git提交到远程仓库需要先添加后推送。

添加文件到Git仓库，分两步：

1.  使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
2.  使用命令`git commit -m <message>`，完成。

推送可以使用命令`git push origin mainr`一次性完成，但是添加文件如果很多则很繁琐，可以搭配git提交的图形化界面来便捷完成。

Sourcetree的使用
-------------

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203554378-1027957581.png)

下载安装后，打开Sourcetree.

第一次运行SourceTree时，SourceTree并不知道我们的Git库在哪。如果本地已经有了Git库，直接从资源管理器把文件夹拖拽到SourceTree上，就添加了一个本地Git库：

我们双击`obsidian`这个本地库，SourceTree会打开另一个窗口，展示这个Git库的当前所有分支以及文件状态。选择左侧面板的“WORKSPACE”-“File status”，右侧会列出当前已修改的文件（Unstaged files）：

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203554945-889931500.png)

✨在使用之前首先要导入公钥，步骤如下

工具->选项

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203555328-365367125.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203555748-1496302737.png)

这里的id\_rsa是公钥

✨使用sourcetree进行提交文件

点击文件状态->暂存所有

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203556182-1520315454.png)

在下面文本框输入提交说明，再点击提交

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203556582-1801237431.png)

提交成功后点击推送

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203557095-1898978139.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203557675-31049123.png)

下面显示的就是推送成功了

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203558158-1270832418.png)

所用软件
----

*   obsidian app
*   MGit

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203558623-1736371061.jpg)

MGit设置
------

MGit配置步骤：

1.  在设置界面的「repos 的根存储位置」拟用于存放 Android 设备上 Obsidian 笔记的路径
2.  在设置页面，点击「SSH Keys」>「+」，新建 SSH 密钥
3.  自己用英文命名密钥的文件名—>点击生成密钥—>将之前的公钥文件内容复制到这里

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203559139-1363841022.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203559904-487200636.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203600519-1283750237.jpg)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203601041-318857767.jpg)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203601579-401832317.png)

4.  从GitHub 复制 SSH Remote 地址（即如下图所示的地址，也可以在仓库主页面点击「下载/克隆」（GitHub 点击「Code」）查看），填入远程地址，点击克隆。

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203602130-701540406.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203602555-1201812304.png)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203602888-1218282374.png)

成功之后，打开 Obsidian for Android。一般来说，Obsidian自动扫描到你克隆到手机的工作空间。如果没有扫描到，那么手动从 Obsidian 进入上述步骤设置的仓库路径，用作工作空间即可。

注意： 从手机端获取github上的文件是需要先通过MGit拉取的

MGit拉取文件步骤
----------

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203603354-670102056.jpg)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203603915-857998952.jpg)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203604441-142373360.jpg)

![](https://img2023.cnblogs.com/blog/3252111/202402/3252111-20240205203605467-1394461615.jpg)

*   [用 Git 在 Android 和 Windows 间同步 Obsidian 数据库 - 少数派 (sspai.com)](https://sspai.com/post/68989#!)
*   [用Mgit在Android上通过git来同步&Pull ( Mgit初始化 ) (更推荐用obsidian-git)(至今个人暂时不使用同步功能) - 经验分享 - Obsidian 中文论坛](https://forum-zh.obsidian.md/t/topic/7152)
*   [obsidian和Git连用实现版本控制（obsidian Git插件介绍） by 软通达 - Obsidian中文教程 - Obsidian Publish](https://publish.obsidian.md/chinesehelp/01+2021%E6%96%B0%E6%95%99%E7%A8%8B/obsidian%E5%92%8CGit%E8%BF%9E%E7%94%A8%E5%AE%9E%E7%8E%B0%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%EF%BC%88obsidian+Git%E6%8F%92%E4%BB%B6%E4%BB%8B%E7%BB%8D%EF%BC%89+by+%E8%BD%AF%E9%80%9A%E8%BE%BE)
*   [Git教程 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/896043488029600)

这个是我为了实现obsidian在电脑端编辑，同时为了方便查看，又在手机端实现了同步，这个教程参考了他人的配置教程，给了我很多帮助，感谢这些贡献者！🌹🌹🌹