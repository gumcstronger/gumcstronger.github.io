---
layout:     post
title:      "sketchfab下载的模型在Qubicle中使用"
subtitle:   " \"sketchfab、Blender、Qubicle\""
date:       2022-09-22 01:00:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
---
- 下载模型

  - [Sketchfab](https://sketchfab.com/)下载3D模型(.gltf格式，包含scene.gltf、scene.bin、textures文件夹)
- 导入模型

  - Blender软件 - 文件/导入/glTF 2.0(.glb/gltf)，选中模型文件scene.gltf
- 检查模型效果是否完整

  - Blender软件看效果。如果有贴图但没有颜色，则点击右上角 - Viewport Shading - Texture(代表Color from texture而不是material)https://sketchfab.com/3d-models/voxel-nooks-cranny-0392eaa461d94a0393621ffc6bd56cb4
- 导出.obj格式

  - Blender软件 - 文件/导出/Wavefront(.obj)(legacy) - 选择scene.gltf相同目录作为导出路径
  - 导出文件为scene.obj和scene.mtl
- 导入前问题处理

  - 如遇到问题，检查obj、mtl、png的文件名和路径是否含有空格
  - 如遇到问题，可以尝试修改scene.mtl中map_Kd、map_d的路径，用vscode打开scene.mtl,将其中的"c:\\****\\textures\\xxx.png"修改为"\\textures\\xxx.png"（此处贴图做法是指向缓存文件的绝对路径，改为指向最早下载模型的贴图的相对路径）
- 导入Qubicle

  - Voxelize/Create Voxelizer... - 弹窗点Ok - Voxelize/Load Mesh - 选中scene.obj
  - 调整模型至最佳 - 点击右侧的Mesh Scale - 修改缩放比例直到Voxel更加贴近原来的3D模型(注意Size不要超过128)
- 保存Qubicle文件(.qbcl)

  - Ctrl + S
- 导出qb格式文件

  - File - Save - Qubicle Binary QB...

### 规格要求

- Qubicle的size不能超过128*128*128

### 问题

- Sketchfab下载gltf后没有texture贴图文件，使用blender转成obj后qubicle导入没有颜色。
  - 因为该gltf文件使用了程序化贴图，并没有烘培出贴图使用。
  - 使用Simplebake烘培出贴图，并在纹理属性中选择烘培出来的图片
- Blender导入模型后编辑模式不见了
  - 无法解决
