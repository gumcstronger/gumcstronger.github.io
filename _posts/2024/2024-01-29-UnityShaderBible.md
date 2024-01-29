---
layout:     post
title:      "[转]Unity Shader Bible翻译学习"
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

Vertex position in object-space.
At that moment, I asked myself, what is the vertex position in object-space? Then I understood that there was previous information that I had to know before starting to read about this subject.

In my experience, I have been able to identify at least four fundamental areas that facilitate the understanding of shaders and their structure, such as properties of a polygonal object, the structure of a render pipeline, matrices, and coordinate systems.

1.0.1 | Properties of a polygonal object.
The word polygon comes from Greek and is composed of poly (many) and gnow (angles). By definition, a polygon refers to a closed plane figure bounded by line segments.

Fig. 1.0.1a
A primitive is a three-dimensional geometric object formed by polygons and is used as a predefined object in different development software. Within Unity, Maya or Blender, we can find other primitives. The most common are: Spheres, Boxes, Quads, Cylinders and Capsules. These objects are different in shape but have similar properties; all have vertices, tangents, normals, UV coordinates and color, which are stored within a data type called “mesh”.

We can access all these properties independently within a shader and keep them in vectors (e.g. float4 pos: POSITION [n]). It is beneficial because we can modify their values and thus generate exciting effects. To understand this concept much better, we will give a small definition of the properties of a polygonal object.

</details>

许多年前，当我刚开始学习 Unity 中的着色器时，有几个原因导致我难以理解书本上的内容。我仍然记得有一天，我希望理解语义 POSITION[n] 的操作，但关于它的介绍只有短短一句：

> 物体空间中的顶点位置。

什么是物体空间中的顶点位置？这时我意识到，在开始学习之旅前我必须先了解一些基础知识。

根据我的经验，至少有四个基本方面有助于理解着色器及其结构，如多边形物体的属性、渲染管线的结构、矩阵以及坐标系。

### **1.0.1 | 多边形物体的属性**

 **多边形（polygon）** 一词来自希腊语，由“多”（poly）和“角”（gnow）组成。根据定义，多边形是指平面的封闭几何图形，由大于2条线段组成，且首尾相连划出的形状。

![](https://pic4.zhimg.com/80/v2-1871dac9ec1db6960deab6999f774d0b_720w.webp)

Fig. 1.0.1a

 **基本体（primitive）** 是由多边形构成的三维几何对象，在诸如Unity、Maya、Blender等不同的开发软件中用作预定义对象。最常见的基本体有球体、正方体、四边形、圆柱体和胶囊体等。这些物体虽然形状各异，但属性相似：它们都有顶点、切线、法线、UV 坐标和颜色，这些属性被存储在一种名为 " **网格（mesh）** "的数据类型中。

我们可以在着色器中独立访问所有这些属性，并将它们保存在向量中（例如 float4 pos: POSITION [n]）。这样做的好处是我们可以修改它们的值，从而编写出令人兴奋的效果。为了更好地理解这一概念，我们将给出关于多边形物体属性的定义。

![](https://pic1.zhimg.com/80/v2-9a7743b06c38b57fd90ad19874744b04_720w.webp)

Fig. 1.0.1b

#### 1.0.2 | 顶点

#### 1.0.3 | 法线

#### 1.0.4 | 切线

#### 1.0.5 | UV坐标

#### 1.0.6 | 顶点颜色

#### 1.0.7 | 渲染管线架构

#### 1.0.8 | 应用阶段

#### 1.0.9 | 几何处理阶段

#### 1.1.0 | 光栅化阶段

#### 1.1.1 | 像素处理阶段

#### 1.1.2 | 渲染管线类型

#### 1.1.3 | 前向渲染

#### 1.1.4 | 延迟渲染

#### 1.1.5 | 我该用哪种渲染管线？

#### 1.1.6 | 矩阵与坐标系统`
