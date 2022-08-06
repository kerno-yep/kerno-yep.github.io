---
layout: page
title: "Hole"
description: " 踩坑填坑记录 " 
header-img: "img/1.jpg"
---

## 2022/8/5 vs code 使用git经常断开连接

解决方案：使用git配置ssh公钥后，将git的http流量代理到本地代理服务器端口

```shell
git config --global http.proxy http://127.0.0.1:port
```



