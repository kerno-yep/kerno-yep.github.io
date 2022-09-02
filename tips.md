---
layout: page
title: "Tips"
description: " 小Tricks记录 " 
header-img: "img/1.jpg"
---

## 2022/8/5 VS Code 使用git经常断开连接

解决：  
使用git配置ssh公钥后，将git的http流量代理到本地代理服务器端口

```shell
git config --global http.proxy http://127.0.0.1:port
```

## 2022/8/6 Typora 永久性插入图片

解决：  
将图片base64编码，利用`<img src='base64编码内容'>`插入  
将图片base64编码，利用`![][1]` `[]:base64编码内容` 插入

## 2022/8/29 fatal: No url found for submodule path 'xxxx' in .gitmodules

解决：  
git rm --cached 'xxxx'

## 2022/8/29 pip换源

解决：
国内常用镜像源汇总

清华镜像  
https://pypi.tuna.tsinghua.edu.cn/simple  
中科大镜像  
https://pypi.mirrors.ustc.edu.cn/simple  
豆瓣镜像  
http://pypi.douban.com/simple/  
阿里镜像  
https://mirrors.aliyun.com/pypi/simple/  
华中科大镜像  
http://pypi.hustunique.com/  
山东理工大学镜像  
http://pypi.hustunique.com/  
搜狐镜像  
http://mirrors.sohu.com/Python/  
百度镜像  
https://mirror.baidu.com/pypi/simple