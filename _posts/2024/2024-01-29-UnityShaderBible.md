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

<details> 
<summary>原文</summary>

Blending is the process of mixing two pixels into one. Its command is compatible with both *Built-in RP* and  *Scriptable RP* .

*Blending* occurs in a stage called " **merging** " which combines the final color of a pixel (those pixels that have been processed in the fragment shader stage) with its depth. This stage, which occurs at the end of the render pipeline; after the  **fragment shader stage** , is where the stencil-buffer, z-buffer and color blending are executed.

By default, this property is not written in our shader since it is an optional function and is used mainly when we work with transparent objects, e.g., when we must draw a pixel with a low opacity level in front of another.

Its default value is "Blend Off", but we can activate it to generate different types of Blending, like those that appear in photoshop.

Its syntax is as follows:

```text
Blend [SourceFactor] [DestinationFactor]
```

" **Blend** " is a function that requires two values called "factors" for its operation and based on an equation it will be the final color that we will obtain on-screen. According to the official documentation in Unity, the equation that defines the value of the Blending is as follows:

> B = SrcFactor * SrcValue [OP] DstFactor * DstValue.

To understand this operation, we must consider the following: The **fragment shader stage** occurs first and then, as an optional process; the  **merging stage** .

**"SrcValue"** (source value), which has been processed in the  **fragment shader stage** , corresponds to the pixel’s RGB color output.

**"DstValue"** (destination value) corresponds to the RGB color that has been written in the "destination buffer", better known as "render target" (SV_Target). When the Blending options are not active in our shader, SrcValue overwrites DstValue. However, if we activate this operation, both colors are mixed to get a new color, which overwrites the previous DstValue.

**"SrcFactor"** (source factor) and "DstFactor" (destination factor) are vectors of three dimensions that vary depending on their configuration. Their main function is to modify the SrcValue and DstValue values to achieve interesting effects.

Some factors that we can find in the Unity documentation are:

* **Off** , disables Blending options.
* **One** , (1, 1, 1).
* **Zero** , (0, 0, 0).
* **SrcColor** is equal to the RGB values of the SrcValue.
* **SrcAlpha** is equal to the Alpha value of the SrcValue.
* **OneMinusSrcColor** , 1 minus the RGB values of the SrcValue (1 - R, 1 - G, 1 - B).
* **OneMinusSrcAlpha** , 1 minus the Alpha of SrcValue (1 - A, 1 - A, 1- A).
* **DstColor** is equal to the RGB values of the DstValue.
* **DstAlpha** is equal to the Alpha value of the DstValue.
* **OneMinusDstColor** , 1 minus the RGB values of the DstValue (1 - R, 1 - G, 1 - B).
* **OneMinusDstAlpha** , 1 minus the Alpha of the DstValue (1 - A, 1 - A, 1- A).

It is worth mentioning that the Blending of the Alpha channel is carried out in the same way in which we process the RGB color of a pixel, but it is done in an independent process because it is not used frequently. Likewise, by not performing this process, the writing on the render target is optimized.

Let's exemplify the above explanation as follows.

Let's say we have an RGB color pixel with the values [0.5R, 0.45G, 0.35B]. This color has been processed by the ** fragment shader stage** , therefore it corresponds to the  *DstValue* . Now, we multiply this value by the "*SrcFactor*  **One** " which equals [1, 1, 1]. Every number multiplied by "1" results in the same value, therefore, the result between the *SrcFactor* and the *DstValue* is the same as its initial value.

> B = **[0.5R, 0.45G, 0.35B]** [OP] DstFactor * DstValue.

"OP" refers to the operation that we are going to perform. By default, it is set to "Add".

> B = [0.5R, 0.45G, 0.35B] + DstFactor * DstValue.

Once we have obtained the value of the first operation, it is overwritten by DstValue, therefore, is set to the same color [0.5R, 0.45G, 0.35B]. So, we will multiply this color by the "*DstFactor*  **DstColor** ", which is equal to the current value we have in the  *DstFactor* .

> DstFactor [0.5R, 0.45G, 0.35B] * DstValue [0.5R, 0.45G, 0.35B] = [0.25R, 0.20G, 0.12B].

Finally, the output color for the pixel is.

> B = [0.5R, 0.45G, 0.35B] + [0.25R, 0.20G, 0.12B].
> B = [0.75R, 0.65G, 0.47B]

If we want to activate Blending in our shader, we must use the **Blend **command followed by **SrcFactor** and then  **DstFactor** .

Its syntax is the following:

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties { … } 
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        Blend SrcAlpha OneMinusSrcAlpha 
    } 
}
```

If we want to use Blending in our shader, it will be necessary to add and modify the * "Render Queue"* . As we already know, the default value of the *"Queue"* tag is  *"Geometry"* , which means that our object will appear opaque. If we want our object to look transparent, then we must first change the *"Queue"* to *"Transparent"* and then add some kind of blending.

The most common types of blending are the following:

* **Blend SrcAlpha OneMinusSrcAlpha** Common transparent blending
* **Blend One One** Additive blending color
* **Blend OneMinusDstColor One** Mild additive blending color
* **Blend DstColor Zero** Multiplicative blending color
* **Blend DstColor SrcColor** Multiplicative blending x2
* **Blend SrcColor One** Blending overlay
* **Blend OneMinusSrcColor One** Soft light blending
* **Blend Zero OneMinusSrcColor** Negative color blending

A different way to configure our Blending is through the dependency "UnityEngine.Rendering. BlendMode". This line of code allows us to change, from the inspector, the Blending of an object in the material. To set it up, first we must add the *"Toggle Enum"* to our properties and then declare both the SrcFactor and the DstFactor.

Its syntax is as follows:

```text
[Enum(UnityEngine.Rendering.BlendMode)] 
    _SrcBlend ("Source Factor", Float) = 1 
[Enum(UnityEngine.Rendering.BlendMode)] 
    _DstBlend ("Destination Factor", Float) = 1
```

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties 
    { 
        [Enum(UnityEngine.Rendering.BlendMode)] 
        _SrcBlend ("SrcFactor", Float) = 1 
        [Enum(UnityEngine.Rendering.BlendMode)] 
        _DstBlend ("DstFactor", Float) = 1 
    }
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        Blend [_SrcBlend] [_DstBlend] 
    } 
}
```

Blending options can be written in different fields: within the **SubShader** field or the **Pass** field, the position will depend on the number of passes and the final result that we need.

</details>

混合（Blending）是将两个像素处理成一个的过程，是内置渲染管线（Built-in）与可编程渲染管线（SRP）都兼容的一种命令。

*混合 *发生在“ **合并（merging）** ”的阶段，它将像素的最终颜色（在片元着色器阶段处理后的像素）与像素的深度结合在一起。这个阶段位于渲染管线的末端，在**片元着色器阶段**之后，是执行模版缓冲（stencil-buffer）、深度缓冲（z-buffer）和颜色混合（color blending）的地方。

混合属性不会被写在新建的默认着色器里，因为这是一个通常被使用在透明物体中的可选功能，比如说当我们必须在一个像素前绘制另一个不透明度较低的像素时。

混合的默认值是“Blend Off”（不开启混合），但我们可以通过设置它来实现不同类型的颜色混合效果，就像在 Photoshop 中的混合效果那样。

混合的语法如下所示：

```text
Blend [SourceFactor] [DestinationFactor]
```

“ **混合** ”是一个函数，需要两个输入，经过一个计算公式得到屏幕上的最终颜色。根据 Unity 的官方文档，定义 混合结果的值的等式如下：

> B = SrcFactor * SrcValue [OP] DstFactor * DstValue.

为了理解这个公式，我们需要记住这个顺序： 首先是 **片元着色器阶段** ，然后是可选的 **合并阶段** 。

**"SrcValue"** (源颜色）代表的是经过**片元着色器**处理的像素的RGB值。

**"DstValue"** (目标颜色) 代表的是已经写入“目标缓冲”的像素的RGB值，目标缓冲更常见的名字是渲染目标（SV_Target）。当我们没有在着色器中设置混合选项时，源颜色会直接覆盖目标值。如果我们设置了混合，源颜色与目标颜色就会混合产生一个新的颜色，再覆盖先前的目标颜色。

**"SrcFactor"** (源系数) and " **DstFactor** " (目标系数) 都是一个三维向量，它们主要的功能是修改源颜色和目标颜色来实现有趣的效果。

我们能在Unity官方文档（[ShaderLab 命令：Blend - Unity 手册](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/cn/2021.2/Manual/SL-Blend.html)）找到一些系数：

* **Off** , 禁用混合选项。
* **One** , (1, 1, 1)。
* **Zero** , (0, 0, 0)。
* **SrcColor** 等于源颜色的RGB值。
* **SrcAlpha** 等于源颜色的透明度。
* **OneMinusSrcColor** , 1 - 源颜色的RGB值 (1 - R, 1 - G, 1 - B)。
* **OneMinusSrcAlpha** , 1 - 源颜色的透明度 (1 - A, 1 - A, 1- A)。
* **DstColor** 等于目标颜色的RGB值。
* **DstAlpha** 等于目标颜色的透明度。
* **OneMinusDstColor** , 1 - 目标颜色的RGB值 (1 - R, 1 - G, 1 - B)。
* **OneMinusDstAlpha** , 1 - 目标颜色的透明度 (1 - A, 1 - A, 1- A)。

值得一提的是，透明度（Alpha）通道的混合与 RGB 颜色混合的方式相同，但由于不常用，所以是在一个独立的流程中完成的。如果不执行透明度混合，写入渲染目标上的过程也会得到优化。

让我们来理解理解上面的这段话。

假设我们现在有一个RGB值为 [0.5R, 0.45G, 0.35B] 的像素，已经经过**片元着色器**的处理，所以这个像素现在就是我们的源颜色。现在，我们把这个源颜色乘上系数“*SrcFactor * **One** ”（即 [1, 1, 1]），由于每个数字乘以 “1” 的结果都是它本身，因此，源系数和源颜色之间的结果就等于 [0.5R, 0.45G, 0.35B] 。

> B = **[0.5R, 0.45G, 0.35B]** [OP] DstFactor * DstValue.

“OP”指的是我们要执行的操作，一般默认设为加法（Add），如下所示：

> B = [0.5R, 0.45G, 0.35B] + DstFactor * DstValue.

到这一步之后，再来看看已经写入颜色缓冲中的目标颜色（假设为 [0.5R、0.45G、0.35B]）。让我们将目标系数设置成“*DstFactor*  **DstColor** ”，该系数等于当前目标颜色的 RGB 值，则有：

> DstFactor [0.5R, 0.45G, 0.35B] * DstValue [0.5R, 0.45G, 0.35B] = [0.25R, 0.20G, 0.12B].

所以最后输出的像素颜色是：

> B = [0.5R, 0.45G, 0.35B] + [0.25R, 0.20G, 0.12B].
> B = [0.75R, 0.65G, 0.47B]

如果我们想要在着色器中开启混合，我们就需要使用到**混合**指令、**源系数**和 **目标系数** ，语法如下所示：

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties { … } 
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        Blend SrcAlpha OneMinusSrcAlpha 
    } 
}
```

如果我们想在着色器里使用混合就必须添加和配置“ *渲染队列（Render Queue）* ”。我们已经学习过默认的“ *队列（Queue）* ”标签是“ *几何（Geometry）* ”，代表我们的物体是不透明的。如果我们想要让物体看起来有透明的感觉，就需要把“队列”标签设置为“ *透明（Transparent）* ”，再设置混合选项。

最常见的几种混合命令如下所示：

* **Blend SrcAlpha OneMinusSrcAlpha** 传统透明度混合
* **Blend One One** 加法
* **Blend OneMinusDstColor One** 软加法
* **Blend DstColor Zero** 乘法
* **Blend DstColor SrcColor** 2x 乘法
* **Blend SrcColor One** 叠加
* **Blend OneMinusSrcColor One** 柔光
* **Blend Zero OneMinusSrcColor** 反色

另一种配置混合的方式是通过“UnityEngine.Rendering. BlendMode”，它允许我们从检查器中更改材质的混合效果。但首先，我们必须在属性中添加 " *Toggle Enum* "，然后声明源系数和目标系数。

语法如下所示：

```text
[Enum(UnityEngine.Rendering.BlendMode)] 
    _SrcBlend ("Source Factor", Float) = 1 
[Enum(UnityEngine.Rendering.BlendMode)] 
    _DstBlend ("Destination Factor", Float) = 1
```

```text
Shader "InspectorPath/shaderName" 
{ 
    Properties 
    { 
        [Enum(UnityEngine.Rendering.BlendMode)] 
        _SrcBlend ("SrcFactor", Float) = 1 
        [Enum(UnityEngine.Rendering.BlendMode)] 
        _DstBlend ("DstFactor", Float) = 1 
    }
    SubShader 
    { 
        Tags { "Queue"="Transparent" "RenderType"="Transparent" } 
        Blend [_SrcBlend] [_DstBlend] 
    } 
}
```

混合选项可以写在**子着色器**或** Pass** 语义块中，具体写在哪里取决于 pass 的数量和我们最终想要取得的结果。

#### 3.1.8 | SubShader透明度遮罩

#### 3.1.9 | SubShader颜色遮罩

#### 3.2.0 | SubShader剔除与深度测试

#### 3.2.1 | ShaderLab剔除

#### 3.2.2 | ShaderLab深度写入

<details>
<summary>原文</summary>

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
</details>

深度写入这个命令控制了物体表面像素写入 Z 缓冲（深度缓冲）的这一过程。它允许我们忽略或写入物体与相机间的深度。深度写入有两个可以设置的值，分别是开启（On）和关闭（Off），默认值为开启。我们通常在处理透明度时（例如混合）会关闭深度写入。

* **ZWrite Off** 处理透明度时
* **ZWrite On** 默认值

为什么我们需要在处理透明度时禁用 Z 缓冲？主要是因为半透明像素叠加（深度冲突）的问题。当我们处理半透明物体时，GPU 通常不知道哪个物体位于另一个物体前面，因此当我们在场景中移动摄像机时，像素之间会产生奇怪的重叠效果。要解决这个问题，我们只需将**深度写入**命令设置为“ **关闭（Off）** ”，停用 Z 缓冲区即可：

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

深度冲突（Z-fighting）发生在两个或多个物体具有相同的深度时，导致 Z 缓冲中的值完全相同。

[https://picx.zhimg.com/80/v2-f16fe9a84f37cb76f4808e66cb3c24e8.png](https://link.zhihu.com/?target=https%3A//picx.zhimg.com/80/v2-f16fe9a84f37cb76f4808e66cb3c24e8_720w.png)

![](https://pic3.zhimg.com/80/v2-66d8a9d1bed11ea11c8d6ea3f4d3e27a_720w.webp)

Fig. 3.2.2a

接着，当尝试在渲染管线末端渲染这些像素时，就会出现上图所示的这种效果。由于 Z 缓冲无法确定究竟哪个物体位于另一个物体的后面，就会产生这些闪烁的线条。线条的形状会根据摄像机的位置而改变。

想修复这个问题，我们只需要用命令关闭深度写入（ZWrite off）、禁用 Z 缓冲区即可。

#### 3.2.3 | ShaderLab深度测试

<details>
<summary>原文</summary>

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

</details>

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
