---
layout:   post
title:    "禁用微信WeChatAppEx进程"
subtitle: "禁用微信WeChatAppEx进程"
date:     2026-01-22 09:15:00
language: zh-CN
author: "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - System
---
## 问题描述

打开Windows微信电脑端，WeChatAppEx进程好多个，每个占用部分CPU和内存。而且关闭WeChatAppEx进程后还会自动打开。

## 解决方法

关闭微信，在任务管理器右键WeChatAppEx进程，打开文件所在的位置，一般是文件夹C:\Users\XXX\AppData\Roaming\Tencent\WeChat\XPlugin\Plugins\RadiumWMPF，删除RadiumWMPF文件夹后重新创建RadiumWMPF，右键属性-安全，对每个组或用户名点击编辑，在拒绝那一列勾选写入，这样禁用写入权限。

之后再重新打开微信，就不再有WeChatAppEx进程。

禁用之后不能再使用小程序的功能

  - 20260301

发现微信非常鸡贼地改到了C:\Users\XXX\AppData\Roaming\Tencent\\xwechatXPlugin\Plugins\RadiumWMPF

同样禁用写入权限
