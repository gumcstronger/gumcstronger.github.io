---
layout:     post
title:      "Minecraft笔记"
subtitle:   "\"Minecraft\""
date:       2023-06-27 12:08:00
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
## 启动器

### 官方启动器Minecraft Launcher（推荐）

* 进入Minecraft Store，搜索Minecraft并支付89元购买官方版本，下载Minecraft Launcher并运行
* 将.minecraft放到自定义目录
  * 关闭minecraft
  * 将C:\Users\用户名]\AppData\Roaming\\.minecraft移动到D:\Data\Game\Minecraft\\.minecraft
  * 管理员身份运行cmd
  * 输入命令: mklink /j "C:\Users\\[用户名]\AppData\Roaming\\.minecraft" "D:\Data\Game\Minecraft\\.minecraft"
  * 当提示为 Junction created for xxx<<===>>xxx，并且目录下出现一个快捷方式图标的.minecraft文件夹，就表示成功了
* minecraft存档路径
  * Java版：C:\Users\\[用户名]\Appdata\Roaming\.minecarft\saves
  * Bedrock版：C:\Users\\[用户名]AppData\Local\Packages\Microsoft.MinecraftUWP_8wekyb3d8bbwe\LocalState\games\com.mojang\minecraftWorlds

### 启动器PCL

* 下载[PCL](https://github.com/Hex-Dragon/PCL2)
* 解压到D:/Data/Game中
* 打开Plain Craft Launcher 2

### 启动器HMCL

* 下载[HMCL](https://github.com/Hex-Dragon/PCL2)
* 解压到D:/Data/Game中
* 打开

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

## Java Map 转 Bedrock

* [chunker](https://chunker.app/)

## Other

* 使用structure block导出3d模型
  * 注意：Java版不存在, Bedrock版才有，且市场模板会禁用Export功能
  * 按T，输入命令行：/give @a minecraft-structure
* 使用Mineways导出3d模型
  * [Mineways](https://www.realtimerendering.com/erich/minecraft/public/mineways/index.html) 将minecraft map导出为.obj(bedrock版使用minecraft的structure block, Java版通过Amulet转化Bedrock版)
* 使用[je2be](https://github.com/kbinani/je2be-core)将java版本地图转化为bedrock版本地图

## 常用指令

* 打开指令框：T
* 飞行指令：创造模式双击空格
* 加载schem：
  * 将filename.schem放到.minecraft/versions/版本/config/worldedit/schematics中
  * //schem load [文件名如JP-Shibuya] [文件格式如schem]
  * //paste
* 修改模式：
  * /gamemode (adventure / creative / spectator / survival)

## 网站

[modrinth](https://modrinth.com/) 含有一些模组,建筑不错。

[resourcepack](https://resourcepack.net/) 资源不错

[planetminecraft](https://www.planetminecraft.com/texture-pack/tim10erys-realistic-128x/) 内容极多

[minecraft-schematics](https://www.minecraft-schematics.com/schematic/235/download/) 我的世界原理图

[MсBuild](https://mcbuild.org/) 原理图和原理图编辑工具

## Minecraft Object/Schematic 外部工具

* [mineways](https://www.realtimerendering.com/erich/minecraft/public/mineways/) minecraft导出obj
* [j-mc-2-obj](https://github.com/jmc2obj/j-mc-2-obj) minecraft地图转obj
* [Amulet-Map-Editor](https://github.com/Amulet-Team/Amulet-Map-Editor) minecraft地图编辑器，只能打开世界存档，无法打开litmatics和schematic
* [voxedit](https://github.com/vengi-voxel/vengi) 导入很多格式文件转化为vox
* ~~[voxconvert](https://github.com/vengi-voxel/vengi) 将obj转化为vox~~

---

* 将新版的.schem文件转化为旧版的.schematic文件

---

* [cubical.xyz](https://cubical.xyz/) schematic在线编辑、创建、查看器，但似乎只能加载schematic
* [Dovetail](https://github.com/Offroaders123/Dovetail) Litematics和NBT数据查看，无法可视化，无法查看schematic
* NBT Explorer 类似Dovetail的数据编辑器，可以查看schematic
* NBT Studio 同NBT Explorer类似的数据编辑器
* [FileToVox](https://github.com/Zarbuz/FileToVox) 将文件转化为vox
* [Jmc2obj](https://github.com/jmc2obj/j-mc-2-obj)同mineways但不太好用
* [MagicaVoxel Viewer ](https://ephtracy.github.io/index.html?page=mv_renderer) 交互式路径跟踪渲染器，并没有导出vox功能
* [付费软件] [avoyd ](https://www.avoyd.com/home.html) 导入minecraft导出obj或vox
* [很久不维护][mcedit2](https://github.com/mcedit/mcedit2) minecraft编辑工具
* [Lite2Edit](https://github.com/GoldenDelicios/Lite2Edit) Litematics转化为schematic的工具
* [NBTExplorer](https://github.com/jaquadro/NBTExplorer) level.dat、schematic、NBT编辑工具
* [NBTEditor](https://github.com/Howaner/NBTEditor) 编辑 .dat 和 .schematic 文件
* [ObjToSchematic](https://github.com/LucasDower/ObjToSchematic) 3D .obj 模型转换为 Minecraft 结构
* [ambient-geometry](https://github.com/HunterTTP/ambient-geometry) 3D 立方体构建
* [schem-app](https://github.com/lukassteinwender/schem-app) 将新版本1.13+的.schem 文件转换为旧的 .schematic 文件

## Minecraft Unity

* [MarkovCraft](https://github.com/DevBobcorn/MarkovCraft) 生成建筑
* **[Theircraft](https://github.com/wetstreet/Theircraft)** Minecraft Unity重建
* **[CornCraft](https://github.com/DevBobcorn/CornCraft)** Minecraft Unity重建

## 资源包

* [Brixel](https://resourcepack.net/brixel-resource-pack/#gsc.tab=0) 问题是64x64的太粗糙，高清的要5美刀。 ![图片](https://resourcepack.net/fl/images/2022/06/Brixel-Resource-Pack-for-Minecraft-textures-2-1-840x473.jpg)
* [minebricks](https://resourcepack.net/minebricks-resource-pack/#gsc.tab=0) 乐高形状 ![图片](https://resourcepack.net/fl/images/2021/04/MineBricks-Resource-Pack-for-minecraft-textures-lego-3-840x473.jpg)
* [coven](https://resourcepack.net/coven-resource-pack/#gsc.tab=0) 精美的 ![图片](https://resourcepack.net/fl/images/2023/08/COVEN-Resource-Pack1-2-840x449.jpg)
* [sketch-hand-drawn](https://resourcepack.net/sketch-hand-drawn-resource-pack/#gsc.tab=0) 手绘草图 ![图片](https://resourcepack.net/fl/images/2014/01/Sketch-Hand-Drawn-Resource-Pack-for-minecraft-1.jpg)
* [optimum-realism](https://resourcepack.net/optimum-realism-resource-pack/#gsc.tab=0) 写实的 ![图片](https://resourcepack.net/fl/images/2022/01/Optimum-Realism-Resource-Pack-for-minecraft-new-r17-6-840x473.jpg)
* [meteor](https://resourcepack.net/meteor-lmx-resource-pack/#gsc.tab=0) 高光 ![图片](https://resourcepack.net/fl/images/2020/10/Meteor-LMX-Resource-Pack-for-minecraft-textures-9-840x464.jpg)
