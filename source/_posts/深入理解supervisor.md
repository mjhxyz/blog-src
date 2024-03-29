---
title: 深入理解supervisor
abbrlink: 48516
date: 2023-04-12 22:43:45
index_img: https://img.mjhxyz.top/20230415234947.png
tags:
- python
- supervisor
- 进程管理
- 进程守护
categories:
- python
---

# 深入理解supervisor

![Supervisor](https://img.mjhxyz.top/20230415234947.png)

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
sudo apt-get install supervisor
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

### 进程状态

- 运行中（RUNNING）：进程正在运行，并且没有出现错误。
- 停止（STOPPED）：进程已经停止运行，可能是由于程序异常退出、被手动停止或者由 Supervisor 自动停止。
- 启动中（STARTING）：进程正在启动，但还没有完全启动完成。
- 停止中（STOPPING）：进程正在停止，但还没有完全停止完成。
- 未知（UNKNOWN）：进程的状态无法确定。
- 启动失败（FATAL）：进程启动失败，超过了最大重试次数。

```bash
supervisor> status
timeit                           STOPPED   Apr 15 12:58 AM

supervisor> status
timeit                           RUNNING   pid 1473, uptime 0:00:16

supervisor> status
timeit                           FATAL     Exited too quickly (process log may have details
```

### 进程组

- 进程组是一组具有`相同配置`的进程
- 方便对于需要管理多个相似进程的场景
- 使用`[group:groupname]`指定进程组
- 只要保证所有配置文件都加载了

```conf

```conf
[group:mygroup]
programs=timeit,hello
```

```bash
supervisor> status
mygroup:hello                    RUNNING   pid 1689, uptime 0:00:53
mygroup:timeit                   RUNNING   pid 1688, uptime 0:00:53
```

接下来就能批量的操作进程了

```bash
supervisor> stop mygroup:*
mygroup:timeit: stopped
mygroup:hello: stopped
```

### 结束进程的方式

superivsor 有两种方式来结束进程, 使用 `SIGTERM` 和 `SIGKILL`, 默认使用 `SIGTERM`

当 supervisor 接收到停止进程的命令时，它首先会向进程发送一个 `SIGTERM` 信号, 这个信号告诉进程需要终止，并且在终止前可以做一些清理工作, 如果进程在收到 SIGTERM 信号后没有做出任何响应，即没有退出进程或者进行清理操作，那么 supervisor 会默认进程出现了异常情况, 此时会发送 `SIGKILL` 信号, 强制结束进程。具体的超时时间可以通过 `stopwaitsecs` 配置项来设置。

```python
import signal
import time

def handle_term(signum, frame):
    print('收到 SIGTERM')
    time.sleep(5)
    print('优雅退出...')

def handle_kill(signum, frame):
    print('收到 SIGKILL')
    time.sleep(5)
    print('强制退出...')

signal.signal(signal.SIGTERM, handle_term)
# signal.signal(signal.SIGKILL, handle_kill) # 无法被捕获到, 会报错的

print('进程已经启动...')
while True:
    time.sleep(1)
```

```bash
supervisor> status
mygroup:hello                    RUNNING   pid 1103, uptime 0:00:12
mygroup:timeit                   RUNNING   pid 1102, uptime 0:00:12
signal_test                      RUNNING   pid 1104, uptime 0:00:1

supervisor> stop signal_test 
signal_test: stopped
supervisor> tail signal_test 
进程已经启动...
收到 SIGTERM
优雅退出..
```

## 其他高级功能

### 事件监听器

supervisor 的事件处理器实现了监控`进程状态变化`和`日志输出`的功能, 使用 supervisor 管理进程时件管理器会监控该进程的状态，并将其状态变化和日志输出作为事件发送给事件监听器。监听器还能使用自己的脚本，来处理。

#### 常见的事件

- PROCESS_STATE: 从一个状态切换到另外的一个状态会触发
- PROCESS_STATE_STARTING: 从一个状态切换到 `STARTING` 状态会触发
- PROCESS_STATE_RUNNING: 从 `STARTING` 状态切换到 `RUNNING` 状态会触发
- PROCESS_STATE_BACKOFF: 从 `STARTING` 状态切换到 `BACKOFF` 状态会触发, 意味着没有成功的进入 `RUNNING` 状态, 会进行重试, 直到达到最大重试次数(`startretries`)
- PROCESS_STATE_STOPPING: 从 `RUNNING` 状态切换到 `STOPPING` 状态会触发
- PROCESS_STATE_EXITED: 从 `RUNNING` 状态切换到 `EXITED` 状态会触发, 意味着进程已经退出
    - 进程本身的运行逻辑已经退出并返回了一个退出码
- PROCESS_STATE_STOPPED: 从 `RUNNING` 状态切换到 `STOPPED` 状态会触发, 意味着进程已经停止
    - 表示已经停止运行，可能是 supervisor 停止了进程
- PROCESS_STATE_FATAL: 从 `BACKOFF` 状态切换到 `FATAL` 状态会触发, 并且超过了最大重试次数(`startretries`), 放弃了重启
- PROCESS_STATE_UNKNOWN: 切换到 `UNKNOWN` 状态会触发, 一般是 supervisor 本身有问题
- 还有很多关于日志输出的,就先不一一列举了，可以去文档上查看

#### 使用步骤

**需要使用到的文件**

1. 编写监听器配置文件
```conf
[eventlistener:test]
command=python test.py
events=PROCESS_STATE_FATAL
```
2. 编写监听器脚本
```python
import sys
def write_stdout(s):
    sys.stdout.write(s)
    sys.stdout.flush()

def write_stderr(s):
    sys.stderr.write(s)
    sys.stderr.flush()
def main():
    while True:
        write_stdout('READY\n')
        line = sys.stdin.readline()
        write_stderr(line)
        headers = dict([ x.split(':') for x in line.split() ])
        data = sys.stdin.read(int(headers['len']))
        write_stderr(data)
        write_stdout('RESULT 2\nOK')
if __name__ == '__main__':
    main()
```

**注意点**

- 一个监听器可以监听`多个事件`，多个事件之间用 `,` 分隔, 具体哪个事件触发了，可以通过读到的标准输出来查看
- 监听器脚本需要满足 `事件监听器协议`:
    1. supervisor 只会向处于 `READY` 状态的监听器发送`标准输入`, 因此监听器需要在启动时向标准输出写入 `READY\n` 来通知监听器已经进入 `READY` 状态
    2. 实现的监听器还需要从 `stdin` 读取事件携带的数据，读取的时候可能会阻塞
    3. 监听器处理完成以后，需要通知 supervisor 监听器已经处理完成
       1. 比如向标准输出写入 `RESULT 2\nOK` 或者 `RESULT 4\FAIL`
       2. 2 和 4 表示数据的长度，`OK` 和 `FAIL` 表示处理结果
       3. 如果返回的是 `FAIL`, 则表示处理器没有处理好事件，会在以后的某个时刻，再次发送
       4. 接着 supervisor 自动进入了 `ACKNOWLEDGED` 状态
    4. 进入 `ACKNOWLEDGED` 状态后，进程可以退出, 如何还需要继续处理，则需要再次向 `stdout` 写入 `READY\n`
- 更多参数可以去看 supervisor 详细文档


#### 使用场景


###### 发送警报

开发公司的一个项目的时候，需要监控进程的启动状态，如果进程启动失败，需要触发蜂鸣器报警。这个时候就可以使用事件监听器来实现。

1. 监听 `PROCESS_STATE_FATAL` 事件，当进程状态切换到 `FATAL` 状态时，说明重试次数已经用完，放弃了重启，此时触发蜂鸣器报警
2. 可以编写监听器配置文件
```conf
[eventlistener:easycube-listener]
command=easycube-listener
events=PROCESS_STATE_FATAL

autostart=true
autorestart=true
log_stdout=true
log_stderr=true
stdout_logfile=/var/log/easycube-listener-out.log
stdout_logfile_maxbytes=5MB
stdout_logfile_backups=3
buffer_size=10
stderr_logfile=/var/log/easycube-listener-err.log
stderr_logfile_maxbytes=5MB
stderr_logfile_backups=3

environment=PATH="/root/easycube-py/venv/bin:%(ENV_PATH)s"
```
4. 实现
```python
import sys
from easycube.libs import command

def write_stdout(s):
    sys.stdout.write(s)
    sys.stdout.flush()

def write_stderr(s):
    sys.stderr.write(s)
    sys.stderr.flush()

def main():
    while True:
        write_stdout('READY\n')
        line = sys.stdin.readline()
        write_stderr(line)
        headers = dict([x.split(':') for x in line.split()])
        data = sys.stdin.read(int(headers['len']))
        write_stderr(data)
        # beep 一下
        command.beep_long(show_cmd=False)
        write_stdout('RESULT 2\nOK')

```

##### 自动重启

可能会有疑问, supervisor 本身就可以自动重启进程, 为什么还要使用事件监听器呢? 我的理解可能有有以下方面的考虑

1. 有条件的重启: 有时候我们只需要在特定条件下才重启进程，例如当进程占用的内存超过一定阈值时才重启
2. 重启前清理: 在重启之前，可能需要做一些清理操作，比如删除某个临时文件等等

##### 日志记录

可以编写一个脚本来监控进程的日志输出，并将其写入文件或数据库中，以便稍后分析和查看。

supervisor 中，日志相关的事件有:

- `PROCESS_LOG_STDOUT `: 当进程向 `stdout` 写入时触发
- `PROCESS_LOG_STDERR `: 当进程向 `stderr` 写入时触发
- `PROCESS_LOG `: 上面两个的组合 

### 进程日志和日志管理

#### 如何查看进程日志

supervisor 中每个进程日志默认保存在 `/var/log/supervisor/` 目录下。可以使用 `tail` 命令来查看进程日志的实时输出。

```bash
tail -f /var/log/supervisor/supervisord.log
```

#### 如何配置日志文件位置

- 可以通过配置文件指定日志文件路径
    - `stdout_logfile`: 标准输出日志文件路径
    - `stderr_logfile`: 标准错误日志文件路径

```conf
stderr_logfile=/var/log/timeit.stderr.log
stdout_logfile=/var/log/timeit.stdout.log
```

#### 如何配置轮转和截断

- 以日志文件的大小进行轮转
    - `stdout_logfile_maxbytes`: 标准输出日志文件的最大字节数
    - `stderr_logfile_maxbytes`: 标准错误日志文件的最大字节数
    - `stdout_logfile_backups`: 标准输出日志文件的备份数量
    - `stderr_logfile_backups`: 标准错误日志文件的备份数量
- 以日志文件的时间进行轮转
    - `logrotate`: 是否开启日志轮转, 当 logrotate 选项启用时，supervisor 会在进程日志文件达到指定大小或超过指定时间间隔时，调用系统的 logrotate 工具来截断和备份日志文件
- 直接使用 logrotate,
    1. 添加配置文件 `/etc/logrotate.d/supervisor`
        ```conf
        /var/log/supervisor/*.log {
            daily
            rotate 7 
            compress
            delaycompress
            missingok
            notifempty
            copytruncate
        }
        ```
    2. 使用 `logrotate -d /etc/logrotate.d/supervisor` 来测试配置是否正确
    3. 配置自动化轮转, 使用 crontab, 每天凌晨 3 点运行一次 logrotate 命令
        ```bash
        0 3 * * * logrotate /etc/logrotate.d/myapp-logrotate.conf
        ```

### 远程管理

supervisor 提供了一个页面来远程管理进程, 需要先启用

先在配置文件中添加 `inet_http_server` 配置项

```conf
[inet_http_server]
port=0.0.0.0:9999
username=mao
password=123456
```

然后重启 supervisor 服务

```bash 
service supervisor restart
```

最后访问网站就能看到页面了

![supervisor web](https://img.mjhxyz.top/20230415232318.png)

## 总结

总的来说, supervisor 是一个非常好用的进程管理工具, 帮助我们轻松地管理和监控多个进程。除此之外，Supervisor 还有一些其他的好用功能:

1. 进程监控
2. 进程管理
3. 日志管理
4. 远程管理
5. 事件监听
6. 进程组管理
7. 进程启动优先级配置&进程启动顺序配置

如果需要管理多个进程，或者需要监控和控制进程的状态和行为等等，那么 `supervisor` 是一个值得尝试的工具。希望本文对您有所帮助，谢谢阅读！
