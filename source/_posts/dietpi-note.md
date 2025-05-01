---
title: DietPi 安装与使用笔记
date: 2025-05-01 18:49:45
tags:
---

DietPi 是一个高度定制化的轻量级 Debian OS，比较适合有经验的开发人员或者资深爱好者使用。其安装过程稍显复杂，额外提供的功能也值得使用。本文将探讨 DietPi 的手工安装过程以及后续使用相关功能时可能遇到的问题。



## 0. 开始前

建议先浏览这些文章

https://wiki.friendlyelec.com/wiki/index.php/DietPi/zh

https://dietpi.com/docs/

一些设备具有板载 eMMC 。

如果你想将已经在使用的 DietPi 迁移到板载eMMC中，请阅读下面这篇文章。

如果你想直接安装 DietPi 到板载 eMMC 中，可以参考下面这篇文章的 7.1.2 章节。

https://dietpi.com/blog/?p=1236



## 1. 下载 DietPi 镜像

前往下载页，在设备列表中找到目标设备。点击图片，展开对应设备的详细说明。有的设备会提供多种镜像，提供额外的功能，可以根据需求选择。

https://dietpi.com/#download



## 2. 将镜像写入到 TF 卡

这真的很简单。我喜欢用 Rufus 来写入。



## 3. 修改预配置文件


写入完成后，你会发现 TF 卡中出现了两个分区。其中的 FAT 分区中有一些配置文件，用于预配置和自动化 DietPi 设置。

这些文件将在第一次启动时被复制到根文件系统中，被修改的设置将生效，并且该分区会被删除。

dietpi.txt 本身就有详细的注释，可以耐下心读完，确认你需要修改哪些地方以及正确的修改方法。

DietPi 的首次启动向导需要一定程度的用户交互，但它有提供无人值守安装的选项，详见下面的文章。

https://dietpi.com/docs/usage/#how-to-do-an-automatic-base-installation-at-first-boot-dietpi-automation

本文不介绍如何进行无人值守安装。

dietpi.txt 中我修改的地方：

```
AUTO_SETUP_LOCALE=zh_CN.UTF-8
AUTO_SETUP_KEYBOARD_LAYOUT=us
AUTO_SETUP_TIMEZONE=Asia/Shanghai

AUTO_SETUP_NET_WIFI_COUNTRY_CODE=CN
AUTO_SETUP_NET_HOSTNAME=DietPi # 改成自己想要的主机名

CONFIG_APT_RASPBIAN_MIRROR=https://mirrors.ustc.edu.cn/raspbian/raspbian/
CONFIG_APT_DEBIAN_MIRROR=https://mirrors.ustc.edu.cn/debian/
```

{% notel blue fa-lightbulb 信息 %}
在 ```/etc/apt/sources.list``` 中：
```https://deb.debian.org/debian-security/```
不受 ```CONFIG_APT_DEBIAN_MIRROR``` 的影响，你可以在安装向导执行时打开一个新终端来修改它。
```https://mirrors.ustc.edu.cn/debian-security/```
{% endnotel %}

{% notel blue fa-lightbulb 信息 %}
Debian 镜像源：
https://www.debian.org/mirror/official#list
https://mirrors.ustc.edu.cn/help/debian.html

Raspbian 镜像源：
https://www.raspbian.org/RaspbianMirrors
https://mirrors.ustc.edu.cn/help/raspbian.html

Raspberrypi 镜像源：
https://mirrors.ustc.edu.cn/help/raspberrypi.html
```
sudo sed \
  -e 's|archive.raspberrypi.org|mirrors.ustc.edu.cn/raspberrypi|g' \
  -e 's|archive.raspberrypi.com|mirrors.ustc.edu.cn/raspberrypi|g' \
  -i.bak \
  /etc/apt/sources.list.d/raspi.list
```
{% endnotel %}

{% note info %} ```http://ftp.cn.debian.org/debian/``` 可以将你重定向到国内的镜像源，但它不能使用https。 {% endnote %}



## 4. 在 DietPi 首次启动向导中可能遇到的问题

### 联网更新 DietPi 时

报错：```curl: (18) HTTP/2 stream 1 was not closed cleanly before end of the underlying stream```

原因：网络环境不佳，未能与 GitHub 建立连接。

解决方法：选择 ```Change command``` ，修改命令为

```
curl -sSfLO https://gh.llkk.cc/https://github.com/MichaIng/DietPi/archive/master.tar.gz
```

原命令

```
curl -sSfLO https://github.com/MichaIng/DietPi/archive/master.tar.gz
```

GitHub文件代理加速：
https://github.akams.cn/



## 5. 在 Dietpi-Software 中安装软件时可能遇到的问题

### fail2ban

问题：下载的内容直接输出到终端，并报错 ```curl: (3) URL using bad/illegal format or missing URL```

原因：原命令有错误？

解决方法：选择 ```Change command``` ，修改命令为

```
curl -sSf https://raw.githubusercontent.com/fail2ban/fail2ban/master/config/filter.d/dropbear.conf -o /etc/fail2ban/filter.d/dropbear.local
```

原命令

```
curl -sSf https://raw.githubusercontent.com/fail2ban/fail2ban/master/config/filter.d/dropbear.conf /etc/fail2ban/filter.d/dropbear.local

```

### python3

问题：安装 pip 速度慢。

原因：网络环境不佳。

解决方法：使用镜像源。

选择 ```Change command``` ，修改命令为

```python3 get-pip.py -i https://mirrors.ustc.edu.cn/pypi/simple --trusted-host mirrors.ustc.edu.cn```

原命令

```python3 get-pip.py```

PyPI 镜像源：
https://mirrors.ustc.edu.cn/help/pypi.html


