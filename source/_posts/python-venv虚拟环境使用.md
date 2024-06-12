---
title: Python Venv虚拟环境使用
date: 2024-04-29 20:31:07
tags: 
- Python
category: Python
---

venv 模块支持创建轻量的“虚拟环境”，每个虚拟环境将拥有它们自己独立的安装在其 site 目录中的 Python 软件包集合。 虚拟环境是在现有的 Python 安装版基础之上创建的，这被称为虚拟环境的“基础”Python，并且还可选择与基础环境中的软件包隔离开来，这样只有在虚拟环境中显式安装的软件包才是可用的。

## 创建虚拟环境

```shell
python -m venv 环境名称或路径名称
```

## 激活虚拟环境

### linux

```shell
source 虚拟环境路径/bin/activate
```

### windows

```
cd 虚拟环境路径
cd Scripts
activate
```

#### activate 报错解决
```shell
PS C:\Users\crt\Desktop\env\Scripts> .\activate
.\activate : File C:\Users\crt\Desktop\env\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled on
 this system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ .\activate
+ ~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
PS C:\Users\crt\Desktop\env\Scripts>
```

默认情况下在计算机上启动 Windows PowerShell 时，执行策略可能是 Restricted。

想了解 计算机上的现用执行策略，打开 PowerShell 然后输入 get-executionpolicy

默认情况下返回的是 Restricted

Restricted 执行策略不允许任何脚本运行。
AllSigned 和 RemoteSigned 执行策略可防止 Windows PowerShell 运行没有数字签名的脚本。

所以要想执行activate要把策略改为 RemoteSigned

以管理员身份打开PowerShell 输入 set-executionpolicy remotesigned