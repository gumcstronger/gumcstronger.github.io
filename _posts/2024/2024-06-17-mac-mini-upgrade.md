---
layout:     post
title:      "Mac Mini Late 2014加装SSD和使用OpenCore Legacy Patcher安装macOS Sonoma"
subtitle:   " \"Unity常用插件和教程\""
date:       2024-06-17 11:15:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开源
---
Mac Mini Late 2014是很早一个版本的Mac，性能较低，并且很多人都是只购买纯HDD。并且无法使用macOS Ventura和macOS Sonoma，因为苹果除了M1、M2版本后已经不再支持Intel的设备升级到新系统，而打包iOS需要升级到Sonoma，只能使用OpenCore Legacy Ptcher。

设备购买：

梅花螺丝刀：T6中孔

SSD：SAMSUNG 970 EVO Plus | NVME PCIe 3.0 * 5  250GB

PCIe与NVMe转接卡：NFHK NVME x4 M.2 NGFF转Late 2014硬盘扩展卡

SSD安装流程：[Mac Mini Late 2014固态硬盘替换](https://zh.ifixit.com/Guide/2014%E5%B9%B4%E6%9C%AB%E6%AC%BE+Mac+Mini+%E5%9B%BA%E6%80%81%E7%A1%AC%E7%9B%98%E6%9B%BF%E6%8D%A2/32646)

OpenCore Legacy Patcher安装Sonoma流程：[OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/INSTALLER.html)

前置：

* [固件升级](https://youroptibay.ru/manual/MacBook-manuals-index/macbook-retina-manuals/macbook-pro-retina-13-repair/macbook-pro-retina-13-late-2013-repair/ustanovka-ssd-samsung-970-evo-plus-v-mac/)：[unetbootin](https://unetbootin.github.io/), [固件](https://semiconductor.samsung.com/consumer-storage/support/tools/) （[论坛说明](https://forums.macrumors.com/threads/samsung-m-2-970-evo-plus-firmware-upgrade-problem-solved.2220069/)）

注意：

* U盘启动器不能含有只读盘，如有的U盘中被写入部分只读信息，则无法制作启动盘（花费了我大半天时间）
* Sonoma需要32G的U盘，如U盘只有16G，可先使用Ventura走完整个流程然后再升级到Sonoma
* 如是加装SSD再安装Sonoma，可先在SSD上普通系统，然后再使用OpenCore Legacy Patcher，可以加快电脑速度加快流程。
* ReportCrash可能导致死循环，Crash然后Report，而Report又Crash。可以禁用ReportCrash
  ```
  launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
  sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
  ```
* 如需要重置，需要先卸载OpenCore Legacy Patcher，重置NVRAM，删除EFI分区，不然无法重新安装原先的系统。
  * 如果系统已经卸载，则可以进入安装或恢复，打开终端，输入：diskutil list 可以看到对应的磁盘
  * 然后输入diskutil eraseDisk JHFS+ XXXX MBR diskX  (将XXXX替换成希望的命名，diskX为实际磁盘标识符例如disk0)