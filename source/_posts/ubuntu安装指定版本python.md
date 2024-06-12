---
title: Ubuntu安装指定版本Python
date: 2024-04-29 20:41:18
tags: 
- Python 
- Ubuntu
category: Linux
---

通过源码方式编译安装python

## 更新软件包列表并安装构建Python所需的软件包

```shell
sudo apt update
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget
```

## 下载、解压python源代码

- 下载
```shell
wget https://www.python.org/ftp/python/3.x.x/Python-3.x.x.tgz
```

- 解压
```shell
tar -xf Python-3.x.x.tgz
```

## 编译安装python

```shell
cd Python-3.x.x

# ./configure --enable-optimizations
# 或者使用上面的命令，–-enable-optimizations选项通过运行多个测试来优化Python二进制文件，这会使构建过程变慢
./configure 

make -j 8 # 为了加快构建时间，请修改-j以使其对应于处理器中的内核数

sudo make altinstall # 不要使用标准的make install，因为它将覆盖默认的系统python3二进制文件

python3.x # 进入解析器
```