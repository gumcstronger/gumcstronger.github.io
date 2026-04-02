---
layout:     post
title:      "解决MacOS Sequoia 15连接蓝牙/无线鼠标后导致强制进入安全模式"
subtitle:   "跨境收款平台对比"
date:       2026-03-09 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - Mac
---
## 状况

Macbook Air 2015出问题的情况:

系统支持版本太低，使用OpenCore  Legacy Patcher升级到MacOs Sequoia。能顺利使用新系统。

无线网卡损坏，使用USB网卡并且安装Wireless-USB-OC-Big-Sur-Adapter第三方驱动。能顺利使用无线网络。

但当插入DOAL-MODE MOUSE(可以同时使用WIFI和蓝牙的鼠标)的USB模块且连接蓝牙后，下次重启就会强制进入安全模式。之后无论怎么重启都只会进入安全模式。如果删除蓝牙后放置一段时间不开机后，再次开机就可以正常进入。

## 分析问题

从 Big Sur 开始，苹果正式宣布废弃传统的内核扩展（Kernel Extensions，即 .kext 文件），要求所有网卡、鼠标、键盘驱动开发者改用运行在用户空间的 NetworkExtension 和 DriverKit 框架。
在这个阶段，Realtek 等厂商直接宣布停止开发 Mac 驱动[[1](https://www.google.com/url?sa=E&q=https%3A%2F%2Fvertexaisearch.cloud.google.com%2Fgrounding-api-redirect%2FAUZIYQH9vQdFyxHSlKpXo7bEo0LOBFSWtMLFq0p8df0lW6wfZNpmFp8DD4EFmPRntNXsTqUJM9lKS7m5eetq4fSs_iMB-_b0sWRsgpGJTFT_v8YLtbkYGMatcLN6ozsmUh_N8sao5N8djPiLowfNQna4ZQBir-2oQijRpLW471zwW4HYqyfa4xwjuyK6z10eG-xI6YJZa_ljR8j7dcWmnA%3D%3D)]。但由于底层还没完全封死，像 chris1111 这样的民间大神还能通过关闭 SIP，把旧驱动“硬塞”进内核里，此时勉强能用，只是偶尔不稳定[[1](https://www.google.com/url?sa=E&q=https%3A%2F%2Fvertexaisearch.cloud.google.com%2Fgrounding-api-redirect%2FAUZIYQH9vQdFyxHSlKpXo7bEo0LOBFSWtMLFq0p8df0lW6wfZNpmFp8DD4EFmPRntNXsTqUJM9lKS7m5eetq4fSs_iMB-_b0sWRsgpGJTFT_v8YLtbkYGMatcLN6ozsmUh_N8sao5N8djPiLowfNQna4ZQBir-2oQijRpLW471zwW4HYqyfa4xwjuyK6z10eG-xI6YJZa_ljR8j7dcWmnA%3D%3D)][[2](https://www.google.com/url?sa=E&q=https%3A%2F%2Fvertexaisearch.cloud.google.com%2Fgrounding-api-redirect%2FAUZIYQFfrhZqlsT0Xonukhzpfex5AJ8_QUEJevTfJtOBlCygiVemoX7pAa1UeJMg02R1ynGawfGodiGnHLoqR6KTmcGut-Bd0vOVSNqlNfjDBVNRaHeqMuDm6IL_zZumcgBcJJk9_m2N5atoyso2QAeXj_68G_Pf)]。

到了 Sonoma，苹果做了一个大动作：彻底删除了系统内核中老旧的 IO80211FamilyLegacy（传统无线网卡堆栈），并大幅修改了 USB 栈。为了让你电脑自带的老网卡能用，OCLP 必须强行向 Sonoma 注入一个叫 IOSkywalkFamily 的底层核心文件。当你安装了第三方的 USB 网卡驱动（比如 Wireless-USB-OC-Big-Sur-Adapter），这个驱动在工作时，会和 OCLP 注入的 IOSkywalkFamily 在争夺底层网络控制权时发生严重冲突。但这种可能，我看Wireless-USB-OC-Big-Sur-Adapter是支持OpenCore Legacy Patcher和Sequoia的，大概率不是这个问题，那么我们分析其他原因：

  * 如果是USB网卡的问题，但我安装Wireless-USB-OC-Big-Sur-Adapter驱动后一直能正常使用。（有一种可能使用Wireless-USB-OC-Big-Sur-Adapter驱动的WIFI图标修改软件的导致的冲突）但这种可能性较低，因为我安装后一直能正常使用。
  * DUAL MODE MOUSE的2.4G接收器的问题：当插入接收器，macOS 会通过设备的Vendor ID (VID) 和 Product ID (PID) 来判断这是什么设备。如果系统有内置驱动 ：它会直接加载 macOS 内建的 Kext 或 DriverKit 驱动。如果是新设备/旧设备：对于老 Mac 上的 OCLP 环境，它会加载 OCLP 为了支持你这个老 Mac 的 USB 接口而提前注入的通用 USB 驱动补丁。OCLP 必须确保 Mac 的所有 USB 接口能工作，所以它注入了一些底层补丁。点击“重新启动”时，系统需要“安全地”卸载所有正在运行的 Kext 和 DriverKit 扩展。2.4G 接收器的驱动（可能是 OCLP 注入的通用 USB 驱动，或者是设备内部的 HID 描述符被旧补丁错误识别了）在被强制卸载时，没有正确响应内核指令，导致内核崩溃 (Kernel Panic)。系统底层自动识别了该 USB 设备的 PID/VID，并强制加载了 OCLP 帮你打上的、用来支持老 USB 接口的底层补丁/Kext。正是这个补丁在重启卸载时崩溃了
  * DUAL MODE MOUSE的蓝牙协议问题：旧蓝牙鼠标”可能使用的是一个非常老的蓝牙协议版本。当 OCLP 注入的补丁尝试用 Sonoma 的新蓝牙框架（IOBluetoothFamily）去和这个老协议握手时，如果补丁本身没有完全适配 Sonoma 的新接口，就会在握手完成的瞬间引发一个无法恢复的内核崩溃。死锁机制触发：系统检测到“鼠标连接”这个行为引发了崩溃，它会记录下来。下次重启时，系统（特别是启动到一定程度时）会自动尝试重新初始化蓝牙设备（即使你把鼠标关了，它也会尝试连接已配对设备），一旦初始化这个老旧的蓝牙芯片模块，就立即触发崩溃，再次进入安全模式。

## 解决问题思路

  * 先使用USB启动盘降级到Ventura(按Atl，进入时选择EFI BOOT图标的)
  * 安装第三方USB驱动，确定可以运行和访问WIFI，重启是否进入安全模式。
  * 连接蓝牙鼠标和键盘，查看是否有问题。
  * 插入2.4G接收机蓝牙和键盘，查看是否有问题。
  * 升级到Sequoia系统
  * 先连接蓝牙鼠标（不要插入2.4G接收器）
  * 如果以上哪一步出了问题，就能确定是哪一步的问题。如果以上都没问题，那么也不用插入2.4G接收器了，确定是接收器问题。
