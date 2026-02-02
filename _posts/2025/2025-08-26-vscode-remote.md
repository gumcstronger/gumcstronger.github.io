---
layout:     post
title:      "远程工作：UU一键远程，iPad第二屏幕,vscode远程代码"
subtitle:   "UU Remote & Ipad Second Screen & Vscode Remote tunnel"
date:       2025-08-26 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
近期因为工作需要，大部分情况下需要使用笔记本工作，奈何笔记本性能不行，只能通过远程来凑。

## UU远程

远程视频选用UU远程登录账号后，可直接从Mac air笔记本远程Windows。甚至可以实现手机开机（暂未测试）。

## iPad作为第二屏幕

~~市面上的软件包括Duet Display(付费)，Yam Display(付费)等软件都需要付费且有可能比较坑。还有想SpaceDesk只支持WIndows。所以最终选用[Deskpad](https://github.com/Stengo/DeskPad)+ [Deskreen](https://github.com/pavlobu/deskreen)方案，因为两者都是开源的。先使用Deskpad开启虚拟第二屏幕，假装有两个屏幕。再使用Deskreen将第二屏幕通过Wifi投射到Ipad上(只要有浏览器就都可以实现)。~~

~~Deskreen似乎这几年没有维护了，目前再Mac OS最新系统上，需要关闭防火墙后才有效。为了安全每次关闭防火墙，连接Deskreen后再重新打开防火墙。~~

Deskreen的延迟太严重，使用Duet Display通过usb连接，记得取消其本地网络设备连接的权限。免费版30分钟重连一次。

## Vscode Remote Tunnel远程代码

直接使用UU远程让笔记本控制台式电脑，键盘和鼠标的输入太慢，打字写代码延迟很久很影响效率。所以改为UU远程控制电脑方便用于Unity调试，而vscode远程代码可以直接在笔记本上写代码，并实时同步到台式机上运行。

  * 在台式主机上的vscode开启Remote Tunnel
  点击右下角Account头像 - Turn On Remote Tunnel Access，使用Github或Microsoft登录。
  * 在笔记本上的vscode安装Remote - Tunnels和Remote Explorer，然后ctrl  + shift + p，输入 Remote-Tunnels: Connect to tunnel，也使用Github或Microsoft账号登录。相同账号登录即可连接。
  * 笔记本电脑vscode - File - Open Folder，直接输入远程台式电脑的文件夹路径，可以直接打开文件夹。
  * 笔记本电脑vscode的扩展界面，所有插件都会有另一个按钮：install in desktop-xxxx。因为台式电脑相当于创建了一个新的vscode服务器运行，这时候所有插件都需要重新安装，所以将需要使用的插件重新点击install in desktop-xxxx安装。如果安装失败，则可能是权限不足，台式主机的vscode改为用管理员打开。

## 如本地局域网可以使用vscode remote ssh来远程

  * 配置Mac的~/.ssh/config加入以下配置来**开启 TCP KeepAlive**，减少重连问题

  ```bash
  # 创建 .ssh 文件夹（如果已经有了也没关系）：
  mkdir -p ~/.ssh

  # 创建 sockets 文件夹（用于存放我们之前提到的连接隧道文件）：
  mkdir -p ~/.ssh/sockets

  # 创建 config 文件：
  touch ~/.ssh/config

  # 设置正确的权限（重要：SSH 对权限要求极严，权限不对会报错）：
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/config

  # 第二步：编辑配置文件
  vim ~/.ssh/config

  # 将以下内容复制并粘贴进去：
  # 这里的 * 代表对所有服务器生效，你也可以把 * 换成特定的服务器 IP 或别名
  Host *
      # 保持连接心跳，防止运营商拔线
      ServerAliveInterval 20
      ServerAliveCountMax 3

      # 开启 SSH 多路复用和后台持久化
      ControlMaster auto
      ControlPath ~/.ssh/sockets/%r@%h-%p
      ControlPersist 1h

      # 允许在后台进行身份验证（可选）
      AddKeysToAgent yes
      UseKeychain yes

  保存并关闭文件。
  ```
