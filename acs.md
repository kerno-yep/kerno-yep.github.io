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





**参考资料**

https://wiki.teamssix.com/CloudService/

