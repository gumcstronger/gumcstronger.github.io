---
layout:     post
title:      "Unity Shader 笔记"
subtitle:   " \"Unity Shader\""
date:       2022-11-24 02:37:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
---
## shader学习入门链接

* [building a graph](https://catlikecoding.com/unity/tutorials/basics/)

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

| **类型**           | **示例语法**                                                                        | **注释**                                                                                                                                                                                                                                                                                                            |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **整数**           | `_ExampleName ("Integer display name", Integer) = 1`                                                                        | This type is backed by a real integer (unlike the legacy `Int` type described below, which is backed by a float). Use this instead of Int when you want to use an integer.                                                                                                                                              |
| **Int** （旧版）   | `_ExampleName ("Int display name", Int) = 1`                                                                                | **Note:** This legacy type is backed by a float, rather than an integer. It is supported for backwards compatibility reasons only. Use the `Integer` type instead.                                                                                                                                                |
| **Float**          | `_ExampleName ("Float display name", Float) = 0.5``_ExampleName ("Float with range", Range(0.0, 1.0)) = 0.5`                | 范围滑动条的最大值和最小值包含在内。                                                                                                                                                                                                                                                                                      |
| **Texture2D**      | `_ExampleName ("Texture2D display name", 2D) = "" {}``_ExampleName ("Texture2D display name", 2D) = "red" {}`               | 将以下值置于默认值字符串中可使用 Unity 的内置纹理之一：“white”（RGBA：1,1,1,1）、“black”（RGBA：0,0,0,1）、“gray”（RGBA：0.5,0.5,0.5,1）、“bump”（RGBA：0.5,0.5,1,0.5）或“red”（RGBA：1,0,0,1）。``如果将该字符串留空或输入无效值，则它默认为 “gray”。`` **注意：** 这些默认纹理在 Inspector 中不可见。 |
| **Texture2DArray** | `_ExampleName ("Texture2DArray display name", 2DArray) = "" {}`                                                             | 有关更多信息，请参阅[纹理数组](https://docs.unity3d.com/cn/current/Manual/class-Texture2DArray.html)。                                                                                                                                                                                                                       |
| **Texture3D**      | `_ExampleName ("Texture3D", 3D) = "" {}`                                                                                    | 默认值为 “gray”（RGBA：0.5,0.5,0.5,1）纹理。                                                                                                                                                                                                                                                                            |
| **Cubemap**        | `_ExampleName ("Cubemap", Cube) = "" {}`                                                                                    | 默认值为 “gray”（RGBA：0.5,0.5,0.5,1）纹理。                                                                                                                                                                                                                                                                            |
| **CubemapArray**   | `_ExampleName ("CubemapArray", CubeArray) = "" {}`                                                                          | 请参阅[立方体贴图数组](https://docs.unity3d.com/cn/current/Manual/class-CubemapArray.html)。                                                                                                                                                                                                                                 |
| **Color**          | `_ExampleName("Example color", Color) = (.25, .5, .5, 1)`                                                                   | 这会在着色器代码中映射到 float4。``材质 Inspector 会显示一个拾色器。如果更愿意将值作为四个单独的浮点数进行编辑，请使用 Vector 类型。                                                                                                                                                                                      |
| **Vector**         | `_ExampleName ("Example vector", Vector) = (.25, .5, .5, 1)`                                                                | 这会在着色器代码中映射到 float4。``材质 Inspector 会显示四个单独的浮点数字段。如果更愿意使用拾色器编辑值，请使用 Color 类型。                                                                                                                                                                                             |

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

## Unity基础光照

标准光照模型：

不符合真实物理光照现象，但易用、速度、效果都较好，被广泛应用。又称Phong光照模型或Blinn-Phong光照模型。

但很多物理现象例如菲涅尔反射（Fresnel reflection）无法表现，或例如某些具有各向异性反射性质的例如拉丝金属、毛发，反射不会变化。

发光类型：

自发光 emissive

高光反射 specular

漫反射 diffuse

环境光 ambient

### 逐顶点光照 per-vertex lighting

,又称高洛斯着色 Gouraud shading

```
// 逐顶点光照 per-vertex lighting (细分程度低的模型背光面和向光面交界处会出现锯齿)
v2f vert(a2v v){
    v2f o;
	// 将顶点位置从模型空间转换到裁剪空间
	o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

	// 获取环境光
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	 
	// 将法线从模型空间转换到世界空间
	fixed3 worldNormal = normalize(mul(v.normal, (float3x3)_World2Object));

	// 获取世界空间的光方向
	fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
	// 计算漫反射光
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));

	// 环境光和漫反射光相加，得到最终光照结果
	o.color = ambient + diffuse;

	return o;
}

fixed4 frag(v2f i) : SV_Target {
	return fixed4(i.color, 1.0);
}
```

### 逐像素光照 per-pixel lighting

```
fixed4 frag(v2f i) : SV_Target {
	// 获取环境光
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	 
	// 将法线从模型空间转换到世界空间
	fixed3 worldNormal = normalize(mul(v.normal, (float3x3)_World2Object));

	// 获取世界空间的光方向
	fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

	// 计算漫反射光
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));

	// 环境光和漫反射光相加，得到最终光照结果
	o.color = ambient + diffuse;

	return fixed4(color, 1.0);

}
```

### 兰伯特光照模型

以上情况暗面是黑的，所以Value开发《半条命》时提出半兰伯特光照模型。

```
fixed4 frag(v2f i) : SV_Target {
	// 获取环境光
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
	 
	// 将法线从模型空间转换到世界空间
	fixed3 worldNormal = normalize(mul(v.normal, (float3x3)_World2Object));

	// 获取世界空间的光方向
	fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

	// 计算漫反射光(使用半兰伯特光照公式修改漫反射光照部分)
	fixed halfLambert = dot(worldNormal, worldLightDir) * 0.5 + 0.5;
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * halfLambert;

	// 环境光和漫反射光相加，得到最终光照结果
	o.color = ambient + diffuse;

	return fixed4(color, 1.0);

}
```

### 高光反射光照模型

```
Properties{
	_Diffuse ("Diffuse", Color) = (1, 1, 1, 1)
	_Specular ("Specular, Color) = (1, 1, 1, 1)
	_Gloss ("Gloss", Range(8.0, 256)) = 20
}

SubShader{
	Pass{
		Tags { "LightMode" = "ForwardBase" }

		CGPROGRAM
		// 定义顶点着色器和片元着色器
		#pragma vertex vert
		#pragma fragment grag

		// 包含Unity内置文件
		#include "Lighting.cginc"

		// 定于和Properties相匹配的变量
		fixed4 _Diffuse
		fixed4 _Specular
		float _Gloss

		//定义顶点着色器输入和输出结构体
		struct a2v {
			float4 vertex : POSITION;
			float3 normal : NORMAL;
		}
		struct v2f {
			float4 pos : SV_POSITION;
			fixed3 color : COLOR;
		}

		//顶点着色器中计算高光反射的光照模型
		v3f vert(a2v v){
			v2f o;
			// 将顶点位置从模型空间转换到裁剪空间
			o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

			// 获取环境光
			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

			// 将法线从模型空间转换到世界空间
			fixed3 worldNormal = normalize(mul(v.normal, (float3x3)_World2Object));

			// 获取世界空间的光照
			fixed3 worldLightDire = normalize(_WOrldSpaceLightPos0.xyz);

			// 计算漫反射光
			fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));

			// 计算入射光线方向关于表面法线的反射方向
			fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
			//  世界空间下的视角方向 = 世界空间计算机的位置 - 顶点位置的世界空间位置
			fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(_Object2World, v.vertex).xyz);

			// 计算高光
			fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);

			o.color = ambient + diffuse + specular;

			return o;
		}


		ENDCG
	}
}
```

### Phong光照模型

```
struct v2f{
	float4 pos : SV_POSITION;
	float3 worldNormal : TEXCOORD0;
	float3 worldPos : TEXCOORD1;
}

// 顶点着色器只需要计算世界空间下的法线方向和顶点方向，并传递给片元着色器
v2f vert(a2v v){
	v2f o;
	// 将顶点坐标从模型空间转换为裁剪空间
	o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

	// 将法线从模型空间转换到世界空间
	o.worldNormal = mul(v.normal, (float3x3)_World2Oject));
	// 将顶点从模型空间转换到世界空间
	o.worldPos = mul(_OBject2World, v.vertex).xyz;

	return o;
}

fixed4 frag(v2f i) : SV_Target {
	// 获取环境光
	fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

	// 计算漫反射光
	fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));

	// 计算入射光线方向关于表面法线的反射方向
	fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
	//  世界空间下的视角方向 
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
	// 计算高光
	fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);

	return fixed4(ambient + diffuse + specular, 1.0);
}
```

### Blinn-Phong光照模型

相比与Phong模型，Blinn-Phong光照模型的高光反色部分更大更亮一些。

```
fixed4 frag(v2f i) : SV_Target{
	//  世界空间下的视角方向 
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);

	// 
	fixed3 halfDir = normalize(worldLightDir + viewDir);

	// 计算高光
	fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

	return fixed4(ambient + diffuse + specular, 1.0);
}
```

### Unity内置函数

| **功能：**                                               | **描述：**                                                    |
| -------------------------------------------------------------- | ------------------------------------------------------------------- |
| `float3 WorldSpaceViewDir (float4 v)`                        | 返回从给定对象空间顶点位置朝向摄像机的世界空间方向（未normalize）。 |
| `float3 ObjSpaceViewDir (float4 v)`                          | 返回从给定对象空间顶点位置朝向摄像机的对象空间方向（未normalize）。 |
| `float2 ParallaxOffset (half h, half height, half3 viewDir)` | 计算视差法线贴图的 UV 偏移。                                        |
| `fixed Luminance (fixed3 c)`                                 | 将颜色转换为亮度（灰阶）。                                          |
| `fixed3 DecodeLightmap (fixed4 color)`                       | 从 Unity 光照贴图（RGBM 或 dLDR，具体取决于平台）解码颜色。         |
| `float4 EncodeFloatRGBA (float v)`                           | 将 [0..1) 范围浮点数编码为 RGBA 颜色，用于存储在低精度渲染目标中。  |
| `float DecodeFloatRGBA (float4 enc)`                         | 将 RGBA 颜色解码为浮点数。                                          |
| `float2 EncodeFloatRG (float v)`                             | 将 [0..1) 范围浮点数编码为 float2。                                 |
| `float DecodeFloatRG (float2 enc)`                           | 解码先前编码的 RG 浮点数。                                          |
| `float2 EncodeViewNormalStereo (float3 n)`                   | 将视图空间法线编码为 0 到 1 范围内的两个数字。                      |
| `float3 DecodeViewNormalStereo (float4 enc4)`                | 从 enc4.xy 解码视图空间法线。                                       |

## 基础纹理

### 单张纹理

```
Properties {
	_Color ("Color Tint", Color) = (1, 1, 1, 1)
	_MainTex ("Main Tex), 2D) = "white" {}
	_Specular ("Specular", Color) = (1, 1, 1, 1)
	_Gloss ("Gloss", Range(8.0, 256)) = 20
}

SubShader {
	Pass {
		Tags { "LightMode"="ForwardBase" }

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cginc"

		fixed4 _Color;
		sampler2D _MainTex;
		float4 _MainTex_ST;
		flxed4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex : POSITION;
			float3 normal : NORMAL;
			// Unity会将第一组纹理坐标存储到该变量中
			float4 texcoord : TEXCOORD0;
		}
		struct v2f {
			float4 pos :SV_POSITION;
			float3 worldNormal : TEXCOORD0;
			float3 worldPos : TEXCOORD1;
			// 存储纹理坐标
			float2 uv : TEXCOORD2;
		}

		v2f vert (a2v v){
			v2f o;
			o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

			o.worldNormal = UnityObjectToWorldNormal(v.normal);

			o,worldPos = mul(_Object2World, v.vertex).xyz;

			//o.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
			o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

			return o;
		}

		fixed4 frag(v2f i) : SV_Target {
			// 世界空间下的法线方向和光照方向
			fixed3 worldNormal = normalize(i.worldNormal);
			fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

			// 材质的反射率 = 使用CG的text2D函数对纹理进行采样(需要被采样的纹理，纹理坐标) * 颜色属性
			fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
			// 环境光 = 环境光照 * 材质的反射率
			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

			// 计算漫反射光照
			fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));

			fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
			fixed3 halfDir = normalize(worldLightDir + viewDir);
			fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

			return fixed4(ambient + diffuse + specular, 1.0);
		}

		ENDCG
	}
}
```

### Unity纹理属性

Texture Type : 纹理类型，一般是Texture、法线纹理时Normal Map，还有Cubemap等高级类型。

Alpha from Grayscale：纹理类型为Texture时，Unity Inspector界面有Alpha from Grayscale复选框，勾选了则透明通道的值会由每个像素的灰度值生成。

Wrap Mode : 纹理坐标超过[0,1]范围如何被平铺。Repeat模式下，纹理坐标超过1，整数部分会被舍弃，而直接使用小数部分进行采样。Clamp模式下，纹理坐标大于1，将会截取到1，小于0，将会截取到0；

Filter Mode : 决定纹理由于变换而产生拉伸时将会采用哪种滤波模式，Point、Bilinear、Trilinear，滤波模式依次提升，性能增大。

    纹理放大时，Point会得到偏马赛克的效果，Billnear和Trilinear会偏模糊，因为做了滤波。

    纹理缩小时，需要处理抗锯齿问题，常用多级渐远纹理mipmapping技术，当物体远离摄像机时，可以直接采用较小的纹理。

Max Size：最大分辨率，需要2的幂大小

Format： Unity内部使用哪种格式存储。

### 凹凸映射 bump mapping

使用一张纹理来修改模型表面的法线，以便为模型提供更多细节。

有两种主要方法提供凹凸映射：

1、高度映射（height mapping）：使用一张高度纹理height map，来模拟表面移位displacement，然后得到一个修改后的法线值。

height map中存储的是强度值 intensity，表示模型表面局部的海拔高度，颜色越浅该位置表面越向外凸起。颜色越深该位置表面越向里凹。缺点是不能试试得到表面法线，而是需要由像素的灰度值计算而得，需要消耗更多性能。通常会跟法线映射一起使用，高度图提供表面凹凸的额外信息，根据法线映射来修改光照。

2、法线映射（normal mapping）：使用一张法线纹理来直接存储表面法线。

法线纹理存储的是表面法线方向，但法线方向范围是[-1, 1]，像素分类范围是[0, 1]，因此需要处理：pixel = (normal + 1) / 2。那么在shader中对法线纹理进行纹理采样后，需要对结果进行一次反映射得到原先的法线方向：normal = pixel * 2 - 1；

模型空间的法线纹理：object-space normal map，法线方向在模型空间。

实际一般采用切线空间的法线纹理tangent-space normal map，模型顶点的切线空间tangent space来存储法线。看起来几乎都是浅蓝色的。

切线空间下计算光照模型的代码如下：

```
Properties {
	_Color ("Color Tint", Color) = (1, 1, 1, 1)
	_MainTex ("Main Tex", 2D) = "white" {}
	// 法线纹理，bump是Unity内置的法线纹理
	_BumpMap ("Normal Map", 2D) = "bump" {}
	// 凹凸成都，为0时法线纹理不会对光照产生任何影响
	_BumpScale ("Bump Scale", Float) = 1.0
	_Specular ("Specular", Color) = (1, 1, 1, 1)
	_Gloss ("Gloss", Range(8.0, 256)) = 20
}

SubShader {
	Pass {
		Tags { "LightMode"="ForwardBase" }

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cginc"

		// 与Properties匹配的变量
		fixed4 _Color;
		sampler2D _MainTex;
		float4 _MainTex_ST;
		sampler2D _BumpMap;
		float4 _BumpMap_ST;
		float _BumpScale;
		fixed4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex : POSITION;
			float3 normal : NORMAL;
			float4 tangent ： TANGENT;
			float4 texcoord : TEXCOORD0;
		}
		struct v2f {
			float4 pos : SV_POSITION;
			float4 uv : TEXCOORD0;
			float3 lightDir : TEXCOORD1;
			float3 viewDir : TEXCOORD2;
		}

		v2f vert (a2v v){
			v2f o;
			o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

			//uv的xy分量存储_MainTex的纹理坐标，zw存储_BumpMap的纹理坐标
			o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
			o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;

			// Unity内置函数,将模型空间下切线方向、副切线方向和法线方向按行排列来得到从模型空间到切线空间的变换矩阵rotation
			TANGENT_SPACE_ROTATION;

			// 光方向从模型空间转换成切线空间
			o,lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
			// 视角方向从模型空间转换成切线空间
			o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
		}

		fixed4 frag(v2f i) : SV_Target{
			fixed3 tangentLightDir = normalize(i.lightDir);
			fixed3 tangentViewDir = normalize(i.viewDir);

			// 利用tex2D对法线纹理_BumpMap进行采样
			fixed4 packedNormal = tex2D(_BumpMap, i.uv.zw);
			fixed3 tangentNormal;

			// 如果texture类型不是Normal Map,则需要将像素值反映射回来，然后再乘以_BumpScale(控制凹凸程度)来得到tangentNormal的xy分量。由于法向是单位向量，所以可以计算得到z。
			// tangentNormal.xy = (packedNormal.xy * 2 - 1) * _BumpScale;
			// tangentNormal.z = sqrt (1.0 - saturate(dot(tangentNormal.xy , tangentNormal.xy)));

			// 
			tangentNormal = UnpackNormal(packedNormal);
			tangentNormal.xy *= _BumpScale;
			tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));

			fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;

			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

			fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));

			fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
			fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss);

			return fixed4 (ambient + diffuse + specular, 1.0);
		}

		ENDCG
	}
}
```

### 渐变纹理

纹理可以用于存储任何表面属性，一个常见用法就是使用渐变纹理控制漫反射光照的结果。

```
Properties {
	_Color ("Color Tint", Color) = (1, 1, 1, 1)
	_RampTex ("Ramp Tex", 2D) = "white" {}
	_Specular ("Specular", Color) = (1, 1, 1, 1)
	_Gloss ("Gloss", Range(8.0, 256)) = 20
}

SubShader {
	Pass {
		Tags {"LightMode"="ForwardBase"}

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cgnic"

		fixed4 _Color;
		sampler2D _RampTex;
		float4 _RampTex_ST;
		float4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex : POSITION;
			float3 normal : NORMAL;
			float4 texcoord : TEXCOORD0;
		}
		struct v2f {
			float4 pos : SV_POSITION;
			float3 worldNormal : TEXCOORD0;
			float3 worldPos : TEXCOORD1;
			float2 uv : TEXCOORD2;
		}

		v2f vert (a2v v){
			v2f o;
			o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

			o.worldNormal = UnityObjectToWorldNormal(v.normal);

			o.worldPos = mul(_Object2World, v.vertex).xyz;

			// 使用TRANSFORM_TEX来计算景观平铺和偏移后的纹理坐标
			o.uv = TRANSFORM_TEX(v.texcoord, _RampTex);

			return o;
		}

		fixed4 frag (v2f i) : SV_Target {
			fixed3 worldNormal = normalize(i.worldNormal);
			fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

			// 对法线方向和光照方向的点击做一次0.5倍缩放和0.5大小的便宜来计算半兰伯特部分halfLambert，[0, 1]
			fixed halfLambert = 0.5 * dot(worldNormal, worldLightDir) + 0.5;
			// 使用halfLambert来构建一个纹理坐标，并用这个纹理坐标对渐变纹理_RampTex进行采样。
			// 由于_RampTex实际是一个一维纹理，因此纹理坐标u、v方向我们都是用了halfLambert
			// 得到的颜色和材质颜色_Color相乘，得到最终漫反射颜色。
			fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert)).rgb * _Color.rgb;

			fixed3 diffuse = _LightColor0.rgb * diffuseColor;

			fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
			fixed3 halfDir = normalize(worldLightDir + viewDir);
			fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

			return fixed4(ambient + diffuse + specular, 1.0);
		}

		ENDCG
	}
}
```

### 遮罩纹理 mask texture

有时候我们希望模型表面某些区域的反光强烈一些，而某些区域弱一些，为了得到更加细腻的效果，可以使用一张遮罩纹理来控制光照。另一种常见的引用时制作地形材质时需要混合多张图片，例如表现草地的纹理、表现石头的纹理、表现裸漏土地的纹理，使用遮罩可以控制如何混合这些纹理。
使用遮罩纹理的流程一般是：通过采样得到遮罩纹理的纹素值，然后使用其中某个（或某几个）通道的值(例如texel.r)来与某种表面属性进行相乘，这样当该通道的值为0时，可以保护表面不受该属性的影响。总而言之，使用遮罩纹理可以让美术人员更加精准（像素级）地控制模型表面的各种性质。

```
Properties {
	_Color ("Color Tint", Color) = (1, 1, 1, 1)
	_MainTex ("Main Tex", 2D) = "white" {}
	_BumpMap ("Normal Map", 2D) = "bump" {}
	_BumpScale ("Bump Scale", Float) = 1.0
	_SpecularMask ("Specular Mask", 2D) = "white" {}
	_SpecularScale ("Specular Scale", Float) = 1.0
	_Specular ("Specular", Color) = (1, 1, 1, 1)
	_Gloss ("Gloss", Range(8.0, 256)) = 20 
}

SubShader {
	Pass {
		Tags { "LightMode"="ForwardBase" }
	}

	CGPROGRAM
	#pragma vertex vert
	#pragma fragment grag

	#include "Lighting.cginc"

	flxed4 _Color;
	sampler2D _MainTex;
	// 主纹理、法线纹理、遮罩纹理定义了共同使用的纹理属性变量_MainTex_ST
	float4 _MainTex_ST;
	sampler2D _BumpMap;
	float _BumpScale;
	sampler2D _SpecularMask;
	float _SpecularScale;
	fixed4 _Specular;
	float _Gloss;

	struct a2v {
		float4 vertex : POSITION;
		float3 normal : NORMAL;
		float4 tangent : TANGENT;
		float4 texcoord : TEXCOORD0;
	}
	struct v2f {
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		float3 lightDir : TEXCOORD1;
		float3 viewDir : TEXCOORD2;
	}

	v2f vert (a2v v){
		v2f o;
		o.pos = mul(UNITY_MATRIX_MVP, v.vertex);

		o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;

		TANGENT_SPACE_ROTATION;

		//对光照方向和视角方向从模型空间变换到切线空间
		o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
		o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;

		return o;
	}

	fixed4 frag (v2f i) : SV_Target {
		fixed3 tangentLightDir = normalize(i.lightDir);
		fixed3 tangentViewDir = normalize(i.viewDir);

		fixed3 tangentNormal = UnpackNormal(tex2D(_BumpMap, i.uv));
		tangentNormal.xy *= _BumpScale;
		tangentNormal.z = sqrt (1.0, saturate(dot(tangentNormal.xy, tangentNormal.xy)));

		fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;

		fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

		fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));

		fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);

		// 对遮罩纹理进行采样，使用r分量来计算掩码值，掩码值跟_SpecularScale相乘一起来控制高光反射的强度。
		fixed specularMask = tex2D(_SpecularMask, i.uv).r * _SpecularScale;

		fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss) * specularMask;

		return fixed4(ambient + diffuse + specular, 1.0);
	}

	ENDCG
}
```

在游戏制作过程中，遮罩纹理可以存储任何我们希望逐像素控制的表面属性，通常我们会利用一张我呢里的RGBA四个通道存储不同的属性，例如把高光反射的强度存储在R通道，把边缘光照的强度存储在G通道，把高光反射的指数部分存储在B通道，把自发光强度存储在A通道。
在《DOTA2》中，每个模型使用4张纹理，一张用于定义模型颜色，一张用于定义表面法线，另外两张则都是遮罩纹理。

## 透明效果

透明测试 Alpha Test：只要某个片元的透明度不满足条件，就会被舍弃。不会再进行任何处理，也不会对颜色缓冲产生任何影响。透明测试不需要关闭深度写入ZWrite。要么完全透明，要么完全不透明。
透明度混合 Alpha Blending：可以得到真正的半透明，使用当前片元的透明度作为混合银子，与已经存储的颜色缓冲中的颜色值进行混合，得到新得颜色。需要关闭深度写入ZWrite。

深度写入ZWrite：深度缓冲Depth buffer/z-buffer，解决可见性问题，决定哪个物体得哪些部分被渲染在前面，根据深度缓存中得值来判断该片元距离摄像机得距离，当渲染一个片元时，需要把它得深度值和已经存在于深度缓冲中的值进行比较，值如果距离摄像机更远，则不会被渲染。但如果开启了透明度混合，需要关闭深入写入，就是另外的了。因为不关闭的话，半透明背后的表面被剔除，无法看到半透明后面的物体。

渲染引擎一般会对物体进行排序再渲染，常用方法是
1）渲染所有不透明物体，并开启透明的深度测试和深度写入
2）把半透明物体按距离摄像机的远近进行排序，然后按照从后往前的顺序渲染这些半透明物体，并开启透明的深度测试，关闭深度写入。

### Unity Shader的渲染顺序

Unity为了解决渲染顺序的问题提供了渲染队列render queue解决方案。我们可以使用SubShader的Queue标签来决定我们的模型归于哪个渲染队列。

| **功能名称**                                     | **内置渲染管线** | **通用渲染管线 (URP)** | **高清渲染管线 (HDRP)**              | **自定义 SRP**                                                                                                                            |
| ------------------------------------------------------ | ---------------------- | ---------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **ShaderLab：子着色器标签代码块**                | 是                     | 是                           | 是                                         | 是                                                                                                                                              |
| **ShaderLab：RenderPipeline 子着色器标签**       | 否                     | 是                           | 是                                         | 否                                                                                                                                              |
| **ShaderLab：Queue 子着色器标签**                | 否                     | 是                           | 是                                         | 是``**注意：** 在自定义 SRP 中，可以定义自己的渲染顺序并选择是否要使用渲染队列。有关更多信息，请参阅 DrawingSettings 和 SortingCriteria。 |
| **ShaderLab：RenderType 子着色器标签**           | 是                     | 是                           | 是                                         | 是                                                                                                                                              |
| **ShaderLab：DisableBatching 子着色器标签**      | 是                     | 是                           | 是                                         | 是                                                                                                                                              |
| **ShaderLab：ForceNoShadowCasting 子着色器标签** | 是                     | 是                           | 是``这会禁用常规阴影，但是不影响接触阴影。 | 是                                                                                                                                              |
| **ShaderLab：CanUseSpriteAtlas 子着色器标签**    | 是                     | 是                           | 是                                         | 是                                                                                                                                              |
| **ShaderLab：PreviewType 子着色器标签**          | 是                     | 是                           | 是                                         | 是                                                                                                                                              |

| **签名** | **值** | **功能**                                        |
| -------------- | ------------ | ----------------------------------------------------- |
| [queue name]   | Background   | 指定背景渲染队列。                                    |
|                | Geometry     | 指定几何体渲染队列。                                  |
|                | AlphaTest    | 指定 AlphaTest 渲染队列。                             |
|                | Transparent  | 指定透明渲染队列。                                    |
|                | Overlay      | 指定覆盖渲染队列。                                    |
| [offset]       | 整数         | 指定 Unity 渲染未命名队列处的索引（相对于命名队列）。 |

```
Properties {
	_Color ("Main Tint", Color) = (1,1,1,1)
	_MainTex ("Main Tex", 2D) = "white" {}
	_AlphaScale ("Alpha Scale", Range(0, 1)) = 1
}

SubShader {
	Tags {"Queue"="Transparent" "IgnoreProjector"="True” "RenderType"="Transparent"}
	Pass {
		Tags {"LightMode"="ForwardBase"}
		// 关闭深度写入ZWrite
		ZWrite Off 
		// 源颜色（片元着色器产生的颜色)的混合因子设为SrcAlpha,目标颜色（已存在于颜色缓冲中的颜色）的混合因子设为OneMinnusSrcAlpha以得到合适的透明效果
		Blend SrcAlpha OneMinusSrcAlpha

		fixed4 _Color;
		sampler2D _MainTex;
		float4 _MainTex_ST;
		fixed _AlphaScale;

		//片元着色器
		fixed4 frag (v2f i) : SV_Target {
			fixed3 worldNormal = normalize(i.worldNormal);
			fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

			fixed4 texColor = tex2D(_MainTex, i.uv);
			fixed3 albedo = texColor.rgb * _Color.rgb;
			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
			fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));

			return fixed4(ambient + diffuse, texColor.a * _AlphaScale);
		}
	}
}
```

### 开启深度写入的半透明

上面的透明代码由于关闭深度写入会造成错误排序，一种解决办法是使用两个Pass来渲染模型，第一个开启深度写入，但不输出颜色，目的仅仅是为了将该模型的深度值写入深度缓冲中。第二个Pass进行侦察的透明度混合，由于上一个Pass已经得到逐像素的正确深度信息，该Pass就可以按照像素级别的深度排序结果进行透明渲染。但这种方法的缺点在于多一个Pass会对性能造成一定影响。

### ShaderLab的混合命令

### 双面渲染的透明效果

## 更复杂的光照
