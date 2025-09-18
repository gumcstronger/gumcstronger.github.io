---
layout:     post
title:      "Unity连接Mumu查看Logcat"
subtitle:   "Unity show android logcat for Simulator Mumu"
date:       2025-09-17 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Game Development
---
## 查看mumu的端口

在mum - 设置中心 - ADB端口，默认为16384

## adb连接端口

 [官方参考](https://mumu.163.com/help/20230214/35047_1073151.html)

* 打开mumu自带的adb所在文件夹，默认在：C:\Program Files\Netease\MuMu\nx_device\12.0\shell, 右键打开powershell
* ，输入

  ```
  adb.exe connect 127.0.0.1:16384
  ```
* 查看Devices

  ```
  adb devices
  ```

  查看当前的设备
  可以看到有一行：`127.0.0.1:16384 device`
* shell

  ```
  adb -s 127.0.0.1:16384 shell
  ```

  实际上在目标安卓设备上打开一个它的原生命令行界面，并把这个界面‘投射’到当前的电脑终端上

## Unity

unity打包界面或Logcat界面，可以直接选择到127.0.0.1:16384的设备选项。

## 脚本

### windows

```
@echo off
title Connect to MuMu Emulator

:: =================================================================
:: 请根据你的实际情况修改下面的两个变量
:: =================================================================

:: 1. MuMu模拟器 adb.exe 所在的文件夹路径
set MUMU_ADB_PATH="C:\Program Files\Netease\MuMu\nx_device\12.0\shell"

:: 2. MuMu模拟器的连接地址 (通常不需要修改)
set DEVICE_ADDRESS=127.0.0.1:16384


:: =================================================================
:: 下面的脚本内容通常不需要修改
:: =================================================================
echo.
echo ======================== MuMu 连接脚本 ========================
echo.
echo  目标路径: %MUMU_ADB_PATH%
echo  设备地址: %DEVICE_ADDRESS%
echo.
echo ===============================================================
echo.

:: 切换到 MuMu 的 adb 目录
cd /d %MUMU_ADB_PATH%

:: 如果目录不存在，则报错退出
if not exist "adb.exe" (
    echo [错误!] 在指定路径中未找到 adb.exe。
    echo 请检查 MUMU_ADB_PATH 变量是否设置正确！
    echo.
    pause
    exit
)

echo 正在尝试连接模拟器...
echo.

:: 执行连接命令
.\adb.exe connect %DEVICE_ADDRESS%

echo.
echo ===============================================================
echo.
echo  脚本执行完毕。请查看上面的连接结果。
echo  如果Unity中仍未显示，请尝试重启ADB服务 (.\adb kill-server)
echo.

:: 暂停脚本，方便查看结果
pause
```

### Mac

```
#!/bin/bash

# =================================================================
# 请根据你的实际情况修改下面的两个变量
# =================================================================

# 1. MuMu模拟器 adb 所在的文件夹路径 (macOS 通常路径如下)
MUMU_ADB_PATH="/Applications/MuMuPlayer.app/Contents/MacOS/adb_tool"

# 2. MuMu模拟器的连接地址 (通常不需要修改)
DEVICE_ADDRESS="127.0.0.1:16384"


# =================================================================
# 下面的脚本内容通常不需要修改
# =================================================================

# 清屏
clear

# 定义颜色
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}======================== MuMu 连接脚本 ========================${NC}"
echo
echo -e "${YELLOW}目标路径:${NC} $MUMU_ADB_PATH"
echo -e "${YELLOW}设备地址:${NC} $DEVICE_ADDRESS"
echo
echo -e "${GREEN}================================================================${NC}"
echo

# 检查路径是否存在
if [ ! -d "$MUMU_ADB_PATH" ]; then
    echo -e "错误: 路径 '$MUMU_ADB_PATH' 不存在。"
    echo "请检查 MUMU_ADB_PATH 变量是否设置正确！"
    exit 1
fi

# 切换到 MuMu 的 adb 目录
cd "$MUMU_ADB_PATH"

# 检查 adb 是否存在
if [ ! -f "adb" ]; then
    echo -e "错误: 在指定路径中未找到 adb 程序。"
    exit 1
fi

echo "正在尝试连接模拟器..."
echo

# 执行连接命令
./adb connect "$DEVICE_ADDRESS"

echo
echo -e "${GREEN}================================================================${NC}"
echo
echo "脚本执行完毕。请查看上面的连接结果。"
echo

# 暂停脚本
read -p "按 Enter 键退出..."
```
