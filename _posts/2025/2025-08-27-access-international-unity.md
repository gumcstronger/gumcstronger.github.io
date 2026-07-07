---
layout:     post
title:      "如何绕过Unity CN并访问国际版Unity Hub"
subtitle:   "How to Bypass Unity CN and Access International Unity Hub"
date:       2025-08-27 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
[官方参考](https://docs.unity3d.com/6000.2/Documentation/Manual/ent-proxy-cmd-file.html)
[论坛参考](https://discussions.unity.com/t/trouble-signing-in-i-am-redirected-to-id-unity-cn/875158/12)

仅供研究和教育！

## 查看代理端口

  * Clash用户：

```
打开Clash
查找 “端口” 或 “HTTP端口” 设置
常见Clash端口：7890、7891或1080
你的代理地址将是：http://127.0.0.1:[端口号]
```

  * Shadowsocks用户：

```
打开Shadowsocks
检查本地端口（通常是1080）
你的代理地址将是：http://127.0.0.1:1080
```

  * V2ray用户

```
打开v2rayN
设置-参数设置
本地混合监听端口(通常是10808)
你的代理地址将是：http://127.0.0.1:10808
```

## 查看Unity Hub安装路径

简单

## Windows用户

  * 创建以下脚本

```bat
@echo off

REM Unity Hub Proxy Launch Script
REM This script launches Unity Hub with proxy settings to bypass Unity CN

echo Starting Unity Hub with proxy settings...

REM Set proxy environment variables
REM IMPORTANT: Change these addresses to match YOUR proxy configuration
set HTTP_PROXY=http://127.0.0.1:10808
set HTTPS_PROXY=http://127.0.0.1:10808
REM Optional: Set additional proxy variables for completeness
set http_proxy=%HTTP_PROXY%
set https_proxy=%HTTPS_PROXY%

REM ========== UPM Cache Paths ==========
REM Set global cache root directory
set UPM_CACHE_ROOT=D:\Caches\Unity

REM Set specific cache paths (higher priority than UPM_CACHE_ROOT)
set UPM_NPM_CACHE_PATH=D:\Caches\Unity\NpmCache
set UPM_CACHE_PATH=D:\Caches\Unity\PackageCache
set ASSETSTORE_CACHE_PATH=D:\Caches\Unity\AssetStoreCache
set UPM_GIT_LFS_CACHE_PATH=D:\Caches\Unity\GitLFSCache

REM ========== Display Current Settings ==========
echo Proxy HTTP: %HTTP_PROXY%
echo Proxy HTTPS: %HTTPS_PROXY%
echo UPM_CACHE_ROOT: %UPM_CACHE_ROOT%
echo UPM_NPM_CACHE_PATH: %UPM_NPM_CACHE_PATH%
echo UPM_CACHE_PATH: %UPM_CACHE_PATH%
echo ASSETSTORE_CACHE_PATH: %ASSETSTORE_CACHE_PATH%
echo UPM_GIT_LFS_CACHE_PATH: %UPM_GIT_LFS_CACHE_PATH%

REM Launch Unity Hub
REM IMPORTANT: Change this path to YOUR Unity Hub location
echo Launching Unity Hub...
REM start "" "D:\ProgramFiles\Unity Hub\Unity Hub.exe"
powershell -Command "Start-Process 'D:\ProgramFiles\Unity Hub\Unity Hub.exe'"

REM Optional: Wait a moment before closing the command window
@REM timeout /t 3 /nobreak >nul

echo Unity Hub launched successfully!
echo You can now close this window or press any key to exit.
pause
```

  * 更改代理端口
  将端口10808该为你的代理端口号
  * 更改Unity Hub路径
  将D:\ProgramFiles\Unity Hub\Unity Hub.exe替换为你的Unity Hub路径
  * 保存脚本
  打开Windows的记事本，复制以上代码，修改后点击**文件**-**另存为**-**在"保存类型"下拉菜单中，选择 “所有文件 (.)”**-命名你的文件launch_unity_hub.cmd-保存到桌面（方便访问）
  * 双击运行launch_unity_hub.bat，就会自动打开Unity Hub
  * 如需要清除旧的Cache

```bat
rmdir /S /Q %LOCALAPPDATA%\Unity\cache
rmdir /S /Q %LOCALAPPDATA%\Unity\cache\packages
rmdir /S /Q "%APPDATA%\Unity\Asset Store-5.x"
```

## Mac用户

在终端运行以下代码生成launchUnityHub.command (注意：端口和Unity Hub路径记得修改)，

```bash
#!/usr/bin/env bash

# ========== Proxy Settings ==========
export HTTP_PROXY=http://127.0.0.1:10808
export HTTPS_PROXY=http://127.0.0.1:10808
export http_proxy=$HTTP_PROXY
export https_proxy=$HTTPS_PROXY

# ========== UPM Cache Paths (macOS) ==========
# 缓存根目录（可自定义，例如 ~/Library/Caches/Unity 即${HOME}/Library/Caches/Unity）
export UPM_CACHE_ROOT="/Volumes/HDD/Cache/Unity"

# 具体缓存子目录（若 UPM_CACHE_ROOT 已设置，Unity 会优先使用子目录变量）
export UPM_NPM_CACHE_PATH="${UPM_CACHE_ROOT}/NpmCache"
export UPM_CACHE_PATH="${UPM_CACHE_ROOT}/PackageCache"
export ASSETSTORE_CACHE_PATH="${UPM_CACHE_ROOT}/AssetStoreCache"
export UPM_GIT_LFS_CACHE_PATH="${UPM_CACHE_ROOT}/GitLFSCache"

# ========== (可选) 打印当前设置，便于调试 ==========
echo "Proxy HTTP: $HTTP_PROXY"
echo "Proxy HTTPS: $HTTPS_PROXY"
echo "UPM_CACHE_ROOT: $UPM_CACHE_ROOT"
echo "UPM_NPM_CACHE_PATH: $UPM_NPM_CACHE_PATH"
echo "UPM_CACHE_PATH: $UPM_CACHE_PATH"
echo "ASSETSTORE_CACHE_PATH: $ASSETSTORE_CACHE_PATH"
echo "UPM_GIT_LFS_CACHE_PATH: $UPM_GIT_LFS_CACHE_PATH"

# ========== Launch Unity Hub ==========
nohup "/Applications/Unity Hub.app/Contents/MacOS/Unity Hub" &>/dev/null &
```

  * 授权
  `chmod +x launchUnityHub.command`
  * 同样双击运行
  * 如需要清除旧的Cache:

```sh
rm -rf ~/Library/Unity/cache/packages
rm -rf ~/Library/Unity/cache
rm -rf ~/Library/Unity/Asset\ Store-5.x/
```
