---
layout:     post
title:      "Unity连接Mumu查看Logcat"
subtitle:   "Unity show android logcat for Simulator Mumu"
date:       2025-09-17 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
## 查看mumu的端口

在mumui - 设置中心 - ADB端口，默认为16384

## adb连接端口

 [官方参考](https://mumu.163.com/help/20230214/35047_1073151.html)

* 打开mumu自带的adb所在文件夹，默认在：C:\Program Files\Netease\MuMu\nx_device\12.0\shell, 右键打开powershell
* ，输入
  ```
  adb.exe connect 127.0.0.1:16384
  ```

* 查看Devices

  ```
  adb devices
  ```

  查看当前的设备
  可以看到有一行：`127.0.0.1:16384 device`
* shell
  ```
  adb -s 127.0.0.1:16384 shell
  ```
  实际上在目标安卓设备上打开一个它的原生命令行界面，并把这个界面‘投射’到当前的电脑终端上

## Unity

unity打包界面或Logcat界面，可以直接选择到127.0.0.1:16384的设备选项。