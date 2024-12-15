# 手机离线跑AI的最佳选择：ChatterUI
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reEShcGg7LAPRFTYvayY7reVqMJ1EUH80pWYq1Wz6Na8x5zrO5RMibusstw/640?wx_fmt=jpeg)

在众多手机本地运行AI的应用程序中，ChatterUI以其卓越的性能和用户体验脱颖而出，成为我个人最推荐的选择。以下是其突出优点：

1\. TTS语音朗读对话：ChatterUI支持文本转语音（TTS）功能，能够将对话内容朗读出来，提供更便捷的交互方式。

2\. 美观排版与重点高亮：该应用的界面设计简洁美观，对话内容排版清晰易读，同时支持对重点部分进行高亮显示，便于用户快速抓住关键信息。

3\. 重新生成答案：ChatterUI允许用户对不满意的答案进行重新生成，提供更准确或更符合需求的回答。

4\. 对话内容可复制：用户可以轻松复制对话内容，便于后续使用或分享。

5、支持输出Markdown表格

令人惊喜的是，ChatterlUI在旧手机小米9（搭载骁龙855处理器和8GB RAM）上能够流畅运行3B模型（模型文件大小约为2GB）。这得益于bartowski老哥对q4质量模型的优化，使其能够更好地适应手机的Arm芯片架构。

对于bartowski老哥的q4质量模型，我有以下推荐：

\- 6-8GB RAM手机：可以流畅运行以下模型：

bartowski/Qwen2.5-3B-Instruct-GGUF

bartowski/Llama-3.2-3B-Instruct-GGUF

\- 12GB RAM手机：可以流畅运行以下模型：

bartowski/Qwen2.5-7B-Instruct-GGUF

bartowski/Ministral-8B-Instruct-2410-GGUF

根据我个人的使用经验，对于bartowski老哥的q4质量模型，手机RAM大小与可流畅运行的模型文件大小之间存在一定的关系。具体而言，手机RAM大小除以2再减去1-2GB，即可得到可流畅运行的模型文件大小。

以8GB RAM手机为例，根据上述公式（8/2-1=3），可以轻松运行3GB大小的模型。

最后是3B模型的表现：

中文的是Qwen 2.5；英文的是Llama 3.2

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESicXZXyhdttS2aOicy2kJ9c7CySDI5c5tpE4Wg6O5SkgrXiaOI641gGpvw/640?wx_fmt=jpeg)

Llama 3.2的中文不好，而且上面的问题也不能答对。但是呢，Llama 3.2也很多优点，比如理解复杂的提示词，英语知识库的质量也比较高，不管是健康知识还是其它知识，感觉都比较靠谱。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESSArH9omxtGJVTWBAnsQUqYWnAlaEnmoOEjzCe2gkYMbEiaXkLV06zrg/640?wx_fmt=jpeg)

上图：知识靠谱性比较

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reES5o5ZW847QBreDzMqckFUGqOngOAao4iaTWMbawUoXHasx0AUAv3jxuw/640?wx_fmt=jpeg)

上图：复杂提示词的理解比较

然后阿里巴巴的Qwen2.5已经推出数学专家模型，代码专家模型，1.5B的模型数学也很好，支持中英文混合输入，大家可以试试用手机离线做数学题😀

![](https://mmbiz.qpic.cn/sz_mmbiz_png/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESVGofEngmwD5KTzLzWEollYTpHoYORTHqzrGwGZAnRmu19l43MaQYTQ/640?wx_fmt=png)

\- 编程专家模型：

bartowski/Qwen2.5-Coder-1.5B-Instruct-GGUF

bartowski/Qwen2.5-Coder-7B-Instruct-GGUF

\- 数学专家模型：

bartowski/Qwen2.5-Math-1.5B-Instruct-GGUF

bartowski/Qwen2.5-Math-7B-Instruct-GGUF

如何下载模型文件？

复制文件全名，直接谷歌搜索就行。

网络不好的朋友可以去镜像网站下载：

https://hf-mirror.com/bartowski/Qwen2.5-3B-Instruct-GGUF

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESRmz6rO2AOgA72veax9pQ5p5TONlpzOeibyFGtkdPwmc1XdfXkHiaKBAQ/640?wx_fmt=jpeg)

总结：3B、8B都是手机上可以跑的模型，1.5B的专家模型的数学也可以很好，通用模型3B的能力也很强，8B通常需要12GB内存的旗舰手机。

常见问题：

1、哪里下载？

下载哪个就复制哪个文件的名字→谷歌搜索→找到针对手机处理器优化的Q4版本，不想折腾就直接下通用的，想折腾的就点图二的Soc功能，然后看看自己手机的处理器是否支持i8mm或者sve。（如果是去年刚买的旗舰手机，那就肯定支持。）

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESiczjMUEP2PwiaQJibklRXtNzKT5DGp9R9zfTyNYf1Tb1TYgauPHiaONtwQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESlNibF9zZGmsym0crjZian1Aadu75D4fpsnVkzCIZKTskYVoec9J1UNibA/640?wx_fmt=png)

bartowski/Qwen2.5-3B-Instruct-GGUF

bartowski/Llama-3.2-3B-Instruct-GGUF

bartowski/Ministral-8B-Instruct-2410-GGUF

2、如何使用ChatterUI？

很简单，下载模型到手机以后，先要选择好系统预设（Llama选Llama，没有列出来的推荐选ChatML）,然后才是导入模型，加载模型。完事！

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reESslubbEjP50fPdqwhVDomA1TC3WjybqThx7ITAXibbic4pswK1Z8bFYJQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/g8Uc5cB8VNwy3DChUvbibPA8SNMA9reES22KAE3IcLibPVOCaR1nDB4reBtw8ndgyloeJnoS84X4uziaIdBULxZxw/640?wx_fmt=jpeg)

\------------一些说明--------------------

1、TTS需要安装Google TTS（安装好就自带en-US离线数据），中文语音离线数据需要自己安装TTS以后，在手机设置搜索“TTS”，然后去下载，普通网络就可以直接下载的（数据不到10M）

2、ChatterUI在后台时间久了，很容易被系统自动释放内存，所以就需要重新Load模型

3、import导入模型A以后，模型A就有两份，为了节约手机空间，可以考虑把原来的模型文件备份到U盘以后，再删除。