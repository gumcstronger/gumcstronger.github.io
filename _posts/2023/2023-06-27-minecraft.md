---
layout:     post
title:      "Minecraft日记"
subtitle:   "\"Minecraft\""
date:       2023-06-27 12:08:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
---
## 启动器

### 启动器PCL

* 下载[PCL](https://github.com/Hex-Dragon/PCL2)
* 解压到D:/Data/Game中
* 打开Plain Craft Launcher 2

### 官方启动器Minecraft Launcher（推荐）

* 进入Minecraft Store，搜索Minecraft并支付89元购买官方版本，下载Minecraft Launcher并运行
* 将.minecraft放到自定义目录
  * 关闭minecraft
  * 将C:\Users\用户名]\AppData\Roaming\\.minecraft移动到D:\Data\Game\Minecraft\\.minecraft
  * 管理员身份运行cmd
  * 输入命令: mklink /j "C:\Users\\[用户名]\AppData\Roaming\\.minecraft" "D:\Data\Game\Minecraft\\.minecraft"
  * 当提示为 Junction created for xxx<<===>>xxx，并且目录下出现一个快捷方式图标的.minecraft文件夹，就表示成功了

## 安装Mod

### 安装Mod加载器

* 需要了解Mod支持的mod加载器，一般为Forge或Fobric
* 打开PCL2 - 下载 - 自动安装 - 1.20.1(选择想要的版本号) - 点击选择Forge或Fabric/Fabric API，会提示当前版本支持的Forge或Fabic版本，选中对应版本 - 点击开始安装 - 等待下载完成
* 点击启动 - 版本选择 - 选择需要的版本 - 启动游戏

### 安装Mod

* 安装[Terra Plus Plus](https://www.curseforge.com/minecraft/mc-mods/terraplusplus)：利用公共在线数据集以 1 比 1 的比例生成地球的结构和特征，没有任何不熟悉的方块或生物群系的 mod，build the world需要支持
  * 下载当前minecraft版本支持的mod，然后放到
* 待定

## Build The Earth

* 下载Minecraft Launcher并购买Minecraft正版
* 下载[Build The Earth安装器](https://buildtheearth.net/faq),并运行install
* 

## 常用指令

* 打开指令框：T
* 飞行指令：创造模式双击空格
* 加载schem：
  * 将filename.schem放到.minecraft/versions/版本/config/worldedit/schematics中
  * //schem load [文件名如JP-Shibuya] [文件格式如schem]
  * //paste
* 待定
