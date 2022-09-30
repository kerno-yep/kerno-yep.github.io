---
layout: page
title: "Tips"
description: " 小Tricks记录 " 
header-img: "img/1.jpg"
---

## 2022/8/5 VS Code 使用git经常断开连接


使用git配置ssh公钥后，将git的http流量代理到本地代理服务器端口

```shell
git config --global http.proxy http://127.0.0.1:port
```

****

## 2022/8/6 Typora 永久性插入图片

将图片base64编码，利用`<img src='base64编码内容'>`插入  
将图片base64编码，利用`![][1]` `[]:base64编码内容` 插入

****

## 2022/8/29 fatal: No url found for submodule path 'xxxx' in .gitmodules

git rm --cached 'xxxx'

****

## 2022/8/29 pip换源

#### 国内常用镜像源汇总

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

****

## 2022/9/19 文件位置

hosts文件位置：`win10`C:\Windows\System32\drivers\etc  
WMPFRuntime文件位置：  C:\Users\user\AppData\Roaming\Tencent\WeChat\XPlugin\Plugins\WMPFRuntime

****

## 2022/9/19 nslookup记录

A记录：Address，记录主机的IP地址  
MX记录：Mail Exchanger，邮件交换记录  
CNAME记录：Canonical Name，将多个域名映射到同一个ip的设置记录  
NS记录：Name Server, 记录域名由哪一个dns服务器进行解析  
PTR记录：Pointer Record，进行反向地址解析调用的记录  
AAAA记录：将域名解析到IPv6地址的DNS记录  
TXT记录：TXT记录一般指为某个主机名或域名设置的说明

****

## 2022/9/20 npm忽略冲突

```vue
npm install --legacy-peer-deps
```

****

## 2022/9/26 tmux 命令备忘

```shell
ctrl+b or ctrl+a  #前缀键
tmux new -s session-name #创建会话
ctrl+b d #退出当前会话窗口
tmux ls  #查看所有会话
tmux a -t session-name #进入会话
tmux switch -t session-name #切换会话
tmux send -t session-name "order" ENTER #发送执行的命令
tmux  capture-pane -p #打印会话的显示
```

***

## 2022/9/28 php语言特性

php项目中，能传filename的地方，基本上都能传协议流