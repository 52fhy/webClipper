# 【对象存储】一文搞懂MinIO部署与使用
**一、MinIO简介**

MinIO是一款开源的对象存储系统（Object Storage），号称世界上最快的对象存储，在特定环境下GET/PUT的结果可达325GiB/S和165GiB/S。对象存储的出现解决了传统存储如NAS、SAN的容量和性能不足的问题，与传统文件存储不同的地方在于它是以对象为单位来存储和检索数据，每个对象都包含了数据本身和元数据，每个对象都包含了数据本身和元数据。元数据主要包括对象的描述信息，如用户（accout）、存储桶（bucket）以及索引（index）等，而数据则包含了各种格式，比如文本、语音、图像、视频和其他非结构化数据。

国内外比较知名的对象存储有阿里云OSS，AWS S3（2006年随着S3的出现开始定义了对象存储这个概念，而MinIO则可以完全兼容S3接口）。MinIO支持的对象文件大小可以从KB级到TB级，客户端则是通过唯一标识来识别每个对象，被上传的对象会被分片存放在每个节点、每个磁盘下以bucket命名的目录中，并且会以每个文件的名字再生成一个子目录，目录下就是被拆分后的数据（xl.meta），该文件无法直接阅读。比如上传一个20M大小名为test.txt的文件到test这个bucket中，那么MinIO各节点的目录树结构为/data/test/test.txt/xl.meta，总占用大小大概在40M左右。

**二、MinIO纠删码**

MinIO有一个最重要的特性就是通过纠删码（Erasure Code，简称EC）来最大化确保数据的安全（至少需要4块磁盘才能使用纠删码特性，不管是单机多磁盘还是多机多磁盘都一样）。该模式会将对象拆分成N/2份数据和N/2分奇偶校验块，然后分散存储在不同的磁盘上，这样部分磁盘损坏也可以通过剩余的数据块和校验块恢复数据。举例来说，假设有12块磁盘组成了MinIO集群，存储的对象就会被分成6个数据块和6个校验块，然后任意损坏6块磁盘（不管是数据盘还是校验盘）都可以保证数据不丢失。

**三、MinIO的安装**

1、通过MinIO官网（https://min.io/）找到客户端和服务端的下载页面

![](https://mmbiz.qpic.cn/mmbiz_png/zkzLCqIqomYzzP2sVAcmrpTvMDT5Kcqb2kQgh24BlPh7HtCcOecljibDsKicJHaKWCarzFpek8aE0WLTQRFibzyUw/640?wx_fmt=png&from=appmsg)

2、下载MinIO服务端和客户端的二进制文件

```ruby

wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio


wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
```

**四、运行MinIO**

**1、单机模式（仅供测试）**

服务启动后可以看到API地址、控制台地址等信息

```ini
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password ./minio server /mnt/data --console-address ":9001"




```

**2、单机多磁盘模式（存在单节点故障）**

需要提前做好磁盘分区和格式再进行启动

```javascript
nohup ./minio server --console-address ":9091" /data/{1..12} > /tmp/min.lo 2>&1 &
```

**3、多节点多磁盘的分布式集群模式（生产推荐）**

分布式Minio官方建议生产环境最少4个节点，因为有N个节点，得至少保证有N/2的节点才能可读，保证至少N/2+1的节点才能可写。

1、配置启动脚本

```makefile
cat /usr/lib/systemd/system/minio.service 
[Unit]
Description=MinIO
Documentation=https://minio.org.cn/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local
User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

Restart=always
LimitNOFILE=65536
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target


```

2、设置MinIO用户和组，启动脚本依赖该账号

```powershell
groupadd -r minio-user
useradd -M -r -g minio-user minio-user
chown minio-user:minio-user /data2 /data3 /data4 /data5
```

3、创建变量配置文件，MinIO启动的时候会依赖

```makefile
cat /etc/default/minio 
MINIO_VOLUMES="http://192.168.254.20{1...3}:9000/data{1...4}"
MINIO_OPTS="--console-address :9001"
MINIO_ROOT_USER=tanglu
MINIO_ROOT_PASSWORD=123456789

```

4、每个节点启动服务，运行后初始的报错可以忽略，因为还没有建立好连接

```sql
systemctl start minio
tailf /var/log/messages
```

5、非脚本启动命令

```ruby
MINIO_ROOT_USER="admin" MINIO_ROOT_PASSWORD="123456789" minio server --console-address ":9001" \
http://192.168.254.101:9000/data2 http://192.168.254.101:9000/data3  http://192.168.254.101:9000/data4 http://192.168.254.101:9000/data5  \
http://192.168.254.102:9000/data2 http://192.168.254.102:9000/data3  http://192.168.254.102:9000/data4 http://192.168.254.102:9000/data5  \
http://192.168.254.103:9000/data2 http://192.168.254.103:9000/data3  http://192.168.254.103:9000/data4 http://192.168.254.103:9000/data5
```

**五、MinIO的客户端管理**

1、集群别名管理

为集群设置别名，只有设置了别名后才能纳入集群管理。这里创建的集群名为myminio，所以后续都用这个集群名进行演示

```cs
# 为集群创建别名
mc alias set myminio http:

#查看纳入管理的集群
mc alias list

#删除别名
mc alias remove myminio
```

2、查看指定集群下的bucket列表

```nginx

mc ls myminio
```

3、查看指定集群下指定bucket的大小

```nginx
mc du myminio/test1
```

4、mc admin命令

用于对集群进行查看和管理，比如节点信息、磁盘信息、集群用量信息等

```nginx

mc admin info myminio
```

![](https://mmbiz.qpic.cn/mmbiz_png/zkzLCqIqomYzzP2sVAcmrpTvMDT5KcqbcDwGCIcspfFXGvCVUt8sliaVXNuDrsH9YEtxYicKWO3VckzHcWGYYy6A/640?wx_fmt=png&from=appmsg)

5、查看MinIO日志

```nginx
mc admin logs myminio
```

6、恢复集群数据

```apache
mc admin heal -r [alias_name]/[bucket_name]
```

7、用户管理

```shell
mc admin user COMMAND

# COMMANDS:                                                                                                       
# add         add a new user                                                                                    
# disable     disable user                                                                                      
# enable      enable user                                                                                       
# remove, rm  remove user                                                                                       
# list, ls    list all users                                                                                    
# info        display info of a user                                                                            
# policy      export user policies in JSON format                                                               
# svcacct     manage service accounts                                                                           
# sts         manage STS accounts
```

8、权限管理

```sql
USAGE:                                                                                                          
  mc admin policy COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]                                                   
                                                                                                                
COMMANDS:                                                                                                       
  create      create a new IAM policy                                                                           
  remove, rm  remove an IAM policy                                                                              
  list, ls    list all IAM policies                                                                             
  info        show info on an IAM policy                                                                        
  attach      attach an IAM policy to a user or group                                                           
  detach      detach an IAM policy from a user or group                                                         
  entities    list policy association entities
```

**六、MinIO常见问题**

1、如果服务启动时出现Authentication failed问题，可以优先检查节点间时间是否同步，一定配置好NTP服务

2、服务启动时告警Detected Linux kernel version older than 4.0.0 release, there are some known potential performance problems with this kernel version. MinIO recommends a minimum of 4.x.x linux kernel version for best performance。如果是CentOS7系统，请将内核版本升级

3、MinIO不会自动重平衡，即不会将对象从旧的节点中迁移到新的节点。但是，MinIO会根据存储空闲大小进行加权选择，加权值是节点的空闲空间量除以所有可用池上的空闲空间。除了可用空间加权，如果继续写入文件会使得磁盘使用率超过99%或者空闲的inode计数低于1000，同样不会再往该节点写入数据

```properties
例如目前集群有3个Server Pool：
Pool A has 3 TiB of free space
Pool B has 2 TiB of free space
Pool C has 5 TiB of free space

Minio分别向各个Pool中写入的概率为：
Pool A：30% = 3 / (3 + 2 + 5)
Pool B：20% = 2 / (3 + 2 + 5)
Pool C：50% = 5 / (3 + 2 + 5)
```