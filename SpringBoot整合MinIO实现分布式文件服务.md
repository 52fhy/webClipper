# SpringBoot整合MinIO实现分布式文件服务
今天分享一个非常不错且开源的分布式存储组件MinIO，有很多朋友在用。

什么是MinIO？
---------

Minio 是个基于 Golang 编写的开源对象存储套件，基于Apache License v2.0开源协议，虽然轻量，却拥有着不错的性能。它兼容亚马逊S3云存储服务接口。可以很简单的和其他应用结合使用，例如 NodeJS、Redis、MySQL等。

### 1\. 应用场景

MinIO 的应用场景除了可以作为私有云的对象存储服务来使用，也可以作为云对象存储的网关层，无缝对接 `Amazon S3` 或者 `MicroSoft Azure` 。

![](https://mmbiz.qpic.cn/mmbiz/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPruoPXYHSzZcotyKqfuCbstAludarRAQcibTH19gVZsRbEdod7cGbiaIApQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

### 2\. 特点

1.  **高性能**：作为一款高性能存储，在标准硬件条件下，其读写速率分别可以达到 `55Gb/s` 和 `35Gb/s`。并且MinIO 支持一个对象文件可以是任意大小，从几kb到最大5T不等。
    
2.  **可扩展**：不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并且支持跨越多个数据中心。
    
3.  **云原生**：容器化、基于K8S的编排、多租户支持。
    
4.  **Amazon S3兼容**：使用 Amazon S3 v2 / v4 API。可以使用Minio SDK，Minio Client，AWS SDK 和 AWS CLI 访问Minio服务器。
    
5.  **SDK支持**：
    

*   GO SDK：https://github.com/minio/minio-go
    
*   JavaSDK：https://github.com/minio/minio-java
    
*   PythonSDK：https://github.com/minio/minio-py
    

7.  **图形化界面**：有操作页面
    
8.  **支持纠删码**：MinIO使用纠删码、Checksum来防止硬件错误和静默数据污染。在最高冗余度配置下，即使丢失1/2的磁盘也能恢复数据。
    

> 功能很强大，本文只是抛砖引玉，有兴趣的朋友自己去探索吧~

安装MinIO
-------

安装非常简单，笔者这里使用docker安装，步骤如下：

### 1\. 获取镜像

执行命令如下：

```
docker pull minio/minio
```

### 2\. 启动镜像

执行命令如下：

```
 docker run -p 9000:9000 -p 9001:9001 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=admin" -e "MINIO_SECRET_KEY=admin" -v /home/data:/data -v /home/config:/root/.minio minio/minio server --console-address ":9000" --address ":9001" /data
```

命令解释如下：

*   **\-p**：**9000**是图形界面的端口，**9001**是API的端口，在使用SDK连接需要用到
    
*   **MINIO\_ACCESS\_KEY**：指定图形界面的用户名
    
*   **MINIO\_SECRET\_KEY**：指定图形界面的密码
    

按照上述两个步骤启动成功即可。

### 3\. 图形界面操作

安装成功后直接访问地址：**http:/ip:9000/login**，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPruibzbjwiagcWeXrCvKhb8qMo6qkF9UTjxDIwnanialFcQPrj5ECvPSNfOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

输入用户名和密码登录成功后，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPru5JJibz7ISmTWibVMCjbs3lfkWgvqWtEz9QCYicqsvVbQAIT0e9MRtnMtw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

菜单很多，这里就不再详细介绍了，笔者这里直接在**Buckets**菜单中创建一个桶为**test**，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPru3ibbIge0ukQZBCjwWzVwNmN6S7ejCjUw5Hrqu3owibrsagDlic3m4yAPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

并且设置这个桶的隐私规则为**public**，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPrubf9voRNfIrDhDQ08OYWAUXeGdZxLMaicUsohBfBU0TMUYYlEIdbvaRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> MinIO到此已经安装设置成功了

SpringBoot整合MinIO上传文件
---------------------

虽然MinIO在图形界面提供了手动上传的操作，但是也可以通过SDK的方式去上传，下面介绍一下Spring Boot 整合MinIO上传文件。

### 1\. 获取accessKey和secretKey

这里的**accessKey**和**secretKey**并不是图形界面登录名和密码，获取很简单，直接在图形界面中操作，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPruAuSGvTyWBexZWTudFeTDHAMeUFEAgX8tXoicFx7M7jC2Z9uSh3txwzw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPruHCbWLeeXR7buXxRSXzMMxDVKXlv3lHKYWPvHzCnKwAjymQsaEd1hYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 2\. 添加依赖

添加MinIO的依赖，如下：

```
<dependency>    <groupId>io.minio</groupId>    <artifactId>minio</artifactId>    <version>8.2.1</version></dependency>
```

### 3\. 添加配置

这里笔者对SDK做了简单的封装，案例源码都会提供，下面只列出部分代码。

在**aplication.yml**配置中添加MInIO相关的配置，如下：

```
minio:  # 访问的url  endpoint: http://192.168.47.148  # API的端口  port: 9001  # 秘钥  accessKey: HQGWFYLWGC6FVJ0CQFOG  secretKey: pUGhAgQhZDxJaLmN3uz65YX7Bb3FyLdLglBvcCr1  secure: false  bucket-name: test # 桶名 我这是给出了一个默认桶名  image-size: 10485760 # 我在这里设定了 图片文件的最大大小  file-size: 1073741824 # 此处是设定了文件的最大大小
```

### 4\. 新建上传文件接口

笔者这里定义了一个上传文件接口，如下：

```
/** * @author 码猿技术专栏 */@RequestMapping("/minio")@RestControllerpublic class MinioController {    @Autowired    private  MinioService minioService;    @PostMapping("/upload")    public String uploadFile(MultipartFile file, String bucketName) {        String fileType = FileTypeUtils.getFileType(file);        if (fileType != null) {            return minioService.putObject(file, bucketName, fileType);        }        return "不支持的文件格式。请确认格式,重新上传！！！";    }}
```

### 5\. 测试

上述4个步骤已经整合完成了，下面直接调用接口上传一张图片试一下，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPru30H3RfyFND6zdcTaVHp2Ubk0BhoeSTjtqUvQXBgYGOkqClJXLjD3HQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接口返回的**URL**就是文件的访问地址，直接输入浏览器访问即可。

在MInIO中也可以看到存储的文件，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPruhic0qHC09vPeDJeZYSa8jplSeibA8HAOjGFNUD2vvAGecRaQB9sSPwDg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果你需要分享给别人，也可以手动分享，有效期是7天，一旦过了这个有效期将会失效，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rDibbGkYt5AQq0q6rpU6JPrun7s0cEgjaNELzq92tfQy2ue2vbGPp2cD7ia3HHv7VjIM06JibpqpB5ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

总结
--

MInIO虽然是个开源项目，但是功能非常强大，小型项目中完全可以用它实现对象存储，也可以使用MinIO搭建一个免费的图床。

项目源码地址
------

https://github.com/chenjiabing666/JavaFamily/tree/master/springboot-minio

