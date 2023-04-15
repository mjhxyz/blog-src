---
title: 深入理解supervisor
abbrlink: 48516
date: 2023-04-12 22:43:45
tags:
- python
- supervisor
- 进程管理
- 进程守护
categories:
- python
---

# 深入理解supervisor

对于我来说，`supervisor` 已经是一个非常熟悉的工具了。在之前的公司，我用它来管理异步任务；现在，我则用它来管理数据采集进程。尽管我已经使用它很久，但我一直没有深入了解过这个工具。因此，我决定花一些时间来深入了解它的工作原理和功能。在本文中，我将分享我对 `supervisor` 的了解和体验，希望能对大家有所帮助。

## 简介

### 什么是 supervisor

`supervisor` 是一个很强大的进程管理工具，可以用于管理进程的启动、停止、重启和日志记录等。它的主要特点包括：

- 方便易用: 提供了简单的命令行工具方便用户进行进程的管理和监控
- 稳定可靠: 使用 python 编写，可靠性高
- 监控能力强: 监控进程的状态和运行日志, 出现问题的时候，会自动重启

## 安装&配置

### 安装方式

主要有两种安装方式：


- 使用 `pip` 安装
```bash
sudo pip install supervisor
```
- 使用 `apt` 安装
```bash
sudo apt-get install python-pip
```

### pip 安装和 apt 安装的区别

- 来源不同
    - 使用pip安装的软件包来自于PyPI
    - 使用apt安装的软件包来自于操作系统的软件包管理系统
- pip 安装的 supervisor 一般会比较新


所以使用 pip 安装 supervisor 可以在虚拟环境中管理 supervisor, 方便管理和升级, 但是由于 pip 安装的 supervisor 可能比较新，所以对于需要维护老版本的 supervisor 的童鞋, 可能会有不兼容的问题。

使用 apt 安装 supervisor 可以直接使用操作系统的软件包管理工具进行安装和管理, 但是安装的版本可能比较老，缺少一些新特性

因此, 需要根据具体的业务常见来选择如何安装 supervisor


### 配置

#### /etc/supervisor/supervisord.conf

`supervisor` 的主配置文件, 默认应该是下面的内容:

```conf
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf
```

里面包括的设置作用如下

- `[unix_http_server]`: 配置 supervisor 的  http server
    - `file`: supervisor 的 socket 文件路径
    - `chmod`: socket 文件的权限
- `[supervisord]`: supervisord 的详细配置
    - `logfile`: 日志文件路径
    - `pidfile`: supervisor 的 pid 文件路径
    - `childlogdir`: 子进程的日志文件路径
- `[supervisorctl]`: supervisorctl 命令的配置
    - `serverurl`: supervisorctl 的地址
- `[include]`: 包含其他配置文件
    - `files`: 配置文件的路径 /etc/supervisor/conf.d/*.conf, 我们只要在 conf.d/ 下面创建自己的配置文件即可

#### 项目配置

1. 准备好要管理的项目, 以及他的虚拟环境:

```python
import time
while True:
    print(int(time.time()), flush=True)
    time.sleep(2)
```
```bash
virtualenv virtualenv --python=python3.9
```

2. 在 `/etc/supervisor/conf.d/` 目录下可以创建一个给自己程序配置的文件, `timeit.conf`, 内容如下:

```conf
[program:timeit]
command=python /home/mao/git/timeit/main.py ; 被管理命令
directory=/home/mao/run ; 工作目录
user=mao ; 运行用户
autostart=true ; 是否在 supervisor 启动时自动启动
autorestart=true ; 是否程序异常退出时自动重启
startretries=3 ; 启动失败后, 重试的次数
startsecs=10 ; 程序在启动后, 保持多少秒,则认为启动成功
stdout_logfile=/var/log/timeit.out.log  ; 标准输出日志文件路径
stderr_logfile=/var/log/timeit.err.log  ; 标准错误日志文件路径

environment=PATH="/home/mao/git/timeit/virtualenv/bin:%(ENV_PATH)s" ; 将虚拟环境的路径添加到 PATH 环境变量中, 挺好用的
```

3. 重新载入配置文件

```bash
supervisorctl reload
```

4. 查看进程状态

```bash
root@work-os:~# supervisorctl 
timeit                           RUNNING   pid 939, uptime 0:00:52
supervisor> status
timeit                           RUNNING   pid 939, uptime 0:00:53
supervisor> tail -f timeit 
==> Press Ctrl-C to exit <==
1681478309
1681478311
1681478313
1681478315
1681478317
```

### 常用命令

- supervisord: 启动 supervisor 服务
- supervisorctl: 命令行工具, 可以用来管理进程
- supervisorctl status: 查看进程状态
- supervisorctl start/stop/restart: 启动/停止/重启进程
- supervisorctl tail -f timeit: 查看进程的日志
- supervisorctl reread: 重新读取配置文件
- supervisorctl update: 更新配置文件
- supervisorctl reload/update: 重新加载配置文件, 不知道什么区别，文档的描述反正是一摸一样的(Reload config and add/remove as necessary, and will restart affected programs)



## 核心概念

## 常用命令

## 其他高级功能

## 总结