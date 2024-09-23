---
layout:     post
title:      "新项目Voxel处理教程"
subtitle:   " \"voxel project\""
date:       2024-06-25 16:15:00
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
1. 将obj转化为Voxel

   - 需要将obj进行面数、顶点数简化：[教程](https://www.youtube.com/watch?v=Erstqc5uSxU)
   - 有限考虑使用voxTool将obj转化为voxel。(其次才使用Qubicle将obj转化为voxel)
2. 将voxel处理为多个

   - 使用voxTool - Optimization将voxel自动拆分。
   - 复制拆分后的vox文件，保留需要的某个模块，删除不需要的，然后使用voxTool - Connectivity连接在一起
