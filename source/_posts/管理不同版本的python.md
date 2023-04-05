---
title: 管理不同版本的python
abbrlink: 63997
date: 2023-04-05 10:41:57
tags:
- python
- 环境
- 版本管理
categories:
- python
---

# 管理不同版本的 python

有时候项目需要使用不同版本的 python，比如项目 A 需要 python 3.6，项目 B 需要 python 3.7，这时候就需要管理不同版本的 python。

之前我遇到这个问题的时候, 是这样处理的:

1. 最开始每一次跑项目的时候, 都重新安装，配置一下系统的 python 版本
2. 后来使用虚拟环境模块, 通过指定 `--python` 路径, 创建一个个版本不相同的虚拟环境, 使用的时候执行一下 `active`

不管是上面的哪个方法，用起来都十分的不方便, 因此后续我就使用了 `conda`,  `pyenv` 来管理不同版本的 python。

## 虚拟环境管理器

我的理解，使用 `conda` 和 `pyenv` 相比手动创建不同版本的虚拟环境有以下优点：

1. 管理更加方便: 都提供了命令行工具来管理不同版本的 python 环境
2. 管理多个环境: 都支持在一个系统中管理多个版本的 python，相互之间不会干扰, 不用担心冲突
3. 空间优化: 使用后，同一系统会共享下载的包，这样就不用每一次创建虚拟环境，都要重新下载相同的包，省时，省空间

总的来说，使用 conda 和 pyenv 可以极大的简化管理 python 的成本，并保证项目使用了正确版本的 python 和所需的库。

## pyenv 使用

### 安装

**windows**

1. 使用 PowerShell 安装
```powershell
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

2. 重新打开控制台， 输入命令来查看安装的版本, 和其他信息
```powershell
# 查看安装的版本
pyenv --version
# 查看支持的 python 版本
pyenv install -l
```

**Linux**

TODO

### 创建虚拟环境并使用

**Windows**

1. 下载指定版本的 python
```powershell
pyenv install 3.7.8
:: [Info] ::  Mirror: https://www.python.org/ftp/python
:: [Installing] ::  3.7.8 ...
:: [Info] :: completed! 3.7.8
```
如果提示下载超时了，则使用国内的镜像下载后，拷贝到 pyenv 的缓存目录下 https://registry.npmmirror.com/binary.html?path=python/3.7.8/ 再执行 install

2. 设置全局默认的 python 版本
```powershell
pyenv global 3.7.8
# 设置完成后，查看 python 版本 
python --version
```

- pyenv global <version> 设置全局默认的 Python 版本。
- pyenv local <version> 在当前目录下设置特定的 Python 版本。
- pyenv shell <version> 在当前终端会话中设置特定的 Python 版本。

3. 通过设定当前目录下指定的版本后，再使用 `virtualenv` 模块创建虚拟环境，对于我来说，就够了
4. 还有 `pyenv-virtualenv` 插件，可以直接创建虚拟环境，不需要再使用 `virtualenv` 模块, 也可以去尝试一下


**Linux**

TODO

## conda 使用

### conda 常用的发行版本

- Miniconda: 包含 conda 和 Python，但不包含其他包，占用空间小
- Anaconda: 包含 conda、Python 和一堆数据科学的软件包(numpy,padas 等), 还有可视化界面

使用 `miniconda` 进行演示

### 安装

**windows**

1. 下载 `miniconda` 安装包 [下载页面](https://docs.conda.io/en/latest/miniconda.html)
2. 后续按照提示一步一步安装即可

### 创建虚拟环境并使用

1. 安装 `minicoada` 后，打开 `Anaconda Prompt`
2. 创建新环境(可以指定 python 版本, 不指定的话则使用默认版本)
```shell
conda create --name condatest python=3.9
```
3. 激活新环境
```shell
conda activate condatest
```
4. 激活以后，就能用这个环境来操作了
```shell
(condatest) PS C:\Users\mao> python -V
Python 3.9.16

(condatest) PS C:\Users\mao> pip -V
pip 23.0.1 from D:\tools\miniconda3\envs\condatest\lib\site-packages\pip (python 3.9
```
5. 安装依赖的时候，最好使用 `conda` 来安装
    - 因为 `conda` 有自动处理依赖关系的能力
    - 使用 `pip` 安装软件包时，不会考虑其他已安装的 `conda` 软件包, 可能出现不兼容的问题
```shell
conda install requests
```
6. 最后使用离开虚拟环境
```shell
conda deactivate
```
