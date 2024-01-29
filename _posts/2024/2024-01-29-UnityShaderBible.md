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
## 第一章 | 着色器编程语言介绍

### 入门知识

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

#### 1.1.6 | 矩阵与坐标系统

### Unity中的Shader

#### 2.0.1 | 什么是着色器(shader)？

#### 2.0.2 | 编程语言介绍

#### 2.0.3 | 着色器的种类

#### 2.0.4 | 标准表面着色器

#### 2.0.5 | 无光照着色器

#### 2.0.6 | 屏幕特效着色器

#### 2.0.7 | 计算着色器

#### 2.0.8 | 光线追踪着色器

### 属性、命令与函数

#### 3.0.1 | 顶点/片元着色器的结构

#### 3.0.2 | ShaderLab着色器

#### 3.0.3 | ShaderLab的属性

#### 3.0.4 | 数字与滑条类型

#### 3.0.5 | 颜色与向量类型

#### 3.0.6 | 纹理类型

#### 3.0.7 | 自定义材质属性绘制器

#### 3.0.8 | MPD开关

#### 3.0.9 | MPD关键词枚举

#### 3.1.0 | MPD枚举

#### 3.1.1 | MPD指数滑条与整数范围

#### 3.1.2 | MPD空白与标题

#### 3.1.3 | ShaderLab子着色器

#### 3.1.4 | ShaderLab标签

#### 3.1.5 | 队列标签

#### 3.1.6 | 渲染类型标签

#### 3.1.7 | SubShader混合

#### 3.1.8 | SubShader透明度遮罩

#### 3.1.9 | SubShader颜色遮罩

#### 3.2.0 | SubShader剔除与深度测试

#### 3.2.1 | ShaderLab剔除

#### 3.2.2 | ShaderLab深度写入

<detail><summary>原文</summary>

This command controls the writing of the surface pixels of an object to the Z-Buffer, that is, it allows us to ignore or respect the depth distance between the camera and an object. ZWrite has two values, which are: On and Off, where "On" corresponds to its default value. We generally use this command when working with transparencies, e.g., when we activate the Blending options.

* **ZWrite Off** For transparency.
* **ZWrite On** Default value.

Why should we disable the Z-Buffer when working with transparencies? Mainly because of the translucent pixel overlay (Z-fighting). When we work with semi-transparent objects, it is common that the GPU does not know which object lies in front of another, producing an overlapping effect between pixels when we move the camera in the scene. To fix this problem, we must simply deactivate the Z-Buffer by turning the **ZWrite** command to " **Off** " as the following example shows:

```text
Shader "InspectorPath/shaderName" 
{
    Properties { … } 
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        Blend SrcAlpha OneMinusSrcAlpha 
        ZWrite Off 
    } 
}
```

The Z-fighting occurs when we have two or more objects at the same distance from the camera, causing identical values in the Z-Buffer.

![](https://pic3.zhimg.com/80/v2-66d8a9d1bed11ea11c8d6ea3f4d3e27a_720w.webp)

Fig. 3.2.2a

This effect occurs when trying to render a pixel at the end of the rendering pipeline. Since the Z-Buffer cannot determine which element is behind the other, it produces flickering lines that change shape depending on the camera's position.

To correct this issue, we simply need to disable the Z-Buffer using the "ZWrite off" command.
</detail>


#### 3.2.3 | ShaderLab深度测试

<detail><summary>原文</summary>
ZTest controls how Depth Testing should be performed and is generally used in multi-pass shaders to generate differences in colors and depths. This property has seven different values, which are:

* **Less.**
* **Greater.**
* **LEqual.**
* **GEqual.**
* **Equal.**
* **NotEqual.**
* **Always.**

Which they correspond to a comparison operation.

 **ZTest Less** : (<) Draws the objects in front. It ignores objects that are at the same distance or behind the shader object.

 **ZTest Greater** : (>) Draws the objects behind. It does not draw objects that are at the same distance or in front of the shader object.

**ZTest LEqual:** (≤) Default value. Draws the objects that are in front of or at the same distance. It does not draw objects behind the shader object.

 **ZTest GEqual** : (≥) Draws the objects behind or at the same distance. Does not draw objects in front of the shader object.

 **ZTest Equal** : (==) Draws objects that are at the same distance. Does not draw objects in front of or behind the shader object.

 **ZTest NotEqual** : (! =) Draws objects that are not at the same distance. Does not draw objects that are the same distance from the shader object.

 **ZTest Always** : Draws all pixels, regardless of the distance of the objects relative to the camera.

To understand this command, we will do the following exercise: Let's suppose we have two objects in our scene; a Cube and a Sphere. The Sphere is in front of the Cube relative to the camera, and the pixel depth is as expected.

![](https://pic2.zhimg.com/80/v2-5d1e61a5bead5a789e43ab0455d8e6cd_720w.webp)

Fig. 3.2.3a

If we position the Sphere behind the Cube then again, the depth values will be as expected, why? Because the **Z-Buffer** is storing depth values for each pixel on the screen. The depth values are calculated concerning the proximity of an object to the camera.

![](https://pic1.zhimg.com/80/v2-e66caf7c13912e77a77604c8aef3f9d0_720w.webp)

Fig. 3.2.3b

Now, what would happen if we activated ZTest Always? In this case, Depth Testing would not be done, therefore, all pixels would appear at the same depth on screen.

![](https://pic4.zhimg.com/80/v2-e647e9872767f75cf77b2653e329babf_720w.webp)

Fig. 3.2.3c

Its syntax is the following:

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties { … } 
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        ZTest LEqual 
    } 
}
```
</detail>

深度测试（ZTest）通常用于在有多 pass 的着色器中生成颜色和深度差异。该属性有七个不同的值，分别是：

* **Less.**
* **Greater.**
* **LEqual.**
* **GEqual.**
* **Equal.**
* **NotEqual.**
* **Always.**

它们与比较操作相对应。

 **ZTest Less** : (<) 绘制位于现有几何体前面的几何体。不绘制位于现有几何体相同距离或后面的几何体。

 **ZTest Greater** : (>) 绘制位于现有几何体后面的几何体。不绘制位于现有几何体相同距离或前面的几何体。

**ZTest LEqual:** (≤) 默认值。绘制位于现有几何体前面或相同距离的几何体。不绘制位于现有几何体后面的几何体。

 **ZTest GEqual** : (≥) 绘制位于现有几何体后面或相同距离的几何体。不绘制位于现有几何体前面的几何体。

 **ZTest Equal** : (==) 绘制位于现有几何体相同距离的几何体。不绘制位于现有几何体前面的或后面的几何体。

 **ZTest NotEqual** : (! =) 绘制不位于现有几何体相同距离的几何体。不绘制位于现有几何体相同距离的几何体。

 **ZTest Always** : 不进行深度测试。绘制所有几何体，无论距离如何。

为了理解深度测试，我们将做以下练习： 假设场景中有两个物体（一个立方体和一个球体），相对于摄像机而言球体位于立方体的前方，像素深度符合预期。

[https://picx.zhimg.com/80/v2-931e73ae4283caf9412196eaf837b81f.png](https://link.zhihu.com/?target=https%3A//picx.zhimg.com/80/v2-931e73ae4283caf9412196eaf837b81f_720w.png)

![](https://pic2.zhimg.com/80/v2-5d1e61a5bead5a789e43ab0455d8e6cd_720w.webp)

Fig. 3.2.3a

如果我们将球体放置在立方体后面，深度值也和我们预期的一样，为什么呢？因为** Z 缓冲**存储的是屏幕上每个像素的深度值，深度值是根据物体与相机之间的深度距离计算得出的。

[https://picx.zhimg.com/80/v2-6fd2325002b2b6e0ee9147786b789b31.png](https://link.zhihu.com/?target=https%3A//picx.zhimg.com/80/v2-6fd2325002b2b6e0ee9147786b789b31_720w.png)

![](https://pic1.zhimg.com/80/v2-e66caf7c13912e77a77604c8aef3f9d0_720w.webp)

Fig. 3.2.3b

现在，如果我们将深度测试设置成“总是（Always）”会发生什么呢？在这种情况下，深度测试并不会发生，所有像素在屏幕上显示的深度是相同的。

[https://picx.zhimg.com/80/v2-18f4d71f1a5a95de290e65bcf96155c5.png](https://link.zhihu.com/?target=https%3A//picx.zhimg.com/80/v2-18f4d71f1a5a95de290e65bcf96155c5_720w.png)

![](https://pic4.zhimg.com/80/v2-e647e9872767f75cf77b2653e329babf_720w.webp)

Fig. 3.2.3c

深度测试的语法如下所示：

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties { … } 
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        ZTest LEqual 
    } 
}
```

#### 3.2.4 | ShaderLab模板

#### 3.2.5 | ShaderLab Pass

#### 3.2.6 | CGPROGRAM/ENDCG

#### 3.2.7 | 数据类型

#### 3.2.8 | Cg/HLSL编程

#### 3.2.9 | Cg/HLSL Include

#### 3.3.0 | Cg/HLSL顶点输入&输出

#### 3.3.1 | Cg/HLSL变量与连接向量

#### 3.3.2 | Cg/HLSL顶点着色器

#### 3.3.3 | Cg/HLSL片元着色器

#### 3.3.4 | ShaderLab回退

### 其他概念与实现

#### 4.0.1 | 着色器和材质的关系，好有一比啊~

#### 4.0.2 | 我们的第一个Cg/HLSL着色器

#### 4.0.3 | 为Cg/HLSL着色器加上透明度

#### 4.0.4 | HLSL函数的结构

#### 4.0.5 | Shader Debug

#### 4.0.6 | 添加URP兼容

#### 4.0.7 | 内置函数

#### 4.0.8 | Abs函数

#### 4.0.9 | Ceil函数

#### 4.1.0 | Clamp函数

#### 4.1.1 | Sin与Cos函数

#### 4.1.2 | Tan函数

#### 4.1.3 | Exp、Exp2与Pow函数

#### 4.1.4 | Floor函数

#### 4.1.5 | Step与SmoothStep函数

#### 4.1.6 | Length函数

#### 4.1.7 | Frac函数

#### 4.1.8 | Lerp函数

#### 4.1.9 | Min与Max函数

#### 4.2.0 | 时间与动画

## 第二章 | 光照、阴影与表面

### 章节介绍

#### 5.0.1 | 配置输入与输出

#### 5.0.2 | 向量

#### 5.0.3 | 点乘

#### 5.0.4 | 叉乘

### 表面

#### 6.0.1 | 法线贴图

#### 6.0.2 | DXT压缩

#### 6.0.3 | TBN矩阵

### 光照

#### 7.0.1 | 光照模型

#### 7.0.2 | 环境光颜色

#### 7.0.3 | 漫反射

#### 7.0.4 | 镜面反射

#### 7.0.5 | 环境反射

#### 7.0.6 | 菲涅尔效应

#### 7.0.7 | 标准表面着色器的结构

#### 7.0.8 | 标准表面着色器的输入与输出

### 阴影

#### 8.0.1 | 阴影映射

#### 8.0.2 | 阴影投射

#### 8.0.3 | 阴影贴图

#### 8.0.4 | 阴影实现

#### 8.0.5 | 内置渲染管线下的阴影贴图优化

#### 8.0.6 | 通用渲染管线下的阴影映射

### Shader Graph

#### 9.0.1 | 什么是Shader Graph

#### 9.0.2 | 准备Shader Graph环境

#### 9.0.3 | 界面分析

#### 9.0.4 | 我们的第一个Shader Graph

#### 9.0.5 | Graph检查器

#### 9.0.6 | 节点

#### 9.0.7 | 用户自定义函数

## 第三章 | 计算着色器、光线追踪与球体追踪

### 高级概念

#### 10.0.1 | 计算着色器的结构

#### 10.0.2 | 我们的第一个计算着色器

#### 10.0.3 | UV坐标与贴图

#### 10.0.4 | 缓冲

### 球体追踪

#### 11.0.1 | 用球体追踪实现函数

#### 11.0.2 | 纹理投影

#### 11.0.3 | 曲面间平滑

### 光线追踪

#### 12.0.1 | 在HDRP中配置光线追踪

#### 12.0.2 | 在场景中使用光线追踪
