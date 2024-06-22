---
layout:     post
title:      "Unity性能优化"
subtitle:   " \"Unity性能优化\""
date:       2024-06-21 11:15:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
---
20240621-unity-performance1.

所有测试在OnePlus 6T手机上进行。

1. 原始Profile(禁用Vsync哦)
   ![profile](/img/2023/20240621-unity-performance1.png)
2. 静态合批
   增加SRP合批

   * 前往 Edit > Project Settings > Player - Other Settings
   * 找到Static Batching组，启用。
     去掉MaterialPropertyBlock(因为使用MaterialPropertyBlock就不能合批)，不使用render.materials而是使用sharedMaterials。
     ![profile](/img/2023/20240621-unity-performance2.png)
3. 禁用加速计
   这个功能定义Unity从设备读取加速度仪信息的频率，在不需要加速仪的游戏中，将它启动或设置了高于需求的频率，会影响性能表现.
   Project Settings->Player->IOS->Other Settings - Accelerometer Frequency  (android没有这个设置)
4. GPU Instancing
   所有的material勾选GPU Instancing
   ![profile](/img/2023/20240621-unity-performance4.png)
5. 所有导入的模型启用纹理压缩
   xx.obj - Model - Mesh Compression 选择Medium
   ![profile](/img/2023/20240621-unity-performance5.png)

   |             | 1初始 | 2Static Batching | 4GPU Instance | 5Mesh Compression |  |
   | ----------- | ----- | ---------------- | ------------- | ----------------- | - |
   | CPU时间(ms) | 35.65 | 33.06            | 33.71         | 33.52             |  |
   | GPU时间(ms) | 29.55 | 20.98            | 23.59         | 22.82             |  |

似乎，并没有太大的作用？