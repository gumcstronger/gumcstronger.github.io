---
layout:     post
title:      "Unity光照笔记"
subtitle:   " \"Global Illumination\""
date:       2023-02-01 20:00:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
---
GI = 直接光照 （主要方向光，主要灯光）+ 间接光照（弹射光照，非重要灯光）+ 环境光照 + 反射光（区域反射，环境反射）。

## 直接光照 Directional illumination

Unity的Lighting - Environment下：

天空球的设置会影响到Environment Lighting（环境光照）的颜色和EnvironmentReflections（环境反射）的信息。

## 间接光照 Indirect illumination

### Light mapping

### Light  Probe

### RSM-based solutions

### Voxel-based solutions

### Ray / Path tracing（Off-line?）

### Photon mapping（Off-line?）

## Ambient Occlusion

## Reflection / Refraction / Scattering

## Shadow
