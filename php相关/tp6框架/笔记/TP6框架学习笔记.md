## TP6框架学习笔记

### 绪论

> lesson 2 开发规范和目录结构

魔术方法：

无需调用 满足条件自启动的方法 例如构造方法，命名时前缀 __

app目录：

应用目录 开发在该目录下完成

当有多个app_name时 则为多应用模式

保证对外（网站页面）只有public文件夹可打开

> lesson 3  调试与配置

.env文件中的 APP_DEBUG 来控制调试模式的开启与否

部署后会被忽略 仅适用于本地测试开发

本地测试时 .env优先级高于config目录下的配置

> lesson 4 URL访问模式

https://ServerName like 127.0.0.1:8000 or tp6/public/index.php

例如 https://localhost:8000/index.php/test/hello/value/world

形如 服务器地址/index.php（可省略，需要设置URL重启）/控制器/操作/参数/值

兼容模式（如果不兼容为啥用tp6框架呢）

> lesson 5 控制器定义

controller 

系统默认控制器文件目录 config/route.php 路由设置

可以通过该文件避免文件冲突

渲染输出

① return  ② json()   ③ halt()中断函数

> lesson 6 多种控制器

一、基础控制器

继承表达 extends 关键字

二、空控制器

单应用模式 Error控制器类来提醒错误

三、多级控制器

在controller目录下新建其他目录 url访问时控制器输入格式为 目录.控制器类

### 数据库操作

> lesson 7 连接数据库

