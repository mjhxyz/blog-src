---
title: 虚拟机安装debian
abbrlink: 28096
date: 2023-04-05 09:17:10
tags:
- 虚拟机
- 网络
- debian
categories:
- debian
---

# 虚拟机安装 debian

由于许多服务需要在Linux服务器上部署，因此对我来说，如果不是直接在Linux上进行开发，我仍然需要一个虚拟机来运行Linux操作系统。无论是测试部署还是进行Linux开发，都会更加便捷。

## 前置准备下载

### 下载 virtualbox

- 去官网下载页面: https://www.virtualbox.org/wiki/Downloads
- 选择对应的版本下载: 我使用的是 windows 宿主机，因此选择了 `Windows hosts` 的版本
- 下载完成后，可能需要 Microsoft Visual C++ 运行库，如果没有，则自行安装 [下载地址](https://support.microsoft.com/zh-cn/help/2977003/the-latest-supported-visual-c-downloads)

### 下载 debian 镜像

- 去官网下载页面: https://www.debian.org/download
- 下载 iso 文件即可

## 安装镜像

1. 打开 virtualbox，点击 `新建` 按钮, 并选择好存放的位置，和要安装的系统镜像文件, 再点击下一步
![新建虚拟机](https://img.mjhxyz.top/20230405102525.png)

2. 设置好账号和密码，然后点击下一步
![设置账号密码](https://img.mjhxyz.top/20230405102852.png)

3. 根据自己宿主机的配置，设置好内存和 CPU, 硬盘资源，然后点击下一步
![内存和CPU](https://img.mjhxyz.top/20230405103008.png)
![硬盘](https://img.mjhxyz.top/20230405103104.png)

4. 最后显示摘要信息，点击 Finish 即可
5. 接着虚拟机会自动运行刚刚新建的虚拟机，然后会进入安装界面，按照步骤安装即可

## 网络配置

安装完成以后，会有一个默认的网卡 `enp0s3`, 使用 `NAT` 将虚拟机内部的 IP 地址映射到宿主机系统的 IP 地址上, 从而实现虚拟机和外部网络之间的通信

但问题是 `NAT` 不会给虚拟机分配一个本地网络地址，也就是说，虚拟机无法通过本地网络地址和宿主机进行通信

**那么解决问题的方案有两种**:

1. 添加一个 `Host Only`: 虚拟网络模式, 虚拟机之间建立一个私有网络，通过虚拟网络适配器实现, 但通过这个虚拟网络，无法访问外部网络和互联网，只能访问宿主机和其他虚拟机。和 `NAT` 网卡结合后，都可以访问
2. 使用 `桥接模式`: 虚拟机和宿主机共享一个 `物理网络`, 虚拟机可以获得本地网络中的 IP 地址，从而可以和宿主机直接通信

### Host Only

#### 配置 Host Only 网络适配器

1. 点击 `管理` -> `工具` -> `Network Manager`, 打开网络管理页面
2. 会发现 virtualbox 会默认安装的时候，创建一个 `Host Only` 适配器
3. 但是 `DHCP` 服务器是默认开启的, 但我的需求是希望每次打开虚拟机的时候，`IP` 都是固定的，因此先关闭 `DHCP`， 手动配置网卡信息

![网络配置](https://img.mjhxyz.top/20230405160549.png)

#### 给虚拟机配置 Host Only 网卡

1. 单击虚拟机，点击`设置` -> `网络` -> `网卡2` -> `启用网络连接` -> `选择 Host Only`
![添加网卡](https://img.mjhxyz.top/20230405160826.png)

2. 启动虚拟机, 输入 `ip addr` 查看网卡信息, 会发现多了一个 `enp0s8`, 但还没有分配到 `IP` 地址
![网卡信息](https://img.mjhxyz.top/20230405161127.png)

3. 手动分配静态 `IP` 地址
```bash
# 切换到 root
su -
# 编辑网卡配置文件
vim /etc/network/interfaces
```
添加如下内容
```bash
auto enp0s8 # 启动时，自动激活网卡
iface enp0s8 inet static # 表示系统使用静态IP地址配置这个网卡，inet 表示使用 IPv4协议
address 192.168.60.100 # 固定的地址
netmask 255.255.255.0 # 子网掩码
gateway 192.168.60.1 # 网关地址, 和之前配置的 Host Only 适配器地址一致
```
4. 保存配置文件后，重启网络服务
```bash
systemctl restart networking
```
5. 此时使用 `ip addr` 后，就能看到 `enp0s8` 网卡已经分配到了 `IP` 地址, 此时宿主机就能和虚拟机进行通信了


### 桥接模式

To Be Continue...