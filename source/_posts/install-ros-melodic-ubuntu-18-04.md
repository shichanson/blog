---
title: "基于Ubuntu安装ROS 系统（melodic版本）"
author: "Mayer Shi"
tags: ["机器人"]
categories: ["ROS"]
date: 2020-06-04 10:25:23
draft: false
---
ROS (Robot Operating System, 机器人操作系统) 提供一系列程序库和工具以帮助软件开发者创建机器人应用软件。它提供了硬件抽象、设备驱动、函数库、可视化工具、消息传递和软件包管理等诸多功能。
<!--more-->


## 安装版本要求：
> * Ubuntu系统版本18.04 版本
> * ROS 版本为Melodic版本

## 安装步骤
1. 配置Ubuntu的repository源,编辑/etc/apt/sources.list文件。

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```
更新源以及更新系统

```bash
sudo apt-get update -y
sudo apt-get upgrade -y 
```

2. 配置ROS的Ubuntu源,
```bash
ROS 镜像使用帮助
新建 /etc/apt/sources.list.d/ros-latest.list，内容为：

你的Debian/Ubuntu版本: 
Ubuntu 18.04 LTS
deb https://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ bionic main
然后再输入如下命令，信任ROS的GPG Key，并更新索引：

sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt update
```

3. 开始安装ROS系统
```bash
sudo apt install ros-melodic-desktop-full -y
```

4. ROS环境变量设置
```bash
echo "source /opt/ros/melodic/setup.bash" >>  ~/.bashrc
source ~/.bashrc
```

5. 安装构建包依赖
```bash
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
```

6. 初始化rosdep工具，在使用许多ROS工具之前，您需要初始化rosdep。rosdep使您能够轻松地为要编译的源安装系统依赖性，并且是运行ROS中某些核心组件所必需的。
```bash
sudo apt install python-rosdep
sudo rosdep init
rosdep update
```

7. 验证当前安装环境是否正确。
```bash
# 开启一个终端执行如下命令
ros@ubuntu:~$ roscore
... logging to /home/mayershi/.ros/log/51594486-a3d7-11ea-a495-001c425709d4/roslaunch-ubuntu-14528.log
Checking log directory for disk usage. This may take a while.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://ubuntu:41953/
ros_comm version 1.14.5


SUMMARY
========

PARAMETERS
 * /rosdistro: melodic
 * /rosversion: 1.14.5

NODES

auto-starting new master
process[master]: started with pid [14541]
ROS_MASTER_URI=http://ubuntu:11311/

setting /run_id to 51594486-a3d7-11ea-a495-001c425709d4
process[rosout-1]: started with pid [14552]
started core service [/rosout]

# 新开一个终端执行命令
ros@ubuntu:~$ rosrun turtlesim turtlesim_node
# 会生产成一个小乌龟节点

# 新开另外一个终端执行命令
ros@ubuntu:~$ rosrun turtlesim turtle_teleop_key
# 在此终端上通过方向键来操控小乌龟移动。
```

效果如图：
![twT9mj.png](https://s1.ax1x.com/2020/06/04/twT9mj.png)
