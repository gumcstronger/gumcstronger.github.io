---
layout:     post
title:      "sketchfab下载的模型在Qubicle中使用"
subtitle:   " \"sketchfab、Blender、Qubicle\""
date:       2022-09-17 01:00:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 美术
---

- 下载模型
    - [Sketchfab](https://sketchfab.com/)下载3D模型(.gltf格式，包含scene.gltf、scene.bin、textures文件夹)
- 导入模型
    - Blender软件 - 文件/导入/glTF 2.0(.glb/gltf)，选中模型文件scene.gltf
    - 检查模型效果是否完整
        - Blender软件 - 点Shading看效果
- 导出.obj格式
    - Blender软件 - 文件/导出/Wavefront(.obj)(legacy) - 选择scene.gltf相同目录作为导出路径
    - 导出文件为scene.obj和scene.mtl
- 导入Qubicle
    - 修改scene.mtl中map_Kd、map_d的路径
        - 用vscode打开scene.mtl,将其中的"c:\\****\\textures\\xxx.png"修改为"\\textures\\xxx.png"（此处贴图做法是指向缓存文件的绝对路径，改为指向最早下载模型的贴图的相对路径）
    - Qubicle软件导入obj文件 
        -  Voxelize/Create Voxelizer... - 弹窗点Ok - Voxelize/Load Mesh - 选中scene.obj
    - 调整模型至最佳
        -  点击右侧的Mesh Scale - 修改缩放比例直到Voxel更加贴近原来的3D模型
        -  注意Size不要超过128