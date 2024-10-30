+++
date = '2024-10-14T15:39:42+08:00'
draft = true
title = 'Git学习笔记（1）'
tags = ['coding', 'git']
categories = ['coding']
+++

### 本地Git向GitHub提交代码

#### 建立SSH连接

初次向github推送自己的代码，需要创建ssh-key

首先在任意目录下打开git bash，键入：

```bash
ssh-keygen -t rsa -C "yourEmail@example.com"
```

会在`~/.ssh`目录下生成两个文件，我们复制公钥：

```bash
clip < ~/.ssh/id_rsa.pub
```

接着进入[SSH and GPG keys (github.com)](https://github.com/settings/keys)

选择`new SSH key`，将公钥粘贴进去即可。

#### 设置本地git

首先，在没有其他特殊需求的情况下，设置全局用户名和邮箱：

```bash
git config --global user.name "yourName"
git config --global user.email "yourEmail@example.com"
```

接着，在你已经配置好github的情况下，测试连接是否正常：

```bash
ssh -T git@github.com
```

#### 进行代码操作

首先将仓库的代码克隆到本地：

```bash
git clone https://github.com/yourName/example.git
```

紧接着，进入到这个目录中去，初始化仓库：

```bash
git init
```

查看仓库状态：

```bash
git status
```

需要注意的是，克隆下来的代码自带git配置，所以不需要在进行分支设置，直接对其进行同步操作即可：

```bash
git pull
```

紧接着可以对仓库内的东西进行修改。



在修改结束后，将仓库内需要更新的文件添加如仓库，一般我们同步全部的资料：

```bash
git add .
```

在这之后我们可以进行代码的提交：

```bash
git commit -m "this is a example"
```

之后将代码同步到云端：

```bash
git push
```

即可完成操作

#### git pull/push 遭遇网络问题

一般来说，github的连接很不稳定，常用VPN进行加速，但因此会使得SSH连接异常，解决方法是将自己git的端口改为同VPN系统代理一样的端口：

```bash
git config --global http.proxy http://127.0.0.1:<端口>
git config --global https.proxy https://127.0.0.1:<相同的端口>
```

特别地，如果需要sock5代理，也是如下操作：

```bash
git config --global http.proxy socks5://127.0.0.1:<端口>
git config --global https.proxy socks5://127.0.0.1:<相同的端口>
```

