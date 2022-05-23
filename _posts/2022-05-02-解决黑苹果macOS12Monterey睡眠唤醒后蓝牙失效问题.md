---
layout: mypost
title: 解决黑苹果macOS12 Monterey睡眠唤醒后蓝牙失效问题
categories: [Tips]
---

## 前言

自己笔记本电脑是装的黑苹果，之前在Catalina里蓝牙很完美，然后昨天手贱格盘升级到12 Monterey。其他地方还好说，就是唯独这个蓝牙十分头疼。

具体问题为：

- 睡眠唤醒后，蓝牙失效。蓝牙键盘无法自动连接上，AirPods也是如此
- 手动去偏好设置中重新关闭再开启蓝牙后，有一定几率可以恢复，也有可能只能重启电脑恢复

远景论坛里查了下据说Monterey下黑苹果的蓝牙似乎都多少存在问题，是因为苹果在Monterey系统中删除了许多老旧的蓝牙模块，重写了蓝牙相关的代码。甚至导致一些白果Mac蓝牙都时不时出问题。所以现如今黑苹果的蓝牙能跑起来就已经算是感恩戴德了……

介于直到目前macOS 12.3.1版本官方仍没有修复该bug的意图，为了在Monterey上优雅地吃上黑苹果，我们只能找一些奇淫巧技来暂时地解决这个问题了。

在网上找了许久，终于发现一种曲线救国的办法：利用sleepwatcher和blueutil实现，电脑睡眠时自动关闭蓝牙和WiFi，唤醒时自动开启蓝牙和WiFi。以此规避掉睡眠蓝牙失效的问题，同时还可以预防WiFi时不时睡醒后抽风的问题。下面是整理出来的具体解决步骤：

## 准备工作

因为用到的`sleepwatcher`和`blueutil`都是Homebrew上面的包，所以最好首先安装Homebrew包管理器。当然如果你发现可以在其他地方直接下载安装，那对于没有用过Homebrew的非程序员网友们来说更好不过了。

在`终端.app`中键入命令，安装Homebrew（此过程比较耗时）：

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

如果你没有科学上网的方式，且嫌弃从GitHub上下载速度缓慢，可以试试[更换国内的源](https://brew.idayer.com/guide/change-source/)。

## 正式步骤

利用Homebrew安装用到的两个包：

```shell
// 安装sleepwatcher和blueutil
brew install sleepwacher
brew install blueutil
```

设置系统自启动sleepwacher后台进程：

```shell
brew services start sleepwatcher
```

这一步需要输入你自己的用户登录密码。

执行完毕后可以检查后台进程是否添加成功：

```shell
ps aux | grep sleepwatcher
```

如果添加成功，则会显示包含 `/usr/local/opt/sleepwatcher/sbin/sleepwatcher -V -s /Users/leon/.sleep -w /Users/leon/.wakeup`的一长串信息。

设置自动化命令，写入文件，赋予权限：

```shell
// 将自动化命令写进 ~/ 下的.sleep和.wake文件中
echo '/usr/local/bin/blueutil -p 0
networksetup -setairportpower en0 off' > ~/.sleep

echo '/usr/local/bin/blueutil -p 1
networksetup -setairportpower en0 on' > ~/.wakeup

// 赋予两个文件权限
sudo chmod 777  ~/.sleep
sudo chmod 777  ~/.wakeup
```

这一步需要输入你自己的用户登录密码。并且会在你自己的用户根目录下创建两个隐藏的文件.sleep和.wake。

至此，利用sleepwatcher和blueutil实现电脑睡眠自动关闭蓝牙和WiFi，唤醒自动打开蓝牙和WiFi的艰巨工作已经完成。可以自己测试效果，静候奇迹的发生！

