---
layout:     post
title:      "Unity Shader Bible翻译学习"
subtitle:   " \"Unity Shader Bible翻译学习\""
date:       2024-01-25 01:30:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 持续更新
---
## **第一章 | 着色器编程语言介绍**

### **入门知识**

#### 1.0.1 | 多边形物体的属性

<details>
  <summary>原文</summary>
Years ago, when I was just starting my studies about shaders in Unity, it was challenging to understand much of the content I found in the books for several factors. I still remember that day of studies, wishing to understand the operation of the semantics POSITION[n]; however, when I managed to find its definition, I found the following statement:

`Vertex position in object-space.
At that moment, I asked myself, what is the vertex position in object-space? Then I understood that there was previous information that I had to know before starting to read about this subject.`

`In my experience, I have been able to identify at least four fundamental areas that facilitate the understanding of shaders and their structure, such as properties of a polygonal object, the structure of a render pipeline, matrices, and coordinate systems.`

`1.0.1 | Properties of a polygonal object.
The word polygon comes from Greek and is composed of poly (many) and gnow (angles). By definition, a polygon refers to a closed plane figure bounded by line segments.`

`Fig. 1.0.1a
A primitive is a three-dimensional geometric object formed by polygons and is used as a predefined object in different development software. Within Unity, Maya or Blender, we can find other primitives. The most common are: Spheres, Boxes, Quads, Cylinders and Capsules. These objects are different in shape but have similar properties; all have vertices, tangents, normals, UV coordinates and color, which are stored within a data type called “mesh”.`

`We can access all these properties independently within a shader and keep them in vectors (e.g. float4 pos: POSITION [n]). It is beneficial because we can modify their values and thus generate exciting effects. To understand this concept much better, we will give a small definition of the properties of a polygonal object.
</details>

#### `1.0.2 | 顶点`

`1.0.3 | 法线
1.0.4 | 切线
1.0.5 | UV坐标
1.0.6 | 顶点颜色
1.0.7 | 渲染管线架构
1.0.8 | 应用阶段
1.0.9 | 几何处理阶段
1.1.0 | 光栅化阶段
1.1.1 | 像素处理阶段
1.1.2 | 渲染管线类型
1.1.3 | 前向渲染
1.1.4 | 延迟渲染
1.1.5 | 我该用哪种渲染管线？
1.1.6 | 矩阵与坐标系统`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`

`strong`

`strong`

* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`
* `a`
* `a`
* `a`
* `9.0.7 | 用户自定义函数`

`strong`

`strong`

* `a`
* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
* `a`

`strong`

* `a`
* `a`
