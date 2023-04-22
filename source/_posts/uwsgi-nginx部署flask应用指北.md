---
title: uwsgi+nginx部署flask应用指北
tags:
  - python
  - flask
  - uwsgi
  - nginx
  - 部署
categories:
  - python
abbrlink: 7455
date: 2023-04-22 19:56:01
---
<!-- 
- 为什么要使用 nginx
- 性能测试和优化建议 
- uwsgi 配置 以适应不同的应用场景
- 一些常见问题和解决方案
- -->

# 使用uwsgi+nginx部署flask应用指北

## 前言

在我之前的工作中，我经常使用uwsgi+nginx来部署Flask应用程序。虽然我知道大概的部署方式，但只是知其然而不知其所以然。我知道需要使用这样的架构来部署，但不太清楚为什么需要使用uwsgi+nginx，以及这种做法的好处是什么。在深入研究和实践后，我逐渐理解了uwsgi和nginx各自的作用和优点，以及它们如何协同工作来提高Flask应用程序的性能和稳定性。

在撰写本文之前，我们假设大家已经具有一定的 Flask 和 uWSGI 的使用经验，并且对 Nginx 有一定的了解。

接下来我将按照以下顺序来解释:

- 如何使用 uWSGI + Nginx 部署 Flask 应用程序。
- 为什么使用 uWSGI + Nginx 进行部署。
- 性能测试
- 优化建议


## 基础回顾

- 对于 Flask 应用程序来说, 已经实现了 `WSGI(Python Web Server Gateway Interface)` 协议, 因此可以直接使用任意 WSGI 服务器来部署, 例如 uwsgi, gunicorn, tornado 等
- uwsgi 是一个 WSGI 服务器, 可以作为 Web 服务器, 接收来自客户端的请求并将其转发给 WSGI 应用程序进行处理

## 如何使用 uwsgi+nginx 部署 flask 应用程序

### 预备说明

- 这个部分将使用 flask 写一个简单的例子，然后设置 uWsgi 服务器，并使用 Nginx 作为反向代理服务器。
- 默认 python3 环境已经存在了, 并且使用的是 `python-venv` 进行虚拟环境的管理, 因为 `python-venv` 是 python3 自带的, 不需要额外安装, 且使用起来也比较方便, 不过如果你使用的是 python2, 那么你需要安装 `virtualenv` 这个包, 用来创建虚拟环境

### 创建 Flask 应用程序

1. 首先创建一个目录, 用来存放我们的应用程序, 例如 `greeting`

```bash
cd /home/mao/project/
mkdir greeting
cd greeting
```

2. 编写 `greeting.py` 文件:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello mao'

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

### 创建项目虚拟环境

1. **创建虚拟环境**
```bash
python3 -m venv venv
```
2. **激活虚拟环境**
```bash
source venv/bin/activate
```
此时终端前面会出现 `(venv)` 字样, 表示虚拟环境已经激活 `(venv) mao@work-os:~/project/greeting$ `

3. **安装依赖**
```bash
pip install flask uwsgi
```

### 使用 flask 内置的开发服务器运行应用程序

在激活虚拟环境后, 直接运行 `greeting.py` 文件即可

```bash
(venv) mao@work-os:~/project/greeting$ python greeting.py
 * Serving Flask app 'greeting'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://10.0.2.15:5000
Press CTRL+C to quit
```

成功运行后, 浏览器访问能看到效果，说明 Flask 应用程序没问题，虚拟环境也没问题

### 配置 uWSGI

1. 添加一个作为我们程序的入口，这个文件名可以随意取，但一般都是 `wsgi.py`，这个文件的作用是让 uWSGI 知道如何加载我们的 Flask 应用程序

```python
from greeting import app

if __name__ == "__main__":
    app.run()
```

2. 直接终端使用 `uwsgi` 命令来启动我们的 wsgi 应用程序

```bash
uwsgi --http :5000 -w wsgi:app
```
此时说明我们虚拟环境中的 uwsgi 已经安装成功了, 并且可以正常运行。

3. 创建 `uwsgi.ini` 文件, 用来配置 uwsgi

```ini
[uwsgi]
module = wsgi:app
master = true
processes = 4
socket = greeting.sock
chmod-socket = 660
touch-reload = /home/mao/project/greeting/greeting-reload
vacuum = true
die-on-term = true
```
其中的各个配置项的含义如下:

- `module` : 指定 wsgi 应用程序的入口, 这里指的是 `wsgi.py` 文件中的 `app` 对象
- `master` : 以 master 模式运行, 会启动一个主进程来管理子进程, 并在需要时重启子进程
- `processes` : 指定 uWSGI 子进程启动的进程数
- `socket` : 指定 uWSGI 服务器监听的 socket 文件
  - 测试的时候，使用的是 http 协议, 但最后是使用 Nginx 来处理客户端的请求, 然后将请求通过 `socket` 文件转发给 uWSGI 服务器
  - 使用 Unix 套接字文件而不是 TCP 套接字文件的原因是
    1. Unix 套接字是本地 IPC 的一种方式, 无需经过网络协议栈的处理, 速度比 TCP 套接字快, `性能更好`
    2. Unix 套接字文件只能被本地用户或具有适当权限的用户访问，因此比 TCP 套接字`更加安全`。
- `chmod-socket` : 指定 socket 文件的权限, 这里是 660, 并且这里要保证 **Nginx 用户有读写权限**
- `touch-reload` : 指定一个文件, 当这个文件被 `touch` 的时候, uWSGI 会自动重启
- `vacuum` : 启用自动清理, 关闭进程时自动清理 Unix 套接字文件
- `die-on-term` : 保证 uWSGI 在收到 TERM 信号时优雅地退出进程, 以避免意外断开客户端连接

到这为止，咱们的 uWSGI 配置就完成了, 现在可以考虑将 uWSGI 作为服务来运行

### 将 uWSGI 作为服务运行

之前公司是使用 `SysVinit` 来管理服务的, 将服务配置文件放在 `/etc/init.d/` 目录下, 然后使用 `service` 命令来管理服务

但是现在 SysVinit 已经被 `Systemd` 替代了, 所以这里就不再介绍 SysVinit 的方式了, 直接介绍 systemd 的方式

1. 在 `/etc/systemd/system/` 目录下创建一个 `greeting.service` 文件, 用来配置 uWSGI 服务

```ini
[Unit]
Description=uWSGI serve greeting
After=network.target

[Service]
User=mao
Group=mao
PIDFile=/home/mao/project/greeting/greeting.pid
WorkingDirectory=/home/mao/project/greeting
Environment="PATH=/home/mao/project/greeting/venv/bin"
ExecStart=/home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini

[Install]
WantedBy=multi-user.target
```

服务配置文件的各个配置项的含义如下:

- `Unit` : 服务的描述信息
  - `Description` : 服务的描述信息
  - `After` : 这里指定该服务必须在网络服务加载之后启动
- `Service` : 服务的配置信息
  - `User` : 服务运行的用户
  - `Group` : 服务运行的用户组
  - `PIDFile` : 指定服务的 PID 文件
  - `WorkingDirectory` : 指定服务运行的工作目录
  - `Environment` : 指定服务运行的环境变量, 这里将虚拟环境的 bin 目录添加到环境变量中, `uwsgi` 命令才能被找到
  - `ExecStart` : 启动 uWSGI 服务的命令
- `Install` : 指定该服务在哪些运行级别下启动
  - `WantedBy` : 表示服务需要`被 multi-user.target 所依赖`, 也就是说, 说该服务应该在系统启动时自动启动


现在我们的 uWSGI 服务已经配置好了, 可以使用 `systemctl` 命令来管理服务了

```bash
# 启动服务
systemctl start greeting
# 查看服务状态
systemctl status greeting
# 开机自启动
systemctl enable greeting
```

成功运行了会看到如下输出:

```bash
● greeting.service - uWSGI serve greeting
     Loaded: loaded (/etc/systemd/system/greeting.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-04-22 22:09:18 CST; 7s ago
   Main PID: 3220 (uwsgi)
      Tasks: 5 (limit: 4675)
     Memory: 20.0M
        CPU: 295ms
     CGroup: /system.slice/greeting.service
             ├─3220 /home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini
             ├─3221 /home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini
             ├─3222 /home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini
             ├─3223 /home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini
             └─3224 /home/mao/project/greeting/venv/bin/uwsgi --ini uwsgi.ini
```

### 配置 Nginx 作为反向代理

目前我们的 uWSGI 服务已经可以正常运行了, 但是目前的 uWSGI 服务只能通过 `socket` 文件来访问, 无法通过 `http` 协议来访问, 所以这里需要配置 Nginx 来作为反向代理，使用 uWSGI 协议将请求发送到 socket 文件中

对于使用 apt 安装的 Nginx, 配置文件在 `/etc/nginx/sites-available/` 目录下, 在这里创建一个配置文件 `greeting.conf`, 然后在 `/etc/nginx/sites-enabled/` 目录下创建一个软连接 `ln -s /etc/nginx/sites-available/greeting.conf /etc/nginx/sites-enabled/greeting.conf`

不过我目前使用的 nginx 是使用源码编译安装的，配置文件在 `/home/mao/nginx/conf/` 目录下, 那么我就在这里创建一个配置文件 `greeting.conf`, 然后在 `/home/mao/nginx/conf/nginx.conf` 文件中的 `http` 配置块中添加 `include /home/mao/nginx/conf/greeting.conf;`

```ini
server {
    listen 80;
    server_name 192.168.60.100;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/mao/project/greeting/greeting.sock;
    }
}
```

Nginx 的基本配置就不多说了，主要有以下重要的配置项:

- include uwsgi_params : 包含 uwsgi_params 文件，以确保请求中包含了正确的参数和格式
- uwsgi_pass unix:/home/mao/project/greeting/greeting.sock : 将请求转发到 uWSGI 服务指定的 socket 文件中

配置完成以后检查配置项是否有错误

```bash
/home/mao/nginx/sbin/nginx -t
```

然后重启 Nginx 服务

```bash
/home/mao/nginx/sbin/nginx -s reload
```

此时就完成了 Nginx 作为反向代理的配置

## 为什么是 uwsgi+nginx

TODO


## 更多 uwsgi 配置

TODO


## 一些常见问题和解决方案

TODO

## 总结

TODO