# ROS入门课程笔记
## ROS 是什么

ROS = 通信机制 + 开发工具 + 应用功能 + 生态系统

目标：提高机器人研发中的软件复用率

Rviz: 机器人可视化工具

MoveIt！: 运动规划功能包(ros平台是不支持的)

相关网站：Google, ROS wiki, ROS Answers, www.ros.org/news

*******************************
## ROS 的 核心概念

### 通信机制

**节点（Node）** 实现某个程序功能的进程，进程间可以使用不同的语言，但是明明必须唯一

**节点管理器（ROS Master）** 又叫控制中心，为节点提供命名和注册服务，帮助节点互相查找、建立连接，提供服务器参数（全局对象字典）：节点用此进行参数提取

#### 通信方式
**话题通信（发布/订阅模式）**

**话题 Topic** 
单向的 异步通信的 同一个话题的订阅者和发布者可以不唯一

**消息 Message 话题数据**
.msg文件定义话题数据接口

**服务通信 （C/S模式）**

同步通信机制 .srv文件定以请求和应答数据结构

**参数** 全局共享字典，适合存储静态、非二进制的配置参数，底层实现rpc，动态参数配置

### 文件系统

**功能包** ROS基本单元

**功能包清单** 记录功能包的基本信息

**元功能包** 组织为同一目的的多个功能包

***************************
## ROS命令行工具
roscore ：启动ros manager

rosrun语法： rosrun pacakageName nodeName

rqt_graph: 绘制当前ros模拟的工作流图

rosnode: 显示系统中的节点 list, info nodeName

rostopic: 显示系统中的话题列表 pub(发布指令) -r(发布频率)

rosmsg: 显示系统中的发布话题的数据结构 show

rosservice: 显示系统中的服务, call(调用指令)

rosbag: 话题记录，record(记录) play(复现)
**********************
## ROS工作空间

workspace 存放工程文件的文件夹,名字需要唯一，包含src(存放功能包), build(--), devel(编译后的文件与脚本), install(安装空间,开发结束后) 文件夹

### 执行语句
mkdir -> cd -> mkdir src -> cd -> catkin_init_workspace

回到工作空间根目录 catkin_make(编译) -> catkin_make install

src内创建功能包 catkin_create_pkg packageName NAME1 NAME2 ...

设置环境变量(让系统找到工作空间) source devel/setup.bash

src内两个文件 package.xml(个人修改)  CMakeLists.txt(编译规则文件)

***************************
## 发布者Publisher的编程实现

## launch启动文件的使用方法

**Launch文件** 通过XML格式来实现多节点的配置和启动

