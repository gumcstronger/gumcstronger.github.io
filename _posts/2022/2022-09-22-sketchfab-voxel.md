---
layout:     post
title:      "sketchfab下载的模型如何处理成体素"
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
  - Blender软件看效果。如果有贴图但没有颜色，则点击右上角 - Viewport Shading - Texture(代表Color from texture而不是material)，则能够正常显示模型贴图颜色
- 处理.obj格式
  - 情况一：只包含一张贴图
    - Blender软件 - 文件/导入/glTF 2.0(.glb/gltf)，选中模型文件scene.gltf
    - Blender软件 - 文件/导出/Wavefront(.obj)(legacy) - 选择scene.gltf相同目录作为导出路径
    - 导出文件为scene.obj和scene.mtl(注意mtl中的贴图依旧需要指向gltf中的贴图)
  - 情况二：包含多张贴图
    - Qubicle软件 - Voxelize / Create Voxelizer - 然后Voxelize / Load Mesh
    - 通过File -Export - Wavefront OBJ 导出obj格式
    - 导出后obj和mtl指向的material文件夹下只有一张贴图
  - 情况三：不包含贴图
    - 下载fbx，将fbx导入Unity，点击Extract Materials，生成模型。
    - 然后使用[Magicavoxel Tools Free](https://assetstore.unity.com/packages/tools/utilities/magicavoxel-tools-free-146116)将mesh转化成voxel
- 体素化
  - 情况一：如模型本来是体素模型，只是obj格式。则通过Qubicle处理即可。

    - Qubicle处理
      Voxelize/Create Voxelizer... - 弹窗点Ok - Voxelize/Load Mesh - 选中scene.obj
      - 调整模型至最佳 - 点击右侧的Mesh Scale - 修改缩放比例直到Voxel更加贴近原来的3D模型(注意Size不要超过128)
      - 导出voxel格式文件
    - Qubicle常见问题
      - 如遇到问题，检查obj、mtl、png的文件名和路径是否含有空格
      - 如遇到问题，可以尝试修改scene.mtl中map_Kd、map_d的路径，用vscode打开scene.mtl,将其中的"c:\\****\\textures\\xxx.png"修改为"\\textures\\xxx.png"（此处贴图做法是指向缓存文件的绝对路径，改为指向最早下载模型的贴图的相对路径）
  - 情况二：模型不是体素模型，而是low poly、卡通甚至写实，则使用[在线体素化器](http://voxelizer.coohex.com/)或[在线体素化器](https://drububu.com/miscellaneous/voxelizer/?out=obj)将obj转化成voxel，然后通过magicavoxel进行调整
