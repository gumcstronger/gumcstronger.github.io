---
layout    : post
title     : "Jekyll环境部署"
subtitle  : "Jekyll Environment"
date      : 2025-12-06 12:15:00
language  : zh-CN
author    : "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog   : true
tags      :
    - Game Development
---
## Windows配置Jekyll环境

(❌已弃用，Ruby新版本会有很多问题)

<!--

### 安装环境 (Ruby + DevKit)

1. **下载 RubyInstaller：**
   访问 [RubyInstaller for Windows](https://rubyinstaller.org/downloads/)。

   * 建议选择 **Ruby+Devkit 3.1.x (x64)** 或更高版本。
2. **安装并运行 `ridk install`：**

   - 安装到D: /ProgramFiles
   - 安装完成后，会自动弹出一个命令行窗口（RubyInstaller Setup Wizard）。

     - 输入 `3` (MSYS2 and MINGW development toolchain) 并按回车。
     - 这是为了确保 Windows 能编译 Jekyll 依赖的一些 C 语言插件。

   - 确保D: \ProgramFiles\\Ruby3xxx\bin放入PATH

### 安装 Jekyll 和 Bundler

打开 Windows 的 PowerShell 或终端（CMD），输入以下命令：

```powershell

gem install eventmachine -v '1.2.7' -- --with-cppflags = "-D_WIN32_WINNT=0x0600 -Wno-attributes"

gem install jekyll bundler

```

验证是否安装成功：

```powershell
jekyll -v
```

### 在本地构建（打包）你的网站

1. 使用终端进入你的网站源代码目录：

```powershell
   cd C: \你的项目路径\coohex-website
```

2. **安装依赖项：**
   如果你的项目里有 `Gemfile` ，先运行：

```powershell
   bundle install
```

3. **正式构建项目（生产环境模式）：**
   为了确保生成的链接正确且代码经过压缩，请使用以下命令构建：

```powershell
   # Windows 环境设置环境变量并构建
   $env:JEKYLL_ENV = "production"
   bundle exec jekyll build
```

   *构建完成后，你会发现项目根目录下多了一个 **`_site`** 文件夹。这个文件夹里的所有内容就是你需要传到服务器上的“包”。*

### 如果在 Windows 上遇到错误：

* **编码错误：** 在终端运行 `chcp 65001` 切换到 UTF-8 编码。
* **性能问题：** Jekyll 在 Windows 上处理大量文件时较慢，如果项目很大，建议在 Windows 上启用 **WSL (Windows Subsystem for Linux)**，在 Linux 子系统中运行 Jekyll，速度会快 5-10 倍。
-->

## MSYS2 安装jekyll环境

(❌已弃用，较为麻烦)

<!--
* ~~安装cmake~~
* ~~安装gcc~~

  + ~~[下载gcc压缩包](https://sourceforge.net/projects/mingw-w64/files/mingw-w64/mingw-w64-release/) 或 [这里](https://gcc-mcf.lhmouse.com/)~~
  + ~~解压后将mingw64/bin加入环境变量~~
  + ~~测试gcc是否安装成功：gcc -v~~
  + ~~将MinGW\bin中的mingw32-make.exe改名为make.exe~~
  + ~~测试make是否安装成功：make -v~~
* 安装Ruby(也可以通过Windows Subsystem for Linux安装)

  + [下载安装包](https://rubyinstaller.org/downloads/), 安装到D:/ProgramFiles
  + 确保D:\ProgramFiles\\Ruby33-X64\bin放入PATH
  + 检测Ruby是否安装成功：ruby -v
  + 安装完会自动弹窗(如果没有则运行ridk install)选择3 MSYS2 and MINGW development tool chain
  + ~~从Ruby3.0开始webrick已经不绑定到Ruby中，需要手动添加: bundle add webrick~~
* ~~Gem添加国内源
 `gem sources --remove https://rubygems.org/`

 `gem sources -a https://gems.ruby-china.com/`

  检测是否成功： `gem sources -l`

  已创建的jekyll项目可用把Gemfile中的链接也修改下~~
* ~~安装RubyGems(Ruby的包管理框架)~~

  + ~~`gem update --system`~~
  + ~~检测RubyGems是否安装成功：`gem -v`~~
  + ~~管理员权限cmd运行： `ridk install`，选择2和选择3 MSYS2 and MINGW development tool chain，弹窗出来选择路径安装。安装完继续等cmd的运行完成~~
  + ~~激活 `ridk enable`~~
  + ~~安装openssl~~
    ~~pacman -Syu
    pacman -S base-devel gcc mingw-w64-x86_64-toolchain
    pacman -S mingw-w64-x86_64-openssl~~

* MSYS2系统环境(打开start command prompt with ruby)

  + 配置国内镜像
    修改D:\ProgramFiles\Ruby33-x64\msys64\etc\pacman.d下的mirrorlist.msys文件增加
    Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch
    同时运行
    sed -i "s#https\?://mirror.msys2.org/#https://mirrors.tuna.tsinghua.edu.cn/msys2/#g" /etc/pacman.d/mirrorlist*

  + 更新Msys2系统环境
    pacman -Sy
    #更新源
    pacman -Syu
    pacman -Su

  + 安装gcc、cmake和其他常用工具
    pacman -S mingw-w64-x86_64-gcc
    pacman -S  mingw-w64-x86_64-toolchain
    pacman -S  base-devel ruby
    pacman -S ruby-ffi
    pacman -S mingw-w64-x86_64-gtk3
    pacman -S  mingw-w64-x86_64-glade
    pacman -S mingw-w64-x86_64-openssl
    pacman -S mingw-w64-x86_64-libffi
    pacman -S mingw-w64-ucrt-x86_64-pkg-config

  + 设置系统环境变量
    D:\ProgramFiles\Ruby33-x64\msys64\mingw64\bin

* 安装Jekyll和Bundler

  + 运行：`gem install jekyll bundler rdiscount`
  + ~~如果运行失败可以尝试使用rubyinstall压缩包，手动[安装MSYS2](https://www.cnblogs.com/CodeWorkerLiMing/p/12274583.html), 然后 `gem install jekyll --platform=ruby`~~
  + 检测Jekyll是否安装成功：`jekyll -v`
  + 如何缺少ssl则运行 `gem install openssl`
* 运行blog
-->

## WSL安装Jekyll环境（Linux则不必安装WSL2）

* 使用WSL2 安装Linux [安装见这里](https://gumcstronger.github.io/2025/04/29/holopart/)
* 安装Ruby [参考官方教程但使用rbenv](https://jekyllrb.com/docs/installation/#requirements)，但使用
  + 使用rbenv安装Ruby

```bash
  # 安装依赖
  sudo apt update
  sudo apt install build-essential libssl-dev libreadline-dev zlib1g-dev autoconf bison libyaml-dev libncurses5-dev libffi-dev libgdbm-dev
  # 安装 rbenv
  cd /data/pack
  # 下载并安装
  curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
  # =============================配置环境变量=============================
  # 将以下内容添加到你的 ~/.bashrc（如果是 zsh 则加到 ~/.zshrc）：
  echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
  echo 'eval "$(rbenv init -)"' >> ~/.bashrc
  source ~/.bashrc
  # 安装特定版本的 Ruby
  rbenv install 3.4.8       # 安装 3.4.8 版本
  rbenv global 3.4.8        # 设置为全局默认版本
  ruby -v                   # 验证
```

* 安装官方教程所需要求

```bash
  # 安装 Ruby 及其他必备组件（去掉了ruby-full)
  sudo apt-get install build-essential zlib1g-dev ruby-build ruby-dev
  # 避免以 root 用户身份安装 RubyGems 包（称为 gem）。而是为您的用户帐户设置一个 gem 安装目录。
  echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
  echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
  echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
  source ~/.bashrc
  # 到项目目录安装 Jekyll 和 Bundler
  cd 项目目录
  gem install jekyll bundler

```

* 参考官方教程安装Jekyll [官方教程](https://jekyllrb.com/docs/)

## Jekyll打包

* 运行blog

```bash
  # 如果是mac使用chruby,则需要切换ruby
  chruby 3.1.3
  # 运行
  bundle exec jekyll serve
  # 修改实时更新
  bundle exec jekyll serve --livereload
  # 打包
  bundle exec jekyll build
```

* 如需要多配置打包
  可以将以下保存到build.sh, 修改配置，直接运行./build.sh, 不用每次修改配置

```bash
#!/bin/bash

# 如果任何命令失败，则立即退出
set -e

# --- 定义源和目标目录 ---
# 构建输出目录 (在项目文件夹内)
SRC_COOHEX_COM="build/www_coohex_com"
SRC_COOHEX_GITHUB_IO="build/coohex_github_io"
SRC_DEVELOPMENT="build/development"

# 最终部署目录 (在项目文件夹外)
DEST_WWW_COOHEX_COM="../www_coohex_com"
DEST_COOHEX_GITHUB_IO="../coohex.github.io"


# --- Build Step ---
echo "--- 开始构建 Jekyll 网站 ---"

echo "1. 正在为 coohex.com 构建..."
bundle exec jekyll build --config _config_www_coohex_com.yml --destination "$SRC_COOHEX_COM"

# echo "2. 正在为 coohex.github.io 构建..."
bundle exec jekyll build --config _config_coohex_github_io.yml --destination "$SRC_COOHEX_GITHUB_IO"

# echo "3. 正在为 development 构建..."
# bundle exec jekyll build --config _config_development.yml --destination "$SRC_DEVELOPMENT"

echo "--- 所有构建已完成！ ---"


# # --- Deployment/Move Step ---
# echo "--- 开始部署文件 ---"

# # 1. 部署到 www_coohex_com
# echo "正在部署到 $DEST_WWW_COOHEX_COM..."
# # 确保目标目录存在
# mkdir -p "$DEST_WWW_COOHEX_COM"
# # 使用 rsync 同步文件：删除目标目录中多余的文件，但排除 .git 目录
# rsync -a --delete --exclude='.git' "$SRC_COOHEX_COM/" "$DEST_WWW_COOHEX_COM/"
# echo "部署到 $DEST_WWW_COOHEX_COM 完成。"

# # 2. 部署到 coohex.github.io
# echo "正在部署到 $DEST_COOHEX_GITHUB_IO..."
# # 确保目标目录存在
# mkdir -p "$DEST_COOHEX_GITHUB_IO"
# # 使用 rsync 同步文件：删除目标目录中多余的文件，但排除 .git 目录
# rsync -a --delete --exclude='.git' "$SRC_COOHEX_GITHUB_IO/" "$DEST_COOHEX_GITHUB_IO/"
# echo "部署到 $DEST_COOHEX_GITHUB_IO 完成。"

# echo "--- 所有文件部署完成！ ---"

```
