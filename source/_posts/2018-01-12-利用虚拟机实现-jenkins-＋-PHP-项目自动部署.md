---
title: 利用虚拟机实现 jenkins ＋ PHP 项目自动部署
date: 2018-01-12 10:17:36
tags:
- Jenkins
- 运维
---

> **前言**
> 记录一次利用虚拟机（两台）实现本地的 jenkins 环境搭建、PHP 代码运行环境搭建以及使用 jenkins 实现 PHP 代码的自动部署

<!-- more -->

## 环境搭建

分为 jenkins 环境搭建和 PHP 环境搭建，jenkins 环境搭建在器一台虚拟机上面（虚拟机1），搭建过程详见：[Jenkins环境搭建]()

下面简单说一下 PHP 环境的搭建，这里使用 [OneinStack](https://oneinstack.com/) 来进行搭建，具体方式详见 [安装 - OneinStack](https://oneinstack.com/install/)

### Git 安装

在虚拟机 1 中（安装 Jenkins 的机器）需要安装 git 才能够拉取代码

这里笔者的虚拟机安装的是 CetOS 自带 git

### python 环境搭建

因为自动部署代码有使用到 python 编写，所以同时也要安装 python 环境

具体代码：

```bash
sudo yum install -y gcc libffi-devel python-devel openssl-devel
```

接下来是 python 的 pip 环境安装、需要先行安装 epel-release

```bash
yum install -y epel-release
yum install -y python-pip
```

升级 pip

```bash
pip install --upgrade pip
```

### absible 安装

```bash
yum install -y ansible
```

输入 `ansible --version` 能够获取到 ansible 版本号即为安装成功

---

## 自动部署思路

自动部署包括机器管理、环境检查、代码部署、服务管理、错误通知等内容。

这里先实现最简单的几步：机器管理、代码部署和服务管理，实现通过 jenkins 对 PHP 代码进行拉取和部署、重启服务等。

具体方式是利用 jenkins 进行自动部署过程的触发，拉取 PHP 代码，然后触发 jenkins 所在机器（虚拟机1）上面的 ansible 的 playbook 任务，连接到本地安装了 PHP 环境的机器（虚拟机2）进行代码推送和服务重启等任务。

---

## 实施过程

### 安装 jenkins 插件

由于整个自动部署过程是从 jenkins 上面进行触发的（这里不选用代码管理库提交代码即通知 jenkins 进行构建的方式），所以我们需要先创建 jenkins 上面的任务，任务会从两个地方拉取代码（PHP 项目代码和部署脚本代码），然后触发部署过程。

要实现从多个地方拉取代码需要安装以下插件：[Multijob plugin]、[Multiple SCMs plugin]。

点击首页右侧的 “系统管理” - “管理插件” - “可选插件” 里面进行安装，勾选“安装完成后重启Jenkins(空闲时)”，即可。等待 jenkins 重启完成之后重新登陆 jenkins 即可进行下一步操作。

### 代码库登录秘钥管理

点击首页右侧的 “Credentials” 进去秘钥管理界面。按照下图方式添加一个对应代码库的域名（如github.com）

![添加域名](/domin.png "添加域名")

接下来是为代码空间添加登录秘钥了（用来拉取代码的账号名和密码）

![添加秘钥](/Credentials.png "添加秘钥")

### 添加及配置 jenkins 任务

接下来就是要添加 jenkins 任务了，点击首页右侧的“新建” - “构建一个自由风格的软件项目”

在“源码管理一栏” - “Multiple SCMs” 去添加要拉取的 PHP代码以及部署脚本代码

需要注意的是要额外设置一个 “Additional Behaviours"：“Check out to a sub-directory” 来定义拉取后的代码放到什么目录

![任务设置](/mission_settings.png "任务设置")

设置完成之后点击保存，然后返回去到任务操作界面，点击“立即构建”如无意外（代码库秘钥不通过、或者虚拟机 1 的 git 没有安装或者异常等）的话，会构建成功，此时点击“工作空间”即能够看到对应的两个代码库的代码已经被正确放到对应的文件夹中

![工作空间](/workspace.png "工作空间")
