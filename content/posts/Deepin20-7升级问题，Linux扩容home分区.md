---
title: Deepin20.7升级问题，Linux扩容home分区
date: 2022-09-03 00:35:46
tags:
    - 开发
    - Linux
    - Bug
    - Deepin
categories:
    - Linux
---



字数：1711

阅读时长：5分钟

## Linux挂载新home分区方法；Deepin 20.7升级问题

### 推荐体验Deepin Linux

Deepin是我很喜欢的Linux发行版，它贴近Windows的图形界面和丰富的软件生态让新用户能快速上手Linux，有漂亮的DDE(Deepin Desktop Environment)，集成deepin-wine和ART(Android RunTime)，可以轻松运行大部分Windows和Android应用，可以说是Deepin让大部分人开始转移到国内开发者开发的Linux系统（虽然国内主导开发的Linux没几个）。

最近在使用Deepin的过程钟出现了一个有趣的问题：将系统版本从Deepin 20.6升级到Deepin20.7之后，无限循环在登录页面无法进入桌面。后面发现这件事与之前做的另一件事有关系，从头开始说起。

### Linux挂载新home分区

在初装Deepin时，我在我的笔记本上压缩出了60G的空间来装它（挂载 / 和 /home）。我只是想用它来学习Linux，同时作为Android开发的环境（Linux上要比Windows快很多，至少在我的电脑上是这样的），在安装完必要的环境并且工作一段时间后，还剩余将尽20G的空间。

但在后来，我发现Linux上做开发是真的爽（主要是懒，Linux上玩游戏需要一定成本），搭配使用zsh（一款shell，Linux默认为bash）可以省去一些琐碎的步骤，于是渐渐地我将IDEA，PyCharm项目全部转移到Linux，后来学习Rust又安装了Clion。

某天我下载某个文件，发现始终无法下载成功，打开文件管理器，这才看到60G已经被占满了。于是我就想为这块分区再分配一些空间，打开GParted后发现，我的硬盘上竟然还有一块60G的未分配空间，这把我乐坏了，直接把/home挂在这块分区上不就行了嘛。

首先用这块空间新建一个分区，并格式化为ext4，之后：

```bash
# 创建新分区的临时挂载目录
sudo mkdir /media/home

# 把这块分区挂载在这个目录
sudo mount /dev/nvme0n1p6 /media/home 	
# 我的分区是/dev/nvme0n1p6
# 可以使用 lsblk 命令查看各分区

# 同步/home 到 /media/home
sudo rsync -aXS /home/. /media/home/.
# 同步时间与home大小有关

# 将原来的home更名
sudo mv /home /home_past

# 创建新的home
sudo mkdir /home

# 取消分区挂载到/media/home
# 并重新挂载到/home
sudo umount /dev/nvme0n1p6
sudo mount /dev/nvme0n1p6 /home

# 查看并记录分区的UUID
# 写入fstab文件，实现开机自动挂载
blkid 					# 记录对应/dev/nvme0n1p6的UUID
sudo vim /etc/fstab 	# 使用vim编辑fstab文件

# 在文件中添加如下内容：
UUID=你的UUID		/home	/ext4	rw,nodev,nosuid		0,2
# :wq保存，之后重启系统

# 若重启后发现/home分区挂载正常，则可以删除旧分区
sudo rm -rf /home
# 千万不要写 sudo rm -rf / 		手动滑稽
```

在这之后，我就用我的新home分区（60G都真香）工作了很长一段时间，直到那天升级系统......



### Deepin 20.7升级后在登陆界面无法进入系统

我是使用apt升级的系统：

```bash
sudo apt update
sudo apt upgrade
```

一切同往常一样的操作，在重启后：

TNND，输入密码，按下Enter，黑屏，再次回到登陆界面，输入密码，黑屏......

系统从这里开始就**进不去了**。



先是怀疑**升级出错**，于是Ctrl + F2进入tty2，登陆后，再次apt，回到tty1，无效。

后又 `dpkg --configure -a`， 再次进入，无效。

怀疑**DDE**出问题了，于是重装dde:

```bash
sudo apt purge dde dde-desktop dde-dock startdde
sudo apt install dde dde-desktop dde-dock startdde
```

依旧无效。

再次怀疑是**显卡驱动**原因，重新安装**Linux内核**：

```bash
# 搜索内核
apt-cache search linux-image

# 安装内核，同时安装image和headers
sudo apt install linux-image-5.18-amd64-desktop
sudo apt install linux-headers-5.18-amd64-desktop
# 5.18.？？ 具体版本号忘记了。。。
```

再次进入依旧无效。

之后怀疑home**分区挂载**的问题，查看分区情况，并重新挂载一遍：

```bash
# 查看分区情况
lsblk

# 重新挂载
sudo umount /dev/nvme0n1p6
sudo mount /dev/nvme0nqp6 /home

lsblk
```

分区挂载正常。

然后怀疑是**桌面管理器**的问题，于是安装kde试试：

```bash
# 安装kde基本包
sudo apt install kde-plasma-desktop

# 在桌面管理器选择界面有两个选项：
# lightdm sddm
# 前者是deepin默认，后者是kde的
# 桌面管理器可以认为与你的登陆界面有关

# 使用下列命令重写切换桌面管理器
sudo dpkg-reconfigure lightdm
```

重启后，进入了kde的登陆界面，在登陆选项竟然看到了**两个deepin桌面**：

deepin 和 **deepin(wayland)**

deepin使用Wayland还在试验阶段，所以默认桌面就是用X11的deepin，选择deepin登陆，失败。

选择plasma登陆，失败。

选择，deepin(wayland)，**竟然成功了！**

然而进入，这什么东西，一个窗口给我整5倍屏幕大小，卡顿，无法显示chrome.......

那么为什么deepin(wayland)可以进入，而其他都不行呢？这让我更加肯定是显卡驱动的问题，之后又重装了kernel 5.10，5.15，5.18，结果还是没用。

呼~没办法，**重装 **吧！好在我的**home**挂载在了另一块分区，所以重装后我的大部分文件不会丢失。

于是拿出U盘，再次安装。

将系统盘格式化，然后重新挂载为根目录，再把home盘挂载为home。

在漫长的安装后，我终于看到了那个登录界面。好了，开始进入系统。

**又一次失败了**。。。。

这可不对了。再试一次，这次把home挂载在系统盘上。

进入系统，这次终于 **成功了！**

这说明了不是因为系统或驱动等问题，**问题出在home上**。

但在查看了默认配置文件之后，发现并没有跟正常文件有什么区别，问题的原因找不到。。。

好吧，没办法，手动复原。

将之前的home中重要的文件手动复制到新的home，然后把那个分区格了，再用重新挂载home的方法挂载一遍，重启，进入系统了，**问题解决！**

### 总结

这次更新系统出现问题，根源是在home分区，暂时未明了具体原因。

目前的解决方案是：在tty1之外的控制台备份home到新的分区，之后重新挂载home，并复原文件。
