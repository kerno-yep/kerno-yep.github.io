---
layout: post
title: 动态链接库劫持
date: 2022-9-22
categories: blog
tags: [笔记,拓展知识]
description: 动态链接库

---

### 动态链接库劫持

***

#### 程序链接方式

静态链接：程序运行前将各个目标模块以及库函数链接
装入链接：在程序编译装入内存时，一边编译，一边链接
运行链接：在程序执行时，需要时才进行链接

#### 动态链接库劫持

LD_PRELOAD环境变量和/etc/ld.so.preload配置文件中指定的动态链接库比LD_LIBRARY_PATH环境变量所定义的链接库查找路径的文件优先级要高，即使用户不使用，仍会加载。  

#### 利用方式  

1.`通过LD_PRELOAD环境变量加载`  
2.`通过/etc/ld.so.preload配置文件加载`  
3.`通过修改动态链接器实现恶意代码`

##### 通过LD_PRELOAD环境变量加载

```shell
LD_PRELOAD = /evil.so #设置恶意动态链接库
export  LD_PRELOAD  #导出生效
```

##### 通过/etc/ld.so.preload配置文件加载

```shell
echo "/evil.so" > /etc/ld.so.preload
```

##### 通过修改动态链接器实现

```shell
#1.修改动态链接器中配置文件路径/etc/ld.so.preload为自定义的路径
#2.在该路径中写入要预加载的恶意动态链接库的绝对路径
```

