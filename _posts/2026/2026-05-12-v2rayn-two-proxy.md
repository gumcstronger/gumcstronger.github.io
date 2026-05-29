---
layout:     post
title:      "AI Cli"
subtitle:   "Claude Code / Codex"
date:       2026-04-09 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - System
---

需求：使用v2rayN实现A走A代理，B走B代理。

**v2rayN 真正的转发逻辑在底部的** **“路由”** **里。**

* **点击上方菜单的** **“设置”** **->** **“路由设置”**。
* **点击**  **->** **“添加”**。
* **起个名字（比如** **A_B_Split**）。
* **添加第一条规则**：

  * **outboundTag**：选 **proxy**（这代表走你当前**激活**的那个服务器）。
* **Domain**：填入域名 A。
* **添加第二条规则**：

  * **outboundTag**：选 **direct** **或者你定义的另一个出口。**
* **难点**：v2rayN 的 GUI 界面在处理“两个不同的远程代理”时非常笨拙。它默认只支持 **proxy**（当前的）和 **direct**（直连）。
