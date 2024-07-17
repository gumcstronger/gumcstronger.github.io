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
## Obj2Vox

- 下载模型

  - [Sketchfab](https://sketchfab.com/)下载3D模型(.gltf格式，包含scene.gltf、scene.bin、textures文件夹)（如果不能下载，可使用[sketchfab-ripper](https://github.com/jiutian1137/sketchfab-ripper/tree/main)下载，并转化为voxel后手动填色）
    - Blender软件看效果。如果有贴图但没有颜色，则点击右上角 - Viewport Shading - Texture(代表Color from texture而不是material)，则能够正常显示模型贴图颜色
  - [Unity商店](https://assetstore.unity.com/)下载3D模型，如是voxel的则可以直接使用。需要再处理则可以通过qubicle转化为obj和全贴图（不能用magicavoxel，因为magicavoxel转化后的贴图只是color palette)
- 处理.obj格式探索

  - 问题:下载obj，该obj由magicavoxel生成，含有xx.obj，xx.mtl，xx.png，但打开并没有颜色信息。
    - 可能是贴图名字被修改：
      - 打开xx.mtl,将map_Kd一行修改为map_Kd xx.png
    - 贴图是palette色板，应用于材质，并不是basecolor或BPR贴图。
      - ❌(SimpleBake导出后黑色)使用[SimpleBake](https://www.gfxcamp.com/blender-simplebake/) 将其渲染为PBR贴图
      - ❌(导出后依旧是palette信息)方法1:使用ObjToSchematic重新转化为obj
      - 方法2:使用teardown团队开发的[voxtool](https://teardowngame.com/voxtool/)（需要修改颜色)
        - 打开后选择model
        - Input Mesh / Input file:
          - 选择obj文件，
          - Swap YZ: true
        - Output vox / vox file: 选择导出voxel地址
        - Materials / platte:
          - Texture选择obj附带的palette图片
          - Material: 随便选一个,可以选择Unphysical
          - Num colors Rows设置为1
        - 如果有多文件可以尝试使用多文件
  - 问题：gltf且只包含一张贴图
    - Blender软件 - 文件/导入/glTF 2.0(.glb/gltf)，选中模型文件scene.gltf
    - Blender软件 - 文件/导出/Wavefront(.obj)(legacy) - 选择scene.gltf相同目录作为导出路径
    - 导出文件为scene.obj和scene.mtl(注意mtl中的贴图依旧需要指向gltf中的贴图)
  - 问题：包含多张贴图
    - Qubicle软件 - Voxelize / Create Voxelizer - 然后Voxelize / Load Mesh
    - 通过File -Export - Wavefront OBJ 导出obj格式
    - 导出后obj和mtl指向的material文件夹下只有一张贴图
  - 问题：不包含贴图
    - 下载fbx，将fbx导入Unity，点击Extract Materials，生成模型。
    - 然后使用[Magicavoxel Tools Free](https://assetstore.unity.com/packages/tools/utilities/magicavoxel-tools-free-146116)将mesh转化成voxel
- 体素化

  - 情况一：如果本来是体素模型，只是Obj格式，且图片是palette但不是纹理贴图，使用Voxtool进行处理。
  - 情况二：如模型本来不一定是体素模型，是obj格式，但没有贴图。则通过Qubicle处理即可。（有时候Qubicle效果更好，有时候在线体素化器效果更好更好）
    - Qubicle处理
      Voxelize/Create Voxelizer... - 弹窗点Ok - Voxelize/Load Mesh - 选中scene.obj
      - 调整模型至最佳 - 点击右侧的Mesh Scale - 修改缩放比例直到Voxel更加贴近原来的3D模型(注意Size不要超过128)
      - 导出voxel格式文件
    - Qubicle常见问题
      - 如遇到问题，检查obj、mtl、png的文件名和路径是否含有空格
      - 如遇到问题，可以尝试修改scene.mtl中map_Kd、map_d的路径，用vscode打开scene.mtl,将其中的"c:\\****\\textures\\xxx.png"修改为"\\textures\\xxx.png"（此处贴图做法是指向缓存文件的绝对路径，改为指向最早下载模型的贴图的相对路径）
  - 情况三：模型不是体素模型，而是low poly、卡通甚至写实，则使用[在线体素化器](http://voxelizer.coohex.com/)或[在线体素化器](https://drububu.com/miscellaneous/voxelizer/?out=obj)将obj转化成voxel，然后通过magicavoxel进行调整
  - 再不行使用voxelizer
- [从图片中提取有限调色板](https://sketchbooky.wordpress.com/2020/09/23/some-tools-for-extracting-a-limited-colour-palette-from-a-picture/)

## Unity美术效果

- Unity使用Voxel Toolkit导入vox时，Mesh Generation Approach需要选择Textured才会颜色正常
- 30*30*30 => 3 * 3 * 3

## 降低顶点数和面熟

- [教程](https://www.youtube.com/watch?v=Erstqc5uSxU)

## Vox的游戏

* [Blocky Farm](https://play.google.com/store/apps/details?id=com.JetToast.BlockyFarm&hl=en_US)
* [Twisty Board 2](https://apps.apple.com/us/app/twisty-board-2/id1245082161)
* [Kingdoms of HF - Dragon War](https://play.google.com/store/apps/details?id=ata.kraken.heckfire&gl=US)
* [Countless Zombies](https://play.google.com/store/apps/details?id=jp.co.studio08.zmb&gl=JP)
* [Westy West Cowboys](https://play.google.com/store/apps/details?id=com.CountrysideGames.WestyWest&gl=AE)
* [Donut Factory Tycoon](https://play.google.com/store/apps/details?id=com.mindstormstudios.tinydonuts.google&gl=US)
* [Pizza Factory Tycoon](https://apps.apple.com/us/app/pizza-factory-tycoon/id1425902375)
* [Formula Clicker Idle Tycoon](https://play.google.com/store/apps/details?id=com.ggds.FormulaClicker&gl=US)
* [Dinos Royale](https://apps.apple.com/us/app/dinos-royale/id1403969940)
* Pixel Grand Battle 3D
