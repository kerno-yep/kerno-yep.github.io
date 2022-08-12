---
layout: page
title: "Awesome Cloud Security"
description: "云安全开坑"
header-img: "img/2.jpg"
---

# 进度记录

### **云服务基础概念** 

**`S3(Simple Storage Service)` 对象存储**，类似网盘，与Amazon的公开云存储服务对应的有S3协议，目前S3协议已经被视为公认的行业标准协议，目前国内的主流厂商基本上都支持S3协议。 

在Amazon S3标准中，对象存储可以看作是多个Bucket部署在服务器上，将要保存的Object存放在Bucket中，其中，Object分为三个部分：`key(Bucket中的唯一标识符)`, `data(存放的数据)`，`Metadata(数据的描述信息)`



**`EC2(Elastic Compute Cloud)`**  弹性计算机，类似云上虚拟机



**`RDS(Relational Database Service)`**   云上的一个数据库



**`IAM(Identity and Access Management)`**  身份与访问管理，云上的身份认证管理



### **漏洞整理**

#### **S3漏洞**

**Bucket爆破** 

以url：`https://bucket.s3.ap-northeast-2.amazonaws.com/aaa` 为例，其中 `bucket`是存储Bucket的名称，`aaa`是key

当不知道当前Bucket名称时，可以根据页面内容回显爆破

当 Bucket 不存在时有两种返回情况，分别是 `InvalidBucketName` 和 `NoSuchBucket` 当 Bucket 存在时也会有两种情况，分别是列出 `Object`和返回 `AccessDenied`

**Bucket接管**

当访问页面有显示`NoSuchBucket`,说明Bucket可以被接管，在控制台创建同名的Bucket即可接管目标Bucket，成功接管之后再次访问页面就会显示`AccessDenied`

在控制台将Bucket设置为公开，且尝试上传访问成功即完成接管



**S3任意文件上传**

当对象存储配置不当，可能会造成任意文件上传和文件覆盖

利用`put方法`对文件进行恶意写入，可能造成钓鱼，挂黑页，暗链等

**Bucket ACL可写**

查看目标Bucket ACL策略，根据预定义组可以分析出`URI=http://acs.amazonmaws.com/groups/global/AllUsers`=>当用户被授予`WRITE`,`WRITE_ACP`,`FULL_CONTROL`权限时，所有用户可以访问Bucket且写入ACL，当修改权限为FULL_CONTROL即可完全控制目标Bucket



**参考资料**

https://wiki.teamssix.com/CloudService/

