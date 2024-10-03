---
layout:     post
title:      "[译]2024年在Mac上安装Ruby的最快、最简单的方法"
subtitle:   " \"2024年在Mac上安装Ruby的最快、最简单的方法\""
date:       2024-10-01 14:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Open Source
---
[原文地址](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#prerequisites)

## 先决条件

支持的 macOS 版本：

* Sonoma (macOS 14)
* Ventura (macOS 13)
* Monterey (macOS 12)

旧版本的 macOS 可能仍然有效，但我不能保证。

### 确保您的计算机已插入电源并具有稳定的互联网连接

按照本指南中的说明清理并准备 Mac 后，可能需要 15 分钟或更长时间才能安装所有内容，因此如果您计划在安装各种工具时执行其他操作，请暂时阻止 Mac 进入睡眠状态，直到安装完成。
确保您没有使用 VPN 或速度缓慢的共享公共 WiFi。如果您有电脑可以加入的移动热点，那么它可能比公共 WiFi 更好。您可以使用 fast.com 等服务比较速度。
另外，请确保您可以访问此页面：https://raw.githubusercontent.com/postmodern/ruby-versions/master/ruby/versions.txt
众所周知，一些互联网和移动服务提供商（如印度的 ACT）会阻止访问某些域名，例如上面来自 GitHub 的域名。

### 如果你在 Ventura 或更高级别

单击 Mac 左上角的 Apple 图标，然后单击“系统设置”。点击右侧的“节能”按钮，确保打开“当显示器关闭时防止自动在电源适配器上睡眠”

### 确保您的 macOS 软件是最新的

开始之前，请确保您拥有适用于当前 macOS 版本的最新 Apple 软件更新。点击屏幕左上角的 Apple 图标，然后点击“系统偏好设置”，然后点击“软件更新”。如果有任何可用更新，请安装它们。如果您已经拥有适用于当前 macOS 版本的最新更新，则无需升级到最新的 macOS。

### 确保 Homebrew 已准备好

如果您知道您尚未尝试安装 Homebrew，则可以跳过此部分。如果您不确定，/usr/local请在Terminal应用程序中运行以下命令来检查文件夹的内容：

```
ls /usr/local
```

如果文件夹中没有任何内容，则说明您没有 Homebrew。

如果您使用的是配备 Apple Silicon 芯片（M1/M2）的 Mac，您还需要检查该/opt/homebrew文件夹：

```
ls /opt/homebrew
```

如果这些文件夹不为空，则表示 Homebrew 已安装，您需要通过运行以下命令来确保 Homebrew 是最新的并且正常运行：

```
brew update && brew doctor
```

最后会显示：

```
Your system is ready to brew
```

如果尚未准备好，请阅读 Homebrew 的说明，看看您是否可以自行解决问题，或者在Homebrew 讨论的帮助下解决问题。Homebrew 通常会提供解决问题的详细说明。您也可以选择忽略错误和警告。在某些情况下，警告可能不会影响 Ruby 安装。本教程的其余部分可能仍然有效，但绝对不能保证。

如果您使用的是 Apple Silicon Mac，并且您的/usr/local文件夹不为空，则意味着您使用 Rosetta 或arch -x86_64终端安装了 Homebrew，或者您将文件从 Intel Mac 传输到了 Apple Silicon Mac。无论这种情况如何发生，它都可能导致各种问题，并且您可能会在遵循本指南时、在安装 gem 或运行项目时遇到bundle install错误pod install。

### 确保您没有安装 RVM、rbenv、asdf 或 frum

在所有 Ruby 管理器中，我推荐chruby，因为它最简单，而且是切换 Ruby 速度最快的管理器之一。Ruby 管理器彼此不兼容，因此如果您有 RVM、rbenv、asdf 或 frum，我建议卸载它们。

检查您是否有其他 Ruby 管理器

在终端中运行以下命令：

```
rvm help

rbenv help

asdf --help

frum versions
```

如果这些命令返回任何信息，则表示您已安装该工具。请在其网站/GitHub repo 上查找每个工具的卸载说明。

如果显示“未找到命令”，则您很可能没有安装它。但是，它也可能仍然安装，只是不在您的 中PATH。如果您不熟悉该术语PATH，我建议您阅读我的指南以[PATH](https://www.moncefbelyamani.com/troubleshooting-command-not-found-in-the-terminal/)了解它。它将教您更自信地解决开发设置问题。

### 确保您的 macOS 用户名和主文件夹名称中没有空格

在正常情况下，Apple 不会允许您创建名称中包含空格的 macOS 用户名，但我见过这种情况在某些罕见情况下发生。主文件夹名称中包含空格通常会导致各种问题。要解决此问题，请仔细遵循Apple 的这些说明。

### 确保您有足够的可用磁盘空间

Apple 的独立 Xcode 命令行工具占用大约 5GB 的磁盘空间，但安装程序可能需要更多的空间才能在您的硬盘上释放，大概是 15GB。一般来说，最好留出大约 10% 的总硬盘空间。

## 安装

### 仅当您使用 Apple Silicon Mac (M1/M2) 时才阅读此部分

如果您使用的是 Apple Silicon Mac，请确保终端不处于 Rosetta 模式。

终端打开后，您可以通过运行此命令来检查：

```
uname -m
```

它应该会显示arm64您是否使用的是 Apple Silicon Mac。如果它显示x86_64，则表示终端（或您正在使用的任何终端应用程序，例如 iTerm）处于 Rosetta 模式。关闭它的方法如下：

* 如果 Terminal/iTerm（或你正在使用的任何终端应用程序）正在运行，请退出
* 前往 Finder
* 查找您的终端应用程序。对于 Apple 终端应用程序，请按 转到“实用工具”文件夹shift-command-U（或从菜单栏中选择“前往”，然后选择“实用工具”）
* 单击终端应用程序一次以选择它，但不要启动它。
* 按下⌘-i键盘快捷键打开信息窗口（或从菜单栏中：“文件”，然后“获取信息”）。
* 取消选中“使用 Rosetta 打开”复选框
* 关闭终端信息窗口
* 启动终端/iTerm/等
* 运行uname -m。现在应该显示arm64，您可以继续阅读本指南的其余部分。
* 跑arch。应该说arm64。

如果uname -m或 都arch没有显示arm64，那么就是有其他东西在打开 Rosetta。检查下面的文件，看是否有arch -x86_64、i386、ARCHFLAGS或其他看起来像是在设置英特尔架构的行。然后删除它们，或将它们注释掉。

* ~/.zshrc
* 〜/ .zprofile
* ~/.zlogin
* 〜/ .bash_profile
* 〜/ .profile
* ~/.bashrc
* 〜/ .config/fish/config.fish

然后退出并重新启动终端应用程序并检查uname -m。

如果仍然显示“x86_64”，则您需要彻底清理开发环境并从头开始。彻底清理大约需要 60 个步骤，手动完成需要一个小时。或者如果您要擦除硬盘并重新安装 macOS，则需要几个小时。

### shell 上的注释

本教程假设您正在使用zsh。如果您不确定，请阅读[我的指南](https://www.moncefbelyamani.com/which-shell-am-i-using-how-can-i-switch/)以找出您正在使用的 shell.zshrc ，并将以下步骤中对 的任何引用替换为.bash_profile如果您使用的是 Bash。

### 步骤 1：安装 Homebrew 和命令行工具

Homebrew是“macOS 缺少的软件包管理器”，它可让您轻松安装数百种开源工具。完整的安装说明可在Homebrew 文档中找到，但您只需运行Homebrew 网站顶部列出的命令即可：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

请注意，Homebrew 网站上列出的命令可能会发生变化，因此请确保我上面列出的命令相同。如果不一致，请告诉我，我会更新。

将命令复制并粘贴到终端窗口中，按下return，然后阅读终端中显示的内容，并注意任何需要您输入的指令。例如，Homebrew 将提示您输入 macOS 密码。请注意，当您输入密码时，终端不会提供视觉反馈。只需慢慢输入并按回车键即可。

Homebrew 还会自动安装 Apple 命令行工具，并且通常在后台安装它们，但如果发生变化，请注意是否出现需要您输入的任何窗口。

（译者注：注意，安装成功后需要留意提示运行命令行添加到PATH）

安装成功后，退出并重新启动终端，然后检查 Homebrew 是否已准备就绪：

```
brew doctor
```

如果得到Your system is ready to brew，则可以继续执行步骤 2

在 Apple Silicon Mac 上，Homebrew 可能会告诉您在安装后运行一些命令，例如：

```
echo "eval $(/opt/homebrew/bin/brew shellenv)" >> ~/.zprofile
eval $(/opt/homebrew/bin/brew shellenv)
```

确保运行这些命令。

退出并重新启动终端，然后检查到目前为止一切是否正常：

```
brew doctor
```

### 步骤 2：使用 ruby-install 安装 chruby 和最新的 Ruby

安装chruby并ruby-install：

```
brew install chruby ruby-install
```

然后安装最新的 Ruby（目前为 3.3.5）：

```
ruby-install ruby
```

**在尝试安装其他 Ruby 版本之前，务必确保可以安装最新的 Ruby。**

这将需要几分钟的时间。如果成功，请转到下一步，配置您的 shell以自动使用chruby。

如果失败并出现!!! Compiling Ruby 3.3.5 failed!、或!!! Installation of ruby 3.3.5 failed!、!!! Configuration of ruby 3.3.5 failed!或或任何其他错误，那么很可能是您的开发设置出现了问题。

**如果 Ruby 安装失败怎么办**

当您拥有干净的开发设置时，最新的 Ruby 可以保证正常工作。如果无法安装最新的 Ruby，则您的系统中很可能存在干扰 Ruby 安装的因素。

不幸的是，有许多因素会影响 Mac 上的 Ruby 安装，例如：

* Xcode/命令行工具版本
* macOS 版本
* 英特尔与 Apple Silicon
* 同时使用和不使用 Rosetta 安装
* 将文件从 Intel Mac 传输到 Apple Silicon Mac
* 所需的 Ruby 版本
* 当前使用的 Ruby 版本
* 你如何安装 Ruby 和/或 gems，以及你使用的工具
* 缺少、配置错误和过时的开发工具
* 同时安装不兼容的工具

如果您正在安装最新的 Ruby，那么唯一能保证在所有情况下都有效的解决方案就是清理您的开发设置并从头开始。这可能涉及 60 个或更多步骤，可能需要 60 分钟才能完成，因此这超出了本教程的范围。

当然，肯定存在更快的解决方案，但这取决于您的具体情况。这就是为什么您可能会看到网上有人说某个解决方案对他们有用，但其他人却说到目前为止对他们没有任何效果。此外，大多数“解决方案”都是黑客和变通方法，可能会导致其他问题。

如果您尝试安装较旧的 Ruby 版本，那么即使您有干净的设置，也可能会失败，因为某些较旧的版本需要不同的设置和工具，具体取决于 Ruby 版本、Mac 型号、macOS 版本和/或 Xcode/命令行工具版本。有关更多详细信息，请阅读下面有关安装其他 Ruby 版本的部分。

**配置 shell**

```
echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
echo "chruby ruby-3.3.5" >> ~/.zshrc
```

请注意，3.3.5上面的命令假定3.3.5是在步骤 2 开始时安装的版本。
退出并重新启动终端，然后检查一切是否正常：

```
ruby -v
```

它应该与您在步骤2中安装的版本相匹配。

### 步骤 3：安装 gem

恭喜！您现在有一个可以运行的 Ruby 开发环境。您现在应该能够安装 Rails、Jekyll、cocoapods、fastlane、bundler、compass 或过去几天或几个月一直尝试安装的任何 gem！

只要确保您不要使用 sudo 来安装任何 gem！

现在您已经有了合适的 Ruby 开发环境，您可以放心地安装 gems，gem install后面跟着 gem 的名称。

请注意，有些 gem 与 Apple Silicon (M1/M2) 不兼容。我建议用更现代的替代品替换它们，或者询问维护人员是否愿意更新它们。以下是几个例子：

therubyracer（阅读如何在 M1 或 M2 Mac 上安装（或删除）therubyracer）

grpc (此修复应该很快就会发布)

接下来，请务必阅读本指南的其余部分。这将为您省去很多困惑和麻烦。

### 如何安装不同版本的 Ruby 并在它们之间切换

要安装其他版本，请运行，`ruby-install`然后运行所需版本。但首先，您需要检查您拥有哪个版本的 Apple 命令行工具 (CLT) 或 Xcode：

```
brew config
```

查找底部以CLT:和 开头的行Xcode:。如果其中任何一个以14或更高版本开头，并且你尝试安装早于 3.1.3、3.0.5 或 2.7.7 的版本（但不是 2.6.x 或更早的版本），那么你需要像这样安装 Ruby：

```
ruby-install 3.3.5 -- --enable-shared
```

否则，如果您的 CLT/Xcode 版本低于 14，请正常安装 Ruby 而无需任何额外选项：

```
ruby-install 3.3.5
```

要切换到这个新安装的版本，请退出并重新启动终端，然后运行chruby该版本。例如：

```
chruby 3.3.5
```

chruby与其他 Ruby 版本管理器不同，您无法设置一个自动在任何地方可用的“全局”Ruby 版本。如果您按照本指南操作，则每次打开新的终端窗口或选项卡时，chruby都会切换到 Ruby 3.3.5，或 shell 文件中最后显示的版本。然后，您应该能够使用 在该特定 Ruby 版本中全局安装 gem gem install name_of_gem。

但是，如果您随后cd进入没有文件的其他目录.ruby-version，或者文件设置为不知道的.ruby-versionRuby 版本，则会恢复为系统 Ruby（Mac 自带的 Ruby），即最新的 Monterey 及更高版本（Ventura、Sonoma）上的 2.6.10。chrubychruby

您绝对不会想使用系统 Ruby！以下是您不应该使用系统 Ruby 的 5 个理由。

您可以通过运行 进行检查which ruby。如果显示/usr/bin/ruby，则这是系统 Ruby，而您不希望出现这种情况。如果您在尝试安装 gem 或运行 时遇到错误bundle install，则首先应检查您是否使用了正确的 Ruby 版本。

这就是为什么我建议所有 Ruby 项目都有一个.ruby-version文件来指定 chruby 知道的版本之一。这样，当您cd进入项目时，chruby将自动检测该.ruby-version文件并切换到文件中定义的版本。

阅读下面的部分来了解如何设置和使用该.ruby-version文件。

### 自动切换到正确版本的推荐方法

强烈推荐的自动切换版本的方法是在你的 Ruby 项目中添加一个 `.ruby-version`具有所需版本号的文件，例如 `3.3.5`。要测试它是否有效：

1. 创建一个测试文件夹并 `cd`进入其中：

   ```
   cd ~
   mkdir testing-chruby
   cd testing-chruby
   ```
2. 创建一个名为的文件，其中 `.ruby-version`包含：`3.3.5`

   ```
   echo '3.3.5' >> .ruby-version
   ```

   假设您已经安装了 3.3.5。如果没有，请安装它，或将其替换 `3.2.5`为您已有的版本。您可以通过运行 来检查安装了哪些版本 `chruby`。
3. `cd`到项目之外的文件夹中，例如主文件夹：`cd ~`
4. 运行 `ruby -v`。 它可能会显示 `2.6.10`(或更旧)，这是 macOS 预装的 Ruby。
5. `cd -`返回到您的项目。这 `-`是在终端中返回上一个目录的快捷方式。
6. 验证 `ruby -v`显示 `3.3.5`
7. 删除测试文件夹：

   ```
   cd ~
   rm -rf testing-chruby
   ```

对于具有的项目 `Gemfile`，最好在中指定 Ruby 版本 `Gemfile`，因为它必须与文件中的版本相匹配 `.ruby-version`，所以您可以通过告诉 Gemfile 从文件中获取版本来轻松实现这一点 `.ruby-version`，方法是将此行放在 Gemfile 中的第一个 gem 之前：

```
ruby File.read(".ruby-version").strip
```

以下是 Gemfile 前几行的示例：

```
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby File.read(".ruby-version").strip

gem "rails", "~> 7.0.4"
```

### 关于使用旧 Ruby 版本的项目的重要说明

如果您尝试使用现有的 Ruby 项目，并且 `Gemfile`/或者 `.ruby-version`指定了早于 3.3.5、3.2.5、3.1.6、3.0.7 或 2.7.8 版本的文件，则您需要更新您的应用程序以使用较新的版本（最佳解决方案），或安装项目中指定的旧版本（不推荐）。

**这是使用 Ruby 时需要理解的一个非常重要的概念，许多人浪费时间尝试安装旧版本的 Ruby，而正确的解决方案是更新项目。**

例如，无法在 Apple Silicon Mac 上安装 Ruby 2.7.0 或 2.7.1（除非您使用 Rosetta，但我不推荐使用）。相反，您应该将项目至少更新到 2.7.8，然后更新到 3.1.6，因为 Ruby 2.7 将于 2023 年 3 月到期。请按照我的文章中的分步说明进行操作，其中解释了[如何以及为何升级项目中的 Ruby 版本](https://www.rubyonmac.dev/how-to-upgrade-the-ruby-version-in-your-project)。

### 现有项目的问题

现在您有了合适的 Ruby 开发环境，但它不一定能解决您项目特有的问题。例如，如果您 `bundle install`在旧的 Ruby 项目中运行后遇到错误，或者在尝试使用 运行 Jekyll 网站 `bundle exec jekyll serve`或使用 运行 Rails 应用程序时遇到错误 `bin/rails s`，这些错误可能是因为您项目中的 gem 和/或 Ruby 版本已过时。

在接下来的两节中，我将介绍如何判断问题是否特定于您的项目，以及故障排除技巧。

#### 如何判断问题是否特定于你的项目

检查您是否可以[创建并运行一个全新的 Rails 应用程序](https://www.moncefbelyamani.com/how-to-install-rails-and-create-a-new-rails-app-on-a-mac-the-easy-way/#install-rails)，和/或检查您是否可以[创建并运行一个全新的 Jekyll 网站](https://www.moncefbelyamani.com/how-to-install-jekyll-on-a-mac-the-easy-way/#install-jekyll)。

如果您能够使用上面链接中的说明生成并运行全新的 Rails 或 Jekyll 项目，则意味着您拥有一个有效的 Ruby 开发设置，并且您在现有项目中看到的问题特定于该项目。阅读下一部分以获取解决问题的提示。

如果您**无法**生成并运行全新的 Rails 或 Jekyll 项目，则表示您的 Ruby 设置有问题，您需要从本教程的开始重新开始。

#### 如何修复现有项目中的问题

我建议的第一件事是确保你所有的 gem 都是最新的。**过时的 gem 是最常见的错误来源。**

例如，`ffi`是 Apple Silicon Macs (M1/M2) 上的常见错误来源。如果您 `ffi`在错误消息中的任何位置看到，则很可能意味着您使用的是 1.15.2 之前的版本。

在这种情况下，请尝试更新 `ffi`到最新版本：

```
bundle update ffi
```

对错误消息中提到的任何其他宝石重复上述操作。

您可能还会发现本指南很有帮助：[如何更新 Gemfile 中的 Gems](https://www.moncefbelyamani.com/how-to-update-gems-in-your-gemfile/)。

#### 更新项目中的 Ruby 版本

我还建议将您的项目至少更新到 Ruby 2.7.8，然后更新到 3.1.6，因为 2.7.8 将在 2023 年 3 月终止使用。这是我编写的详细指南，解释了[如何以及为何升级项目中的 Ruby 版本](https://www.rubyonmac.dev/how-to-upgrade-the-ruby-version-in-your-project)。

### 关于不同 Ruby 版本的 gem 的说明

每个版本的 Ruby 中都会单独安装 Gems。例如，如果您在 3.1.6 中安装了 Jekyll 和/或 Rails，然后稍后安装了 Ruby 2.7.8，则必须在 Ruby 2.7.8 中再次安装它们。

在 Mac 上的 Ruby Ultimate 版本中，您可以指定在安装其他 Ruby 版本时要自动安装哪些 gem。只需在名为 的文本文件中添加每个 gem 的名称即可 `.default-gems`。

无论在哪个版本的 Ruby 中安装 gem，一定要使用 `gem install`后面跟 gem 的名称。切勿[使用 sudo 来安装 gems](https://www.moncefbelyamani.com/why-you-should-never-use-sudo-to-install-ruby-gems/)。

### 下一步

现在，您已具备在 Mac 上安装 gem 所需的最低要求，但您可能需要其他工具来配合 Jekyll 或 Rails 使用，例如 Postgres、Node、Redis 和 Yarn。Mac[上的 Ruby](https://www.rubyonmac.dev/pricing/?utm_campaign=install-ruby-guide)会安装和配置您立即开始使用所需的一切。

Mac 上的 Ruby 还可以通过简单地键入 来使您的开发环境保持最新状态 `rom`。软件包会定期更新新功能和安全修复，因此能够一次轻松更新所有工具非常重要。
