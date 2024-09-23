---
layout:     post
title:      "Unity URP SimpleLit.shader 学习"
subtitle:   " \"Unity Shader\""
date:       2022-12-11 12:37:00
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
## SimpleLit.shader

```csharp
// Shader targeted for low end devices. Single Pass Forward Rendering.
Shader "Universal Render Pipeline/Simple Lit"
{
    // Keep properties of StandardSpecular shader for upgrade reasons.
    Properties
    {
        [MainTexture] _BaseMap("Base Map (RGB) Smoothness / Alpha (A)", 2D) = "white" {}
        [MainColor]   _BaseColor("Base Color", Color) = (1, 1, 1, 1)

        _Cutoff("Alpha Clipping", Range(0.0, 1.0)) = 0.5

        _Smoothness("Smoothness", Range(0.0, 1.0)) = 0.5
        _SpecColor("Specular Color", Color) = (0.5, 0.5, 0.5, 0.5)
        _SpecGlossMap("Specular Map", 2D) = "white" {}
        _SmoothnessSource("Smoothness Source", Float) = 0.0
        _SpecularHighlights("Specular Highlights", Float) = 1.0

        [HideInInspector] _BumpScale("Scale", Float) = 1.0
        [NoScaleOffset] _BumpMap("Normal Map", 2D) = "bump" {}

        [HDR] _EmissionColor("Emission Color", Color) = (0,0,0)
        [NoScaleOffset]_EmissionMap("Emission Map", 2D) = "white" {}

        // Blending state
        _Surface("__surface", Float) = 0.0
        _Blend("__blend", Float) = 0.0
        _Cull("__cull", Float) = 2.0
        [ToggleUI] _AlphaClip("__clip", Float) = 0.0
        [HideInInspector] _SrcBlend("__src", Float) = 1.0
        [HideInInspector] _DstBlend("__dst", Float) = 0.0
        [HideInInspector] _SrcBlendAlpha("__srcA", Float) = 1.0
        [HideInInspector] _DstBlendAlpha("__dstA", Float) = 0.0
        [HideInInspector] _ZWrite("__zw", Float) = 1.0
        [HideInInspector] _BlendModePreserveSpecular("_BlendModePreserveSpecular", Float) = 1.0

        [ToggleUI] _ReceiveShadows("Receive Shadows", Float) = 1.0
        // Editmode props
        _QueueOffset("Queue offset", Float) = 0.0

        // ObsoleteProperties
        [HideInInspector] _MainTex("BaseMap", 2D) = "white" {}
        [HideInInspector] _Color("Base Color", Color) = (1, 1, 1, 1)
        [HideInInspector] _Shininess("Smoothness", Float) = 0.0
        [HideInInspector] _GlossinessSource("GlossinessSource", Float) = 0.0
        [HideInInspector] _SpecSource("SpecularHighlights", Float) = 0.0

        [HideInInspector][NoScaleOffset]unity_Lightmaps("unity_Lightmaps", 2DArray) = "" {}
        [HideInInspector][NoScaleOffset]unity_LightmapsInd("unity_LightmapsInd", 2DArray) = "" {}
        [HideInInspector][NoScaleOffset]unity_ShadowMasks("unity_ShadowMasks", 2DArray) = "" {}
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "SimpleLit" "IgnoreProjector" = "True" "ShaderModel"="4.5"}
        LOD 300

        Pass
        {
            Name "ForwardLit"
            Tags { "LightMode" = "UniversalForward" }

            // Use same blending / depth states as Standard shader
            Blend[_SrcBlend][_DstBlend], [_SrcBlendAlpha][_DstBlendAlpha]
            ZWrite[_ZWrite]
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local_fragment _EMISSION
            #pragma shader_feature_local _RECEIVE_SHADOWS_OFF
            #pragma shader_feature_local_fragment _SURFACE_TYPE_TRANSPARENT
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _ _ALPHAPREMULTIPLY_ON _ALPHAMODULATE_ON
            #pragma shader_feature_local_fragment _ _SPECGLOSSMAP _SPECULAR_COLOR
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            // -------------------------------------
            // Universal Pipeline keywords
            // _MAIN_LIGHT_SHADOWS决定是否使用主光阴影
            // _MAIN_LIGHT_SHADOWS_CASCADE是是否主光阴影使用级联阴影
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
            // Additional Lights相关的关键字是控制附加灯相关的光照和阴影
            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            // Lightmap shadow mixing和shadow mask是Unity的混合光照相关。
            #pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
            #pragma multi_compile _ SHADOWS_SHADOWMASK
            #pragma multi_compile_fragment _ _ADDITIONAL_LIGHT_SHADOWS
            #pragma multi_compile_fragment _ _SHADOWS_SOFT
            #pragma multi_compile_fragment _ _SCREEN_SPACE_OCCLUSION
            #pragma multi_compile_fragment _ _DBUFFER_MRT1 _DBUFFER_MRT2 _DBUFFER_MRT3
            #pragma multi_compile_fragment _ _LIGHT_LAYERS
            #pragma multi_compile_fragment _ _LIGHT_COOKIES
            #pragma multi_compile _ _CLUSTERED_RENDERING

            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile _ DIRLIGHTMAP_COMBINED
            // lightmap开关
            #pragma multi_compile _ LIGHTMAP_ON
            #pragma multi_compile _ DYNAMICLIGHTMAP_ON
            #pragma multi_compile_fog
            #pragma multi_compile_fragment _ DEBUG_DISPLAY

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma instancing_options renderinglayer
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #pragma vertex LitPassVertexSimple
            #pragma fragment LitPassFragmentSimple
            #define BUMP_SCALE_NOT_SUPPORTED 1

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitForwardPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            ColorMask 0
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON

            // -------------------------------------
            // Universal Pipeline keywords

            // This is used during shadow map generation to differentiate between directional and punctual light shadows, as they use different formulas to apply Normal Bias
            #pragma multi_compile_vertex _ _CASTING_PUNCTUAL_LIGHT_SHADOW

            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "GBuffer"
            Tags{"LightMode" = "UniversalGBuffer"}

            ZWrite[_ZWrite]
            ZTest LEqual
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            //#pragma shader_feature _ALPHAPREMULTIPLY_ON
            #pragma shader_feature_local_fragment _ _SPECGLOSSMAP _SPECULAR_COLOR
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local_fragment _EMISSION
            #pragma shader_feature_local _RECEIVE_SHADOWS_OFF

            // -------------------------------------
            // Universal Pipeline keywords
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
            //#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            //#pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS
            #pragma multi_compile_fragment _ _SHADOWS_SOFT
            #pragma multi_compile_fragment _ _DBUFFER_MRT1 _DBUFFER_MRT2 _DBUFFER_MRT3
            #pragma multi_compile_fragment _ _LIGHT_LAYERS

            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile _ DIRLIGHTMAP_COMBINED
            #pragma multi_compile _ LIGHTMAP_ON
            #pragma multi_compile _ DYNAMICLIGHTMAP_ON
            #pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
            #pragma multi_compile _ SHADOWS_SHADOWMASK
            #pragma multi_compile_fragment _ _GBUFFER_NORMALS_OCT
            #pragma multi_compile_fragment _ _RENDER_PASS_ENABLED

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma instancing_options renderinglayer
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #pragma vertex LitPassVertexSimple
            #pragma fragment LitPassFragmentSimple
            #define BUMP_SCALE_NOT_SUPPORTED 1

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitGBufferPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "DepthOnly"
            Tags{"LightMode" = "DepthOnly"}

            ZWrite On
            ColorMask R
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            #pragma vertex DepthOnlyVertex
            #pragma fragment DepthOnlyFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/DepthOnlyPass.hlsl"
            ENDHLSL
        }

        // This pass is used when drawing to a _CameraNormalsTexture texture
        Pass
        {
            Name "DepthNormals"
            Tags{"LightMode" = "DepthNormals"}

            ZWrite On
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            #pragma vertex DepthNormalsVertex
            #pragma fragment DepthNormalsFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitDepthNormalsPass.hlsl"
            ENDHLSL
        }

        // This pass it not used during regular rendering, only for lightmap baking.
        Pass
        {
            Name "Meta"
            Tags{ "LightMode" = "Meta" }

            Cull Off

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            #pragma vertex UniversalVertexMeta
            #pragma fragment UniversalFragmentMetaSimple
            #pragma shader_feature EDITOR_VISUALIZATION

            #pragma shader_feature_local_fragment _EMISSION
            #pragma shader_feature_local_fragment _SPECGLOSSMAP

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitMetaPass.hlsl"

            ENDHLSL
        }

        Pass
        {
            Name "Universal2D"
            Tags{ "LightMode" = "Universal2D" }
            Tags{ "RenderType" = "Transparent" "Queue" = "Transparent" }

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            #pragma vertex vert
            #pragma fragment frag
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _ALPHAPREMULTIPLY_ON

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/Utils/Universal2D.hlsl"
            ENDHLSL
        }
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "SimpleLit" "IgnoreProjector" = "True" "ShaderModel"="2.0"}
        LOD 300

        Pass
        {
            Name "ForwardLit"
            Tags { "LightMode" = "UniversalForward" }

            // Use same blending / depth states as Standard shader
            Blend[_SrcBlend][_DstBlend], [_SrcBlendAlpha][_DstBlendAlpha]
            ZWrite[_ZWrite]
            Cull[_Cull]

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local_fragment _EMISSION
            #pragma shader_feature_local _RECEIVE_SHADOWS_OFF
            #pragma shader_feature_local_fragment _SURFACE_TYPE_TRANSPARENT
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _ _ALPHAPREMULTIPLY_ON _ALPHAMODULATE_ON
            #pragma shader_feature_local_fragment _ _SPECGLOSSMAP _SPECULAR_COLOR
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            // -------------------------------------
            // Universal Pipeline keywords
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            #pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
            #pragma multi_compile _ SHADOWS_SHADOWMASK
            #pragma multi_compile_fragment _ _SHADOWS_SOFT
            #pragma multi_compile_fragment _ _ADDITIONAL_LIGHT_SHADOWS
            #pragma multi_compile_fragment _ _SCREEN_SPACE_OCCLUSION
            #pragma multi_compile_fragment _ _LIGHT_LAYERS
            #pragma multi_compile_fragment _ _LIGHT_COOKIES
            #pragma multi_compile _ _CLUSTERED_RENDERING


            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile _ DIRLIGHTMAP_COMBINED
            #pragma multi_compile _ LIGHTMAP_ON
            #pragma multi_compile _ DYNAMICLIGHTMAP_ON
            #pragma multi_compile_fog
            #pragma multi_compile_fragment _ DEBUG_DISPLAY

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #pragma vertex LitPassVertexSimple
            #pragma fragment LitPassFragmentSimple
            #define BUMP_SCALE_NOT_SUPPORTED 1

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitForwardPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            ColorMask 0
            Cull[_Cull]

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            // -------------------------------------
            // Universal Pipeline keywords

            // This is used during shadow map generation to differentiate between directional and punctual light shadows, as they use different formulas to apply Normal Bias
            #pragma multi_compile_vertex _ _CASTING_PUNCTUAL_LIGHT_SHADOW

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "DepthOnly"
            Tags{"LightMode" = "DepthOnly"}

            ZWrite On
            ColorMask R
            Cull[_Cull]

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            #pragma vertex DepthOnlyVertex
            #pragma fragment DepthOnlyFragment

            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/DepthOnlyPass.hlsl"
            ENDHLSL
        }

        // This pass is used when drawing to a _CameraNormalsTexture texture
        Pass
        {
            Name "DepthNormals"
            Tags{"LightMode" = "DepthNormals"}

            ZWrite On
            Cull[_Cull]

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            #pragma vertex DepthNormalsVertex
            #pragma fragment DepthNormalsFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitDepthNormalsPass.hlsl"
            ENDHLSL
        }

        // This pass it not used during regular rendering, only for lightmap baking.
        Pass
        {
            Name "Meta"
            Tags{ "LightMode" =  "Meta" }

            Cull Off

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            #pragma vertex UniversalVertexMeta
            #pragma fragment UniversalFragmentMetaSimple

            #pragma shader_feature_local_fragment _EMISSION
            #pragma shader_feature_local_fragment _SPECGLOSSMAP
            #pragma shader_feature EDITOR_VISUALIZATION

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitMetaPass.hlsl"

            ENDHLSL
        }

        Pass
        {
            Name "Universal2D"
            Tags{ "LightMode" = "Universal2D" }
            Tags{ "RenderType" = "Transparent" "Queue" = "Transparent" }

            HLSLPROGRAM
            #pragma only_renderers gles gles3 glcore d3d11
            #pragma target 2.0

            #pragma vertex vert
            #pragma fragment frag
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _ALPHAPREMULTIPLY_ON

            #include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/Utils/Universal2D.hlsl"
            ENDHLSL
        }
    }
    Fallback  "Hidden/Universal Render Pipeline/FallbackError"
    CustomEditor "UnityEditor.Rendering.Universal.ShaderGUI.SimpleLitShader"
}

```

## SimpleLitInput.hlsl

```csharp
#ifndef UNIVERSAL_SIMPLE_LIT_INPUT_INCLUDED
#define UNIVERSAL_SIMPLE_LIT_INPUT_INCLUDED

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl"

CBUFFER_START(UnityPerMaterial)
    float4 _BaseMap_ST;
    half4 _BaseColor;
    half4 _SpecColor;
    half4 _EmissionColor;
    half _Cutoff;
    half _Surface;
CBUFFER_END

#ifdef UNITY_DOTS_INSTANCING_ENABLED
    UNITY_DOTS_INSTANCING_START(MaterialPropertyMetadata)
        UNITY_DOTS_INSTANCED_PROP(float4, _BaseColor)
        UNITY_DOTS_INSTANCED_PROP(float4, _SpecColor)
        UNITY_DOTS_INSTANCED_PROP(float4, _EmissionColor)
        UNITY_DOTS_INSTANCED_PROP(float , _Cutoff)
        UNITY_DOTS_INSTANCED_PROP(float , _Surface)
    UNITY_DOTS_INSTANCING_END(MaterialPropertyMetadata)

    #define _BaseColor          UNITY_ACCESS_DOTS_INSTANCED_PROP_WITH_DEFAULT(float4 , _BaseColor)
    #define _SpecColor          UNITY_ACCESS_DOTS_INSTANCED_PROP_WITH_DEFAULT(float4 , _SpecColor)
    #define _EmissionColor      UNITY_ACCESS_DOTS_INSTANCED_PROP_WITH_DEFAULT(float4 , _EmissionColor)
    #define _Cutoff             UNITY_ACCESS_DOTS_INSTANCED_PROP_WITH_DEFAULT(float  , _Cutoff)
    #define _Surface            UNITY_ACCESS_DOTS_INSTANCED_PROP_WITH_DEFAULT(float  , _Surface)
#endif

TEXTURE2D(_SpecGlossMap);       SAMPLER(sampler_SpecGlossMap);

// 根据不同的关键字，从高光贴图中采样出高光颜色或者直接使用材质定义的高光色。注意这儿的_GLOSSINESS_FROM_BASE_ALPHA,前面alpha test那儿说过，这个关键字表示base贴图的alpha存储的是glossiness,否则就使用高光贴图的alpha作为glossiness使用。这儿使用exp2解码smoothness，说明贴图中存储的是log空间。
half4 SampleSpecularSmoothness(float2 uv, half alpha, half4 specColor, TEXTURE2D_PARAM(specMap, sampler_specMap))
{
    half4 specularSmoothness = half4(0, 0, 0, 1);
#ifdef _SPECGLOSSMAP
    specularSmoothness = SAMPLE_TEXTURE2D(specMap, sampler_specMap, uv) * specColor;
#elif defined(_SPECULAR_COLOR)
    specularSmoothness = specColor;
#endif

#ifdef _GLOSSINESS_FROM_BASE_ALPHA
    specularSmoothness.a = alpha;
#endif

    return specularSmoothness;
}

inline void InitializeSimpleLitSurfaceData(float2 uv, out SurfaceData outSurfaceData)
{
    outSurfaceData = (SurfaceData)0;

    // 采样贴图函数SampleAlbedoAlpha，就是使用SAMPLE_TEXTURE2D采样一张2D贴图，采样出来的结果是一个half4。可以理解为这个half4包含了rgb反射率(albedo)和alpha，即不是简单的将贴图中的texel看成是一个颜色
    half4 albedoAlpha = SampleAlbedoAlpha(uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap));
    // 而diffuseAlpha.a和_BaseColor.a相乘得到最终的alpha
    outSurfaceData.alpha = albedoAlpha.a * _BaseColor.a;
    // 使用AlphaDiscard函数进行Alpha Test。clip函数当接受的参数值小于0时丢弃这个片段。这儿的offset一般都不会设置，就是0，可以忽略.
    // 而alpha和cutoff比较，如果alpha < cutoff则片段没通过alpha test被丢弃。所以cutoff设置的是可通过alpha test的最小的alpha值，大于或等于cutoff的alpha值是通过alpha test的
    // void AlphaDiscard(real alpha, real cutoff, real offset = 0.0h)
    // {
    //     #ifdef _ALPHATEST_ON
    //         clip(alpha - cutoff + offset);
    //     #endif
    // }
    AlphaDiscard(outSurfaceData.alpha, _Cutoff);
  
    // diffuseAlpha.rgb和_BaseColor.rgb相乘作为最终的diffuse系数（其实仍然是反射率）
    outSurfaceData.albedo = albedoAlpha.rgb * _BaseColor.rgb;
    outSurfaceData.albedo = AlphaModulate(outSurfaceData.albedo, outSurfaceData.alpha);

    // 采样高光贴图以及获取smoothness。根据不同的关键字，从高光贴图中采样出高光颜色或者直接使用材质定义的高光色。注意这儿的_GLOSSINESS_FROM_BASE_ALPHA,前面alpha test那儿说过，这个关键字表示base贴图的alpha存储的是glossiness,否则就使用高光贴图的alpha作为glossiness使用。这儿使用exp2解码smoothness，说明贴图中存储的是log空间。
    // half4 SampleSpecularSmoothness(half2 uv, half alpha, half4 specColor, TEXTURE2D_PARAM(specMap, sampler_specMap))
    // {
    //     half4 specularSmoothness = half4(0.0h, 0.0h, 0.0h, 1.0h);
    // #ifdef _SPECGLOSSMAP
    //     specularSmoothness = SAMPLE_TEXTURE2D(specMap, sampler_specMap, uv) * specColor;
    // #elif defined(_SPECULAR_COLOR)
    //     specularSmoothness = specColor;
    // #endif

    // #ifdef _GLOSSINESS_FROM_BASE_ALPHA
    //     specularSmoothness.a = exp2(10 * alpha + 1);
    // #else
    //     specularSmoothness.a = exp2(10 * specularSmoothness.a + 1);
    // #endif

    //     return specularSmoothness;
    // }
    half4 specularSmoothness = SampleSpecularSmoothness(uv, outSurfaceData.alpha, _SpecColor, TEXTURE2D_ARGS(_SpecGlossMap, sampler_SpecGlossMap));
    outSurfaceData.metallic = 0.0; // unused
    outSurfaceData.specular = specularSmoothness.rgb;
    outSurfaceData.smoothness = specularSmoothness.a;
    // 采样法线贴图
    // 如果定义了相应的关键字，则采样法线贴图，且根据是否使用了bump scale，使用不同的解码函数。这两个unpcak函数在SRP Core的Packing.hlsl中，涉及到法线贴图相关的编码方式，这儿不细说，也许会有一篇单独分析法线贴图会说。一般来说，法线贴图会保存切线空间的法线，因此这儿的变量名为normalTS
    // half3 SampleNormal(float2 uv, TEXTURE2D_PARAM(bumpMap, sampler_bumpMap), half scale = 1.0h)
    // {
    // #ifdef _NORMALMAP
    //     half4 n = SAMPLE_TEXTURE2D(bumpMap, sampler_bumpMap, uv);
    //     #if BUMP_SCALE_NOT_SUPPORTED
    //         return UnpackNormal(n);
    //     #else
    //         return UnpackNormalScale(n, scale);
    //     #endif
    // #else
    //     return half3(0.0h, 0.0h, 1.0h);
    // #endif
    // }
    outSurfaceData.normalTS = SampleNormal(uv, TEXTURE2D_ARGS(_BumpMap, sampler_BumpMap));
    outSurfaceData.occlusion = 1.0;
    // 采样自发光贴图, 采样一张自发光贴图再乘以一个自发光颜色
    // half3 SampleEmission(float2 uv, half3 emissionColor, TEXTURE2D_PARAM(emissionMap, sampler_emissionMap))
    // {
    // #ifndef _EMISSION
    //     return 0;
    // #else
    //     return SAMPLE_TEXTURE2D(emissionMap, sampler_emissionMap, uv).rgb * emissionColor;
    // #endif
    // }
    outSurfaceData.emission = SampleEmission(uv, _EmissionColor.rgb, TEXTURE2D_ARGS(_EmissionMap, sampler_EmissionMap));
}

#endif

```

## SimpleLitForwardPass.hlsl

```csharp
#ifndef UNIVERSAL_SIMPLE_LIT_PASS_INCLUDED
#define UNIVERSAL_SIMPLE_LIT_PASS_INCLUDED

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

struct Attributes
{
    // 坐标
    float4 positionOS    : POSITION;
    // 法线
    float3 normalOS      : NORMAL;
    // 切线
    float4 tangentOS     : TANGENT;
    float2 texcoord      : TEXCOORD0;
    // 静态和动态uv
    float2 staticLightmapUV    : TEXCOORD1;
    float2 dynamicLightmapUV    : TEXCOORD2;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    // uv
    float2 uv                       : TEXCOORD0;

    // 世界坐标
    float3 positionWS                  : TEXCOORD1;    // xyz: posWS

    // 法线和视角方向
    #ifdef _NORMALMAP
        half4 normalWS                 : TEXCOORD2;    // xyz: normal, w: viewDir.x
        // 如启动了法线贴图，则需要输出切线坐标系
        half4 tangentWS                : TEXCOORD3;    // xyz: tangent, w: viewDir.y
        half4 bitangentWS              : TEXCOORD4;    // xyz: bitangent, w: viewDir.z
    #else
        half3  normalWS                : TEXCOORD2;
    #endif

    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        half4 fogFactorAndVertexLight  : TEXCOORD5; // x: fogFactor, yzw: vertex light
    #else
        half  fogFactor                 : TEXCOORD5;
    #endif

    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        float4 shadowCoord             : TEXCOORD6;
    #endif

    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 7);

#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD8; // Dynamic lightmap UVs
#endif

    float4 positionCS                  : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};

void InitializeInputData(Varyings input, half3 normalTS, out InputData inputData)
{
    inputData = (InputData)0;

    // positionWS直接从Varyings input得到
    inputData.positionWS = input.positionWS;

    // viewDirWS和normalWS则是根据是否使用法线贴图分别设置
    #ifdef _NORMALMAP
        // 如果使用了法线贴图, viewDirWS是存放在NTB三个向量的w中的。而normalWS是将法线贴图中存放的切线空间法线，也就是传入的normalTS,从切线空间变换到世界空间得到。
        half3 viewDirWS = half3(input.normalWS.w, input.tangentWS.w, input.bitangentWS.w);
        // 这样构造出来的矩阵的3个行分别是TBN三个向量，而关于空间变换矩阵有一个公共性质就是矩阵中包含了变换后的坐标轴，比如在切线空间中，三个坐标轴TBN分别是x,y,z轴单位向量，通过切线空间到世界空间的变换矩阵会分别变换成世界空间的TBN，也就是我们这儿的3个参数，那么以这三个世界空间向量组成的矩阵就是从切线空间变换到世界空间的矩阵。这儿3个向量是按行填入矩阵的，所以矩阵的3行就分别是3个轴，这样的矩阵如果想让任意向量从切线空间变换到世界空间，需要使用行向量在左乘矩阵的方式，这也正是TransformTangentToWorld
        inputData.tangentToWorld = half3x3(input.tangentWS.xyz, input.bitangentWS.xyz, input.normalWS.xyz);
        // real3 TransformTangentToWorld(real3 dirTS, real3x3 tangentToWorld)
        // {
        //     // Note matrix is in row major convention with left multiplication as it is build on the fly
        //     return mul(dirTS, tangentToWorld);
        // }
        inputData.normalWS = TransformTangentToWorld(normalTS, inputData.tangentToWorld);
    #else
        half3 viewDirWS = GetWorldSpaceNormalizeViewDir(inputData.positionWS);
        inputData.normalWS = input.normalWS;
    #endif

    // 计算好viewDirWS和normalWS后还要分别归一化，但是用了两个不同的函数，有什么区别呢？ normalWS使用的是NormalizeNormalPerPixel方法
    // real3 NormalizeNormalPerPixel(real3 normalWS)
    // {
    //     #if defined(SHADER_QUALITY_HIGH) || defined(_NORMALMAP)
    //         return normalize(normalWS);
    //     #else
    //         return normalWS;
    //     #endif
    // }
    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    // 根据是否使用高质量shader，或者是否使用了法线贴图，来决定是否执行归一化法线。 viewDirWS使用的是SafeNormalize方法
    // Normalize that account for vectors with zero length,所谓Safe，就是检查了一下向量的长度的平方，是否为0，如果小于FLT_MIN则取FLT_MIN,即最小的正32位浮点数。rsqrt这个hlsl函数计算平方根的倒数，和原向量相乘就做了归一化。这个函数可以避免向量长度过小引起的除0错误
    // real3 SafeNormalize(float3 inVec)  
    // {
    //     real dp3 = max(FLT_MIN, dot(inVec, inVec));
    //     return inVec * rsqrt(dp3);
    // }
    viewDirWS = SafeNormalize(viewDirWS);

    inputData.viewDirectionWS = viewDirWS;

    // 计算shadow coord，如果已经在顶点shader里面计算了直接从varying获取，或者如果定义了MAIN_LIGHT_CALCULATE_SHADOWS则使用TransformWorldToShadowCoord将世界空间位置变换到主光源空间，否则就是不使用阴影直接填充0。
    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        inputData.shadowCoord = input.shadowCoord;
    #elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
        inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
    #else
        inputData.shadowCoord = float4(0, 0, 0, 0);
    #endif
  
    // fogCoord和vertex lighting都是直接从varying获取，之前在VS里面计算好了
    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        inputData.fogCoord = InitializeInputDataFog(float4(inputData.positionWS, 1.0), input.fogFactorAndVertexLight.x);
        inputData.vertexLighting = input.fogFactorAndVertexLight.yzw;
    #else
        inputData.fogCoord = InitializeInputDataFog(float4(inputData.positionWS, 1.0), input.fogFactor);
        inputData.vertexLighting = half3(0, 0, 0);
    #endif

#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif

    // normalizedScreenSpaceUV 是归一化的屏幕空间UV，这是用于采样屏幕空间的uv坐标，比如SSAO会用到。
    inputData.normalizedScreenSpaceUV = GetNormalizedScreenSpaceUV(input.positionCS);
    // shadowMask是Mixed Lights的shadow Mask Lighting Mode相关的。在这种模式下面，会采样一张shadow mask贴图来获取物体上一个点最多和4盏灯的遮挡关系，以决定该点是否能被某个灯照射到。
    inputData.shadowMask = SAMPLE_SHADOWMASK(input.staticLightmapUV);

    #if defined(DEBUG_DISPLAY)
    #if defined(DYNAMICLIGHTMAP_ON)
    inputData.dynamicLightmapUV = input.dynamicLightmapUV.xy;
    #endif
    #if defined(LIGHTMAP_ON)
    inputData.staticLightmapUV = input.staticLightmapUV;
    #else
    inputData.vertexSH = input.vertexSH;
    #endif
    #endif
}

///////////////////////////////////////////////////////////////////////////////
//                  Vertex and Fragment functions                            //
///////////////////////////////////////////////////////////////////////////////

// Used in Standard (Simple Lighting) shader
Varyings LitPassVertexSimple(Attributes input)
{
    Varyings output = (Varyings)0;

    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
    // 输入模型空间的法线和切线
    // VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)
    // {
    //     VertexNormalInputs tbn;
    //     real sign = tangentOS.w * GetOddNegativeScale();             //这个符号是Object space切线的w和GetOddNegativeScale返回值的乘积
    //     tbn.normalWS = TransformObjectToWorldNormal(normalOS);       //计算世界空间的法线
    //     tbn.tangentWS = TransformObjectToWorldDir(tangentOS.xyz);    //计算世界空间的切线
    //     tbn.bitangentWS = cross(tbn.normalWS, tbn.tangentWS) * sign; //算副切线是使用法线和切线的叉积，但是其结果需要校正符号
    //     return tbn;
    // }
    // real GetOddNegativeScale()
    // {
    //     // FIXME: We should be able to just return unity_WorldTransformParams.w, but it is not
    //     // properly set at the moment, when doing ray-tracing; once this has been fixed in cpp,
    //     // we can revert back to the former implementation.
    //     return unity_WorldTransformParams.w >= 0.0 ? 1.0 : -1.0;
    // }
    VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

#if defined(_FOG_FRAGMENT)
        half fogFactor = 0;
#else
        // 其实就是简单的深度雾，根据当前坐标的z值在clip和far之间的比例（统一到[0,1]之间）使用线性插值或者指数函数计算雾的颜色值。
        half fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
#endif

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
    output.positionWS.xyz = vertexInput.positionWS;
    output.positionCS = vertexInput.positionCS;

#ifdef _NORMALMAP
    // 如果使用法线贴图，则会输出世界空间的法线，切线，和副切线。
    // 并且会把世界空间的视线方向夹带在这3个向量的w中，节省了一个输出向量.
    half3 viewDirWS = GetWorldSpaceViewDir(vertexInput.positionWS);
    output.normalWS = half4(normalInput.normalWS, viewDirWS.x);
    output.tangentWS = half4(normalInput.tangentWS, viewDirWS.y);
    output.bitangentWS = half4(normalInput.bitangentWS, viewDirWS.z);
#else
    // 如果不使用法线贴图，那么只要输出世界空间的法线和视线方向即可
    // 这个函数会根据shader quality以及是否使用法线贴图来决定是否要归一化法线
    // real3 NormalizeNormalPerVertex(real3 normalWS)
    // {
    //     #if defined(SHADER_QUALITY_LOW) && defined(_NORMALMAP)
    //         return normalWS;
    //     #else
    //         return normalize(normalWS);
    //     #endif
    // }
    output.normalWS = NormalizeNormalPerVertex(normalInput.normalWS);
#endif

    // 输出lightmap UV和SH
    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
    OUTPUT_SH(output.normalWS.xyz, output.vertexSH);

    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        // 顶点光照，用VertexLighting函数，输入世界空间的位置和法线，计算逐顶点的光照颜色
        // half3 VertexLighting(float3 positionWS, half3 normalWS)
        // {
        //     half3 vertexLightColor = half3(0.0, 0.0, 0.0);
        // #ifdef _ADDITIONAL_LIGHTS_VERTEX
        //     uint lightsCount = GetAdditionalLightsCount();
        //     for (uint lightIndex = 0u; lightIndex < lightsCount; ++lightIndex)
        //     {
        //         Light light = GetAdditionalLight(lightIndex, positionWS);
        //         half3 lightColor = light.color * light.distanceAttenuation;
        //         vertexLightColor += LightingLambert(lightColor, light.direction, normalWS);   //使用LightingLambert函数计算出一个漫反射光照颜色
        //     }
        // #endif
        //     return vertexLightColor;
        // }
        // half3 LightingLambert(half3 lightColor, half3 lightDir, half3 normal)
        // {
        //     half NdotL = saturate(dot(normal, lightDir));
        //     return lightColor * NdotL;
        // }
        // 从上可见：顶点光照计算的就是附加光的漫反射lambert颜色，所有附加光都计算然后叠加到一起。
        half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);
        output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
    #else
        output.fogFactor = fogFactor;
    #endif

    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        output.shadowCoord = GetShadowCoord(vertexInput);
    #endif

    return output;
}

// Used for StandardSimpleLighting shader
half4 LitPassFragmentSimple(Varyings input) : SV_Target
{
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);

    SurfaceData surfaceData;
    InitializeSimpleLitSurfaceData(input.uv, surfaceData);

    InputData inputData;
    InitializeInputData(input, surfaceData.normalTS, inputData);
    SETUP_DEBUG_TEXTURE_DATA(inputData, input.uv, _BaseMap);

#ifdef _DBUFFER
    ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
#endif
    // 计算光照的核心函数就是UniversalFragmentBlinnPhong，传入的参数除了InputData，还有之前计算出来的diffuse, specular, smoothness, emission和alpha。之后使用MixFog将光照计算出来的颜色和雾的颜色进行混合，最后使用OutputAlpha计算最终的alpha值
    // alf4 UniversalFragmentBlinnPhong(InputData inputData, half3 diffuse, half4 specularGloss, half smoothness, half3 emission, half alpha)
    // {  
    //     // To ensure backward compatibility we have to avoid using shadowMask input, as it is not present in older shaders
    // #if defined(SHADOWS_SHADOWMASK) && defined(LIGHTMAP_ON)
    //     half4 shadowMask = inputData.shadowMask;
    // #elif !defined (LIGHTMAP_ON)
    //     half4 shadowMask = unity_ProbesOcclusion;
    // #else
    //     half4 shadowMask = half4(1, 1, 1, 1);
    // #endif
    half4 color = UniversalFragmentBlinnPhong(inputData, surfaceData);
    color.rgb = MixFog(color.rgb, inputData.fogCoord);
    // alpha还是那个alpha,只不过如果表面类型是不透明，就直接设置为1
    // half OutputAlpha(half outputAlpha, half surfaceType = 0.0)
    // {
    //     return surfaceType == 1 ? outputAlpha : 1.0;
    // }
    color.a = OutputAlpha(color.a, _Surface);

    return color;
}

#endif

```

## ShadowCasterPass.hlsl

```csharp
#ifndef UNIVERSAL_SHADOW_CASTER_PASS_INCLUDED
#define UNIVERSAL_SHADOW_CASTER_PASS_INCLUDED

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"

// Shadow Casting Light geometric parameters. These variables are used when applying the shadow Normal Bias and are set by UnityEngine.Rendering.Universal.ShadowUtils.SetupShadowCasterConstantBuffer in com.unity.render-pipelines.universal/Runtime/ShadowUtils.cs
// For Directional lights, _LightDirection is used when applying shadow Normal Bias.
// For Spot lights and Point lights, _LightPosition is used to compute the actual light direction because it is different at each shadow caster geometry vertex.
float3 _LightDirection;
float3 _LightPosition;

struct Attributes
{
    float4 positionOS   : POSITION;
    float3 normalOS     : NORMAL;
    float2 texcoord     : TEXCOORD0;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float2 uv           : TEXCOORD0;
    float4 positionCS   : SV_POSITION;
};

float4 GetShadowPositionHClip(Attributes input)
{
    float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
    float3 normalWS = TransformObjectToWorldNormal(input.normalOS);

#if _CASTING_PUNCTUAL_LIGHT_SHADOW
    float3 lightDirectionWS = normalize(_LightPosition - positionWS);
#else
    float3 lightDirectionWS = _LightDirection;
#endif

    float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, lightDirectionWS));

#if UNITY_REVERSED_Z
    positionCS.z = min(positionCS.z, UNITY_NEAR_CLIP_VALUE);
#else
    positionCS.z = max(positionCS.z, UNITY_NEAR_CLIP_VALUE);
#endif

    return positionCS;
}

Varyings ShadowPassVertex(Attributes input)
{
    Varyings output;
    UNITY_SETUP_INSTANCE_ID(input);

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
    output.positionCS = GetShadowPositionHClip(input);
    return output;
}

half4 ShadowPassFragment(Varyings input) : SV_TARGET
{
    Alpha(SampleAlbedoAlpha(input.uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap)).a, _BaseColor, _Cutoff);
    return 0;
}

#endif

```
