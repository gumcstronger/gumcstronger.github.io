---
layout:     post
title:      "Unity Shader 笔记"
subtitle:   " \"Unity Shader\""
date:       2022-11-24 02:37:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开源
---
## 渲染流水线

应用阶段(Application Stage)  ：输出渲染图元

几何阶段(Geometry Stage)	：输出屏幕空间的顶点信息

光栅化阶段(Rasterizer Stage)

## Unity Shader基础

### ShaderLab

一种说明性语言，用来编写Unity Shader，基础结构如下：Shader "ShaderName"{

```shader
    // 属性
    Properties {
        Name ("display name", PropertyType) = DefaultValue
    }
    // 子着色器(多个子着色器则优先选择第一个，第一个不能执行则选择第二个，以此类推，都不能执行再Fallback)
    // 真正的Shader代码在这里，例如Surface Shader 或 Vertex / Fragment Shader / Fixed Function Shader
    SubShader {
        // 标签，可选
        [Tags]
  
        // 渲染状态，可选
        [RenderSetup]

 	// 一系列Pass
        Pass {
   	    [Name]
	    [Tags]
	    [RenderSetup]
	    //other code

	    // CG代码(CGPROGRAM / ENDCG)或HLSL代码(HLSLPROGRAM / ENDHLSL)
	    CGPROGRAM 

	    #pragma vertex vert
	    #pragma fragment frag

	    float4 vert(float4 v : POSITION) : SV_POSITION {
	        return mul (UNITY_MATRIX_MVP, v);
	    }

	    fixed4 frag() : SV_Target {
	        return fixed4(1.0, 0.0, 0.0, 1.0);
	    }

	    ENDCG
        }
	// Other Passes

	// 使用其他shader的pass
	UsePass "OtherShaderPass"

	// 抓取屏幕并将结果存储在一张纹理中，以便后续的Pass处理
	GrabPass "_WaterBackgroundTexture"
    }
    // 所有SubShader不能使用则使用该最低级的，或者Fallback Off关闭Fallback功能
    Fallback "VectoxLit"
}
```

### Properties属性类型

| 属性类型        | 默认值定义语法                   | 例子                                     |
| --------------- | -------------------------------- | ---------------------------------------- |
| Int             | number                           | _Int("Int", Int) = 2                     |
| Float           | number                           | _Float("Float", Float) = 1.5             |
| Range(min, max) | number                           | _Range("Range", Range(0.0, 5.0)) = 3.0   |
| Color           | (number, number, number, number) | _Color("Color", Color) = (1,1,1,1)       |
| Vector          | (number, number, number, number) | _Vector("Vector", Vector) = (2, 3, 6, 1) |
| 2D              | "defaulttexture" {}              | _2D("2D", 2D) = "" {}                    |
| Cube            | "defaulttexture" {}              | _Cube("Cube", Cube) = "white" {}         |
| 3D              | "defaulttexture" {}              | _3D{"3D", 3D} = "black" {}               |

### SubShader

| 标签类型            | 说明                                                                                                                                                                                 | 例子                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------- |
| Queue               | 控制渲染顺序，指定该物体应该属于哪个渲染队列，通<br />过这种方式可以保证所有的透明物体可以在所有不透明<br />物体后面被渲染。我们也可以自定义使用的渲染队列来<br />控制物体的渲染顺序 | Tags { "Queue" = "Transparent" }        |
| RenderType          | 对着色器进行分类，例如这是一个不透明的着色器，或<br />是一个透明的着色器。这可以被用于着色器替换功能。                                                                               | Tags { "RenderType" = "Opaque" }        |
| DisableBatching     | 是否对该SubShader使用批处理                                                                                                                                                          | Tags { "DisableBatching" = "True" }     |
| ForceNoShadowCating | 控制使用该SubShader的物体是否会投射阴影                                                                                                                                              | Tags { "ForceNoShadowCating" = "True" } |
| IgnoreProjector     | True则不受Projector的影响，通常用于半透明物体                                                                                                                                        | Tags { "IgnoreProjector" = "True" }     |
| CanUseSpriteAtlas   | 当该SubShader是用于Sprites时，设置为False                                                                                                                                            | Tags { "CanUseSpriteAtlas" = "False" }  |
| PreviewType         | 指明材质面板如何预览该材质。默认情况下，材质面板<br />显示为球形，可以设置为"Plane"、“SkyBox”                                                                                      | Tags { "PreviewType" = "Plane" }        |

### RenderSetup

| 状态名称 | 设置指令                                                         | 解释                                   |
| -------- | ---------------------------------------------------------------- | -------------------------------------- |
| Cull     | Cull Back \ Front \ Off                                          | 设置剔除模式：剔除背面、正面、关闭剔除 |
| ZTest    | ZTest Less Greater \ LEqual \ GEqual \ Equal \ NotEqual \ Always | 设置深度测试时使用的函数               |
| ZWrite   | ZWrite On \ Off                                                  | 开启、关闭深度写入                     |
| Blend    | Blend SrcFactor DstFactor                                        | 开启并设置混合模式                     |

Pass Tags

| 标签类型       | 说明                                  | 例子                                            |
| -------------- | ------------------------------------- | ----------------------------------------------- |
| LightMode      | 定义该Pass在Unity的渲染流水线中的角色 | Tags {  "LightMode" = "ForwardBase" }         |
| RequireOptions | 用于指定当满足某些条件时才渲染该Pass  | Tags {  "RequireOptions" = "SoftVegetation" } |

## Shader数学基础（即线性代数的基础部分）

### 笛卡尔坐标系 Cartesian Coordinate System

左手坐标系和右手坐标系：左右手坐标系无法通过旋转来转换，左右手坐标系Z轴都指向外。

Unity使用的是左手坐标系，但对于观察空间来说，Unity使用的是右手坐标系，也就是以摄像机为原点的坐标系

### 向量的计算

向量与常量的乘除：略

向量的加法和减法：略

向量的模：长度

向量归一化：单位向量 normalized

#### 向量点积

dot(a, b) = a * b = (ax, ay, az) * (bx, by, bz) = ax*bx + ay*by + az*bz，点积的集合意义是投影（projection)

dot(a, b) = |a||b|CosX

#### 向量叉积

produte(a, b) = a X b = (ax, ay, ax)*(bx, by, bz) = (ay*bz - az*bz, az*bx - ax*bz, ax*by - ay*bx)

produte(a, b) = |a||b|SinX

### 矩阵

矩阵和常量的乘法：略

矩阵和矩阵的乘法：

r x b的矩阵 与 n x c的矩阵 相乘得到 r x c的矩阵，两个相乘的矩阵必须满足n相等的条件，不然不能相乘。

AB != BA

(AB)C = A(BC)

方块矩阵：3x3或4x4的矩阵，行列相等的矩阵

单位矩阵：任何矩阵和他相乘依旧是原来的矩阵即是单位矩阵。

转置矩阵 transposed matrix

逆矩阵 inverse matrix

正交矩阵 orthogonal matrix

### 矩阵的意义：变换

缩放scale、旋转rotation、错切shear、镜像reflection、正交投影orthographic projection

### 裁剪空间

在Camera中会使用到，两种投影类型：正交投影 orthographic projection  和 透视投影 perspective projection

### 屏幕空间

将视锥体投影到屏幕空间，得到2D像素位置，需要进行齐次除法 homogeneous division，也叫透视除法 perspective division

### 法线变换

TODO:

### [Unity Shader内置变量](https://docs.unity.cn/cn/2022.1/Manual/SL-UnityShaderVariables.html)

Unity内置变换矩阵

| 变量名             | 描述                       |
| ------------------ | -------------------------- |
| UNITY_MATRIX_MVP   | 当前模型 * 视图 * 投影矩阵 |
| UNITY_MATRIX_MV    | 当前模型 * 视图矩阵。      |
| UNITY_MATRIX_V     | 当前视图矩阵。             |
| UNITY_MATRIX_P     | 当前投影矩阵。             |
| UNITY_MATRIX_VP    | 当前视图 * 投影矩阵        |
| UNITY_MATRIX_T_MV  | 模型转置 * 视图矩阵。      |
| UNITY_MATRIX_IT_MV | 模型逆转置 * 视图矩阵。    |
| _Object2World      | 当前模型矩阵。             |
| _World2Object      | 当前世界矩阵的逆矩阵。     |

摄像机和屏幕

| **名称**                 | **类型** | **值**                                                                                                                                                                                            |
| ------------------------------ | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _WorldSpaceCameraPos           | float3         | 摄像机的世界空间位置。                                                                                                                                                                                  |
| _ProjectionParams              | float4         | `x` 是 1.0（如果当前使用[翻转投影矩阵](https://docs.unity.cn/cn/2022.1/Manual/SL-PlatformDifferences.html)进行渲染，则为 –1.0），`y` 是摄像机的近平面，`z` 是摄像机的远平面，`w` 是远平面的倒数。 |
| _ScreenParams                  | float4         | `x` 是摄像机目标纹理的宽度（以像素为单位），`y` 是摄像机目标纹理的高度（以像素为单位），`z` 是 1.0 + 1.0/宽度，`w` 为 1.0 + 1.0/高度。                                                          |
| _ZBufferParams                 | float4         | 用于线性化 Z 缓冲区值。`x` 是 (1-远/近)，`y` 是 (远/近)，`z` 是 (x/远)，`w` 是 (y/远)。                                                                                                         |
| unity_OrthoParams              | float4         | `x` 是正交摄像机的宽度，`y` 是正交摄像机的高度，`z` 未使用，`w` 在摄像机为正交模式时是 1.0，而在摄像机为透视模式时是 0.0。                                                                      |
| unity_CameraProjection         | float4x4       | 摄像机的投影矩阵。                                                                                                                                                                                      |
| unity_CameraInvProjection      | float4x4       | 摄像机投影矩阵的逆矩阵。                                                                                                                                                                                |
| unity_CameraWorldClipPlanes[6] | float4         | 摄像机视锥体平面世界空间方程，按以下顺序：左、右、底、顶、近、远。                                                                                                                                      |

时间

| **名称**  | **类型** | **值**                                                             |
| --------------- | -------------- | ------------------------------------------------------------------------ |
| _Time           | float4         | 自关卡加载以来的时间 (t/20, t, t*2, t*3)，用于将着色器中的内容动画化。 |
| _SinTime        | float4         | 时间正弦：(t/8, t/4, t/2, t)。                                           |
| _CosTime        | float4         | 时间余弦：(t/8, t/4, t/2, t)。                                           |
| unity_DeltaTime | float4         | 增量时间：(dt, 1/dt, smoothDt, 1/smoothDt)。                             |

光照

| **名称**                                          | **类型** | **值**                                                                                     |
| ------------------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| _LightColor0*（在 UnityLightingCommon.cginc 中声明）*   | fixed4         | 光源颜色。                                                                                       |
| _WorldSpaceLightPos0                                    | float4         | 方向光：（世界空间方向，0）。其他光源：（世界空间位置，1）。                                     |
| unity_WorldToLight*（在 AutoLight.cginc 中声明）*       | float4x4       | 世界/光源矩阵。用于对剪影和衰减纹理进行采样。                                                    |
| unity_4LightPosX0、unity_4LightPosY0、unity_4LightPosZ0 | float4         | *（仅限 ForwardBase 通道）* 前四个非重要点光源的世界空间位置。                                 |
| unity_4LightAtten0                                      | float4         | *（仅限 ForwardBase 通道）* 前四个非重要点光源的衰减因子。                                     |
| unity_LightColor                                        | half4[4]       | *（仅限 ForwardBase 通道）* 前四个非重要点光源的颜色。                                         |
| unity_WorldToShadow                                     | float4x4[4]    | World-to-shadow matrices. One matrix for Spot Lights, up to four for directional light cascades. |

延迟着色和延迟光照，在光照通道着色器中使用（全部在 UnityDeferredLibrary.cginc 中声明）：

| **名称**      | **类型** | **值**                                                                                     |
| ------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| _LightColor         | float4         | 光源颜色。                                                                                       |
| unity_WorldToLight  | float4x4       | 世界/光源矩阵。用于对剪影和衰减纹理进行采样。                                                    |
| unity_WorldToShadow | float4x4[4]    | World-to-shadow matrices. One matrix for Spot Lights, up to four for directional light cascades. |

顶点光照渲染（Vertex 通道类型）：

最多可为 Vertex 通道类型设置 8 个光源；始终从最亮的光源开始排序。因此，如果您希望 一次渲染受两个光源影响的对象，可直接采用数组中前两个条目。如果影响对象 的光源数量少于 8，则其余光源的颜色将设置为黑色。

| **名称**      | **类型** | **值**                                                                                                                                                                                        |
| ------------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unity_LightColor    | half4[8]       | 光源颜色。                                                                                                                                                                                          |
| unity_LightPosition | float4[8]      | View-space light positions. (-direction,0) for directional lights; (position,1) for Point or Spot Lights.                                                                                           |
| unity_LightAtten    | half4[8]       | Light attenuation factors.*x* is cos(spotAngle/2) or –1 for non-Spot Lights; *y* is 1/cos(spotAngle/4) or 1 for non-Spot Lights; *z* is quadratic attenuation; *w* is squared light range. |
| unity_SpotDirection | float4[8]      | View-space Spot Lights positions; (0,0,1,0) for non-Spot Lights.                                                                                                                                    |

光照贴图

| **名称**   | **类型** | **值**                                             |
| ---------------- | -------------- | -------------------------------------------------------- |
| unity_Lightmap   | Texture2D      | 包含光照贴图信息。                                       |
| unity_LightmapST | float4[8]      | 缩放 UV 信息并转换到正确的范围以对光照贴图纹理进行采样。 |

雾效和环境光

| **名称**           | **类型** | **值**                                                                                                                                                                                       |
| ------------------------ | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unity_AmbientSky         | fixed4         | 梯度环境光照情况下的天空环境光照颜色。                                                                                                                                                             |
| unity_AmbientEquator     | fixed4         | 梯度环境光照情况下的赤道环境光照颜色。                                                                                                                                                             |
| unity_AmbientGround      | fixed4         | 梯度环境光照情况下的地面环境光照颜色。                                                                                                                                                             |
| UNITY_LIGHTMODEL_AMBIENT | fixed4         | 环境光照颜色（梯度环境情况下的天空颜色）。旧版变量。                                                                                                                                               |
| unity_FogColor           | fixed4         | 雾效颜色。                                                                                                                                                                                         |
| unity_FogParams          | float4         | 用于雾效计算的参数：(density / sqrt(ln(2))、density / ln(2)、–1/(end-start) 和 end/(end-start))。*x* 对于 Exp2 雾模式很有用；_y_ 对于 Exp 模式很有用，_z_ 和 *w* 对于 Linear 模式很有用。 |

其他

| **名称**    | **类型** | **值**                                                                                                                                                                      |
| ----------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unity_LODFade     | float4         | 使用[LODGroup](https://docs.unity.cn/cn/2022.1/Manual/class-LODGroup.html) 时的细节级别淡入淡出。*x* 为淡入淡出（0 到 1），_y_ 为量化为 16 级的淡入淡出，_z_ 和 *w* 未使用。 |
| _TextureSampleAdd | float4         | 根据所使用的纹理是 Alpha8 格式（值设置为 (1,1,1,0)）还是不是该格式（值设置为 (0,0,0,0)）由 Unity **仅针对 UI **自动设置。                                                         |
