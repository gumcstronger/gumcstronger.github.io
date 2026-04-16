---
layout:     post
title:      "AI Code助手配置"
subtitle:   "Claude Code / Codex"
date:       2026-04-09 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - AI
---
## Gemini

### 安装

* Gemini Code Assist(VSCode插件)

只要开启Agent就会显示：There was a problem getting a response.猜测是免费用户会被限制在 Flash 模型中，而Flash 用不来 Agent。

```json
{
  // vscode 配置
  "geminicodeassist.enableTelemetry": false,
  "geminicodeassist.chat.changeView":"Default diff view",
  "geminicodeassist.inlineSuggestions.enableAuto": false,
  // "geminicodeassist.project": "xxxx",            // free 无效
  "geminicodeassist.agentYoloMode": true,             // 开启 Yolo 模式，自动执行, 不要停下来请求权限
}
```

* Gemini Cli （npm安装, 开启Agent）

### Skill && MCP

* 通过CC Switch 安装 superpowers（brainstorming、dispatching-parallel-agents、using-superpowers、writing-plans、executing-plans）
* 通过CC Switch 安装[vscode-mcp](https://github.com/tjx666/vscode-mcp)

```json
{
    "vscode-mcp": {
      "command": "npx",
      "args": ["-y", "@vscode-mcp/vscode-mcp-server@latest"],
      "env": {},
      "includeTools": [
        "get_symbol_lsp_info",
        "get_diagnostics",
        "get_references",
        "health_check",
        "rename_symbol"
      ]
    }
}
```

* 通过CC Switch 安装[excel-master](https://github.com/guillehr2/Excel-MCP-Server-Master)

```json

  {
      "excel-master": {
        "command": "npx",
        "args": [
          "-y",
          "@guillehr2/excel-mcp-server@latest"
        ],
        "timeout": 60000
      }
  }

```

* [所有配置链接](https://geminicli.com/docs/reference/configuration/)

### clibot(for Discord/Wechat)

* 安装[Go](https://go.dev/dl/)
* 安装clibot

```bash
git clone https://github.com/keepmind9/clibot.git
cd clibot
go build -o clibot.exe ./cmd/clibot
# 复制config.yaml
cp configs/config.mini.yaml configs\config.yaml
# 配置具体的信息
# 可以启动
clibot serve --config configs\config.yaml
# 设置开机启动
# 第一步：打开启动文件夹,按下键盘上的 Win + R 键。在弹出的对话框中输入 shell:startup 并按回车。这会打开一个名为“启动”的文件夹。
# 第二步：创建快捷方式,在文件夹空白处点击 右键 -> 新建 -> 快捷方式。在“请键入对象的位置”框中，直接复制并粘贴以下完整命令：(如没有安装pwsh则使用powershell.exe)
pwsh.exe -NoExit -Command "cd 'D:\Pack\AI\clibot'; clibot serve --config  .\configs\config.yaml"
=# 具体命令
slist                              # 列出所有会话
suse <session>                     # 切换到指定会话
snew <name> <type> <dir> [cmd]     # 创建新会话（仅管理员）
sdel <name>                        # 删除会话（仅管理员）
sclose [name]                      # 关闭会话
sstatus [name]                     # 显示会话状态
whoami                             # 显示你的信息
status                             # 显示所有会话状态
echo                               # 显示你的 IM 信息
help                               # 显示帮助
```

## Codex

### 安装

* Codex桌面版
* VSCode Codex插件

vscode codex插件如果开启WSL就会使用WSL的配置，需要进入/home/xx/.condex中修改配置，所以不开启WSL

### Agents

### MCP

* [VSCode MCP](https://github.com/tjx666/vscode-mcp)

不支持partial class，但能获取诊断信息，用于获取诊断信息并修复。

MCP（模型上下文协议）客户端能够实时访问丰富的 VSCode 上下文信息

```bash
# vscode安装vsc-lsp-mcp插件
# .condex/config.toml添加配置：
[mcp_servers.vscode-mcp]
command = "bunx"
args = ["-y", "@vscode-mcp/vscode-mcp-server@latest"]
env = { "VSCODE_MCP_DISABLED_TOOLS" = "health_check,list_workspaces,open_files" }
startup_timeout_ms = 16_000
```

---

## AI助手通用必备

### [CC Switch](https://github.com/farion1231/cc-switch)

用于：

1. 切换第三方API
2. 添加管理MCP服务器
3. 添加管理Skill

### [jq](https://jqlang.org/download/)

下载后添加到PATH

### [rtk](https://github.com/rtk-ai/rtk)

降低token

### [Ripgrep](https://github.com/burntsushi/ripgrep)

似乎AI助手的grep底层使用的都是ripgrep。不确定是否真实，所以也安装ripgrep方便使用

---

## Claude Code

备注：为什么弃用，因为对其他AI的支持很弱，anthropic的AI像Claude Sonne才能用起来顺畅。

### 安装

* 安装[Claude Code](https://github.com/anthropics/claude-code)

  备注：npm安装方式已弃用

```bash
  # 管理员权限运行powershell
  irm https://claude.ai/install.ps1 | iex

  # 配置关闭claude code遥感
  # 永久生效：以下似乎无效，下方vscode settings.json的配置才有效
  # [Environment]::SetEnvironmentVariable("CLAUDE_TELEMETRY", "off", "User")
  # [Environment]::SetEnvironmentVariable("CLAUDE_NO_TELEMETRY", "1", "User")
  # [Environment]::SetEnvironmentVariable("ANTHROPIC_NO_TELEMETRY", "1", "User")
  # [System.Environment]::SetEnvironmentVariable('DISABLE_TELEMETRY', '1', 'User')
```

* 安装[Claude Code for VS Code](vscode:extension/anthropic.claude-code)

  VS Code插件

```json
# VSCode的Setting.json必须显性禁用所有非核心功能的网络请求，包括遥测上报和自动更新检查。不然会一直等待遥感失败，导致等到几分钟才能进入AI的请求。
"claudeCode.environmentVariables": [
        {
            "name": "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC",
            "value": "1"
        },
        {
            "name": "DISABLE_TELEMETRY",
            "value": "1"
        },
    ],
```

### CALUDE.md

### MCP

* [ripgrep](https://github.com/burntsushi/ripgrep)

据说claude code底层使用的是ripgrep。不确定claude code内部是否包含了ripgrep，直接安装ripgrep方便使用

```bash
# 安装
winget install BurntSushi.ripgrep.MSVC
```

* [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)

```bash
# 安装
irm https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/scripts/setup-windows.ps1 | iex
# 启用 MCP 会话启动时的自动索引
codebase-memory-mcp config set auto_index true
# 保持最新状态
codebase-memory-mcp update
# 告诉claude测试codebase-memory-mcp的使用会进行项目索引
```

* (弃用)[~~Superpowers~~](https://github.com/obra/superpowers)
* (弃用)~~claude-md-management~~
* (弃用)~~planning-with-files~~
* (弃用)~~Remember~~
* (弃用)LSP

  放弃LSP方案，因为C# LSP无法正确处理partial。
* (弃用) LSP- [VSC-LSP-MCP](https://github.com/beixiyo/vsc-lsp-mcp)

使用VSCode LSP的MCP，MCP（模型上下文协议）客户端能够实时访问丰富的 VSCode 上下文信息

```md
  1. vscode安装vsc-lsp-mcp插件
  2. 通过CC Switch给Claude添加MCP：
  "lsp-mcp": {
    "type": "http",
    "url": "http://127.0.0.1:9527/mcp"
  },
  3. .claude/rules/TOOLS.LSP.md要求Claude Code生成对VSCode-MCP的使用。
```

* [LSP-VSCode MCP](https://github.com/tjx666/vscode-mcp)

不支持partial class，但能获取诊断信息，用于获取诊断信息并修复。

MCP（模型上下文协议）客户端能够实时访问丰富的 VSCode 上下文信息

```md
  1. vscode安装vsc-lsp-mcp插件
  2. 通过CC Switch给Claude添加MCP：
  "vscode-mcp": {
    "args": [
      "/c",
      "npx",
      "-y",
      "@vscode-mcp/vscode-mcp-server@latest"
    ],
    "command": "cmd",
    "type": "stdio"
  }
  3. .claude/rules/TOOLS.md要求Claude Code生成对VSCode-MCP的使用。
```

* [弃用]LSP-Claude Code官方CSharp-lsp

dotnet安装[csharp-ls](https://github.com/razzmatazz/csharp-language-server)
Claude Code VS Code插件市场安装csharp-ls Plugin

* (弃用)LSP-[VSCode LSP MCP Server](https://marketplace.visualstudio.com/items?itemName=trademe.vscode-lsp-mcp)(作者：Trad Me)

这个最简单，安装vscode插件。
运行VSCode命令: "LSP MCP: Install for Claude Code"
需要运行

```bash
# 确保mcp-proxy安装
uv tool install mcp-proxy
# 确保加入PATH，可能会输出Executable directory C:\Users\Gumc\.local\bin is already in PATH
uv tool update-shell
```

```json
{
  "mcpServers": {
    "vscode-lsp": {
      "type": "http",
      "url": "http://localhost:37140/mcp"
    }
  }
}
```

* (弃用)LSP-roslyn-refactor

感觉很慢

### Hook

## Antigravity

复杂功能特别是需要分析现有的代码进行实现，则使用Antigravity。

## VSCode插件

* Copilot

Copilot Pro无限使用GPT-5 Mini是不错的，可惜只有首月免费。

* Codex插件

某鱼某淘可购买business或Plus也有25元，可能有风险。
免费额度也很慷慨。

* Gemini Code Assist

复杂问题使用，Gemini 3的思考和代码能力最强。当然Geimin Code Assist经常会自动切换到Gemini Pro 2.5会降智的。
一般遇到复杂功能、不确定如何实现的需求或找Bug，则在aistudio使用Gemini 3讨论。如需要与代码交互(如找Bug)则使用Gemini Code Assist或Antigravity。

* Trae

基本废了，基本作为补全使用，不会用来写代码。
trae使用梯子通过trae.ai登录海外账号，似乎可以无限使用Gemini 2.5,不过现在经常出错，似乎海外账号不支持vscode插件了。
目前使用起来很慢，估计很快就可以弃用了。

* Code Web Chat

相当于合法通过vscode将上下文发送到web页面，然后获取web页面结果返回到vscode，自动进行editor等操作。
相比于很多逆向API(违规），这个合规的，自动帮忙提交上下文到网页版后并自动获取结果来对比。
但是，"codeWebChat.reuseLastTab": true,这个配置似乎不生效，每次都重开一个标签，有毛病。

备注：与Code Web Chat类似的有个：[openlink](https://github.com/afumu/openlink)([视频](https://www.bilibili.com/video/BV17Yw3z7EJd/?spm_id_from=333.1391.0.0&vd_source=f355063fe070b37905b1cec42ccf5c6c))，但还需要自己解决gemini外的前端适配和skill。

* Claude Code

除非用Claude官方模型，不然第三方其他AI支持很差。
