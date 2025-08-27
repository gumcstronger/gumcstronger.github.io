---
layout:     post
title:      "Vscode实现远程工作"
subtitle:   "Vscode Remote tunnel"
date:       2025-08-26 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
近期因为工作需要，大部分情况下需要使用笔记本工作，奈何笔记本性能不行，只能通过远程来凑。

而直接使用UU远程让笔记本控制台式电脑，键盘和鼠标的输入太慢，打字写代码延迟很久很影响效率。所以改为UU远程控制电脑方便用于Unity调试，而vscode远程代码可以直接在笔记本上写代码。

* 在台式主机上的vscode开启Remote Tunnel
  点击右下角Account头像 - Turn On Remote Tunnel Access，使用Github或Microsoft登录。
* 在笔记本上的vscode安装Remote - Tunnels和Remote Explorer，然后ctrl  + shift + p，输入 Remote-Tunnels: Connect to tunnel，也使用Github或Microsoft账号登录。相同账号登录即可连接。
* 笔记本电脑vscode - File - Open Folder，直接输入远程台式电脑的文件夹路径，可以直接打开文件夹。
* 笔记本电脑vscode的扩展界面，所有插件都会有另一个按钮：install in desktop-xxxx。因为台式电脑相当于创建了一个新的vscode服务器运行，这时候所有插件都需要重新安装，所以将需要使用的插件重新点击install in desktop-xxxx安装。如果安装失败，则可能是权限不足，台式主机的vscode改为用管理员打开。
