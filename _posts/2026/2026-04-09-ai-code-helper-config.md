---
layout:     post
title:      "AI Code助手配置"
subtitle:   "Claude Code / Codex"
date:       2026-04-09 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    false
tags:
    - AI
---
## AI Coding 必备

### [PowerShell 7](https://learn.microsoft.com/zh-cn/powershell/scripting/install/install-powershell-on-windows)

### [cockpit-tools](https://github.com/jlcodes99/cockpit-tools)

通用 AI IDE 账号管理工具，用于切换不同账号Auth

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

似乎AI助手的grep底层使用的都是ripgrep。但如果我们配置了通过vscode进行shell绕过权限，这时候需要支持rg所以这里手动安装。

```bash
winget install BurntSushi.ripgrep.MSVC
```

### cc-connect(for Discord/Wechat等)

  * 安装

```bash
# 安装cc-connect（wechat需要使用beta版本cc-connect@beta)
npm install -g cc-connect
# 创建配置文件
# 创建C:\Users\Gumc\.cc-connect
# 将官方的config.example.toml复制到.cc-connect/config.toml


```

  * [cc-connect配置文件]

```conf
  # 配置config.toml，以下是我的配置(从discord获取token)

  data_dir = ""
  language = "zh"

  [[projects]]
    name = "framework"
    [projects.agent]
      type = "gemini"
      [projects.agent.options]
        mode = "yolo"
        model = "gemini-3.1-flash-lite-preview"
        work_dir = "C:\\Users\\Gumc\\Desktop\\WorkSpace\\framework"

    [[projects.platforms]]
      type = "discord"
      [projects.platforms.options]
        token = "token"

  [log]
    level = "info"

  [speech]
    enabled = false
    provider = ""
    language = ""
    [speech.openai]
      api_key = ""
      base_url = ""
      model = ""
    [speech.groq]
      api_key = ""
      model = ""
    [speech.qwen]
      api_key = ""
      base_url = ""
      model = ""

  [display]
  thinking_messages = true # Show/hide thinking messages (default: true) / 是否显示思考消息（默认 true）
  thinking_max_len = 1000   # Max chars for thinking messages (default: 300) / 思考消息最大字符数（默认 300）
  tool_max_len = 1000       # Max chars for tool use messages (default: 500) / 工具调用消息最大字符数（默认 500）
  tool_messages = true     # Show/hide tool progress messages (default: true) / 是否显示工具进度消息（默认 true）

  [stream_preview]
  enabled = true            # Enable/disable streaming preview (default: true) / 启用/禁用流式预览（默认 true）
  interval_ms = 1500        # Min ms between updates (default: 1500) / 更新最小间隔毫秒数（默认 1500）
  min_delta_chars = 30      # Min new chars before sending update (default: 30) / 发送更新前最少新增字符数（默认 30）
  max_chars = 2000          # Max preview length (default: 2000) / 预览最大长度（默认 2000）


  [rate_limit]
  max_messages = 5         # Max messages per window; 0 = disabled (default: 20) / 窗口内最大消息数；0 = 禁用（默认 20）
  # window_secs = 60          # Window size in seconds (default: 60) / 窗口时间秒数（默认 60）

  [cron]
```

  * [cc-connect使用指南](https://github.com/chenhg5/cc-connect/blob/main/docs/usage.zh-CN.md)
  * 配置开机启动

---

## Gemini

### 安装

  * Gemini Code Assist(VSCode插件) 用处不大，仅在头脑风暴时进行。

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

  * Gemini Cli （npm安装）
  * [所有配置链接](https://geminicli.com/docs/reference/configuration/)

### Skill & MCP

  * superpowers

  复制superpowers的skill到项目.agents/skills下（brainstorming、dispatching-parallel-agents、executing-plans、receiving-code-review、requesting-code-review、subagent-driven-development、using-superpowers、writing-plans、writing-skills）

<!-- * (弃用) 通过CC Switch 安装[vscode-mcp](https://github.com/tjx666/vscode-mcp)

```
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
``` -->

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

  * 通过CC Switch 安装[vscode-mcp-servr](https://marketplace.visualstudio.com/items?itemName=JuehangQin.vscode-mcp-server)

```json

  {
    "vscode-mcp-server": {
        "command": "npx",
        "args": ["mcp-remote@next", "http://localhost:3000/mcp"]
    }
  }

```

  * [Unity MCP](https://github.com/CoplayDev/unity-mcp)

```bash
# unity通过github安装：https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main
# 进入unity - window - mcp for unity - toggle mcp window
# 进入connect，选择传递方式为stdio, 复制configuratioon到项目目录配置文件，工具只开启execute_menu_item和read_console和manage_prefabs
# 可以禁用手机数据
"unityMCP": {
      "command": "uvx",
      "args": [
        "--from",
        "mcpforunityserver==9.6.6",
        "mcp-for-unity",
        "--transport",
        "stdio"
      ],
      "type": "stdio",
      "env": {
        "DISABLE_TELEMETRY": "true"
      }
    }
# 让AI自己写markdown,工具的参数见源码：https://github.com/CoplayDev/unity-mcp/tree/beta/Server/src/services/tools

```

  * [unity-mcp-server](https://github.com/AnkleBreaker-Studio/unity-mcp-server)
  * [semble](https://github.com/MinishLab/semble)
  * [code-index-mcp](https://github.com/johnhuang316/code-index-mcp/)

  * [AgentLint](https://github.com/0xmariowu/AgentLint)
  * [poormansadvisor](https://github.com/leighstillard/poormansadvisor)
  * [prompt-improver](https://github.com/ndpvt-web/prompt-improver)

<!-- ### (已弃用)clibot(for Discord/Wechat)

消息会被截断，所以已弃用

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
``` -->

## Codex

### 安装

  * Codex Cli
  * ~~Codex桌面版~~
  * ~~VSCode Codex插件~~

vscode codex插件如果开启WSL就会使用WSL的配置，需要进入/home/xx/.condex中修改配置，所以不开启WSL

  * [codex全部配置](https://developers.openai.com/codex/config-sample)

### Skill & MCP

  同Gemini Cli

<!-- * [VSCode MCP](https://github.com/tjx666/vscode-mcp)

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
``` -->

---

## Claude Code

### 安装

  * 安装[Claude Code](https://github.com/anthropics/claude-code)

```bash
  # 管理员权限运行powershell (备注：旧的npm安装方式已弃用)
  irm https://claude.ai/install.ps1 | iex
```

  * ~~安装[Claude Code for VS Code](vscode:extension/anthropic.claude-code)~~

  ~~VS Code插件~~

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

### Skill & MCP

  同Gemini Cli

<!-- * [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)

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

感觉很慢 -->

### OpenCode

## Antigravity

复杂功能特别是需要分析现有的代码且使用Gemini 3 Pro时，则使用Antigravity。

## VSCode插件(暂时不使用)

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

## AI思考

1. 经常只改动他知道的，不理解上下文和改动的代码的涉及意图：
    例如：原本代码逻辑是A, AI加了个逻辑if判断后跳转到B，后面觉得B并不合适需要去掉，AI会基于B的判断去改为判断后走A逻辑。而不是按原来那样，直接就走A逻辑了。
    例如：Tween动画，需要调用OnComplete，原本的逻辑是加了个包装器在OnComplete后也进行RemoveTween。但如果不需要RemoveTween,AI只会删掉RemoveTween，而不会理解到包装器也是为了OnComplete而存在的。需要把包装器也删掉。
    我加了文档，不确定最后是否会按文档来，慢慢等待测试情况。

2. ✅ AI的大部分时间花费在找代码,要通过read去读取。想想我们自己是如何理解代码的，1个是整套代码框架有基本理解。2是vscode直接搜索能很快匹配大部分代码信息。但AI这个过程会很慢，特别遇到高峰期基本30分钟干不了人10分钟的事情。
    测试<https://github.com/MinishLab/semble> 和 <https://github.com/johnhuang316/code-index-mcp>, 结合用感觉还可以，似乎理解变快，但经常回退到Grep。

3. cli无法自动匹配任务去使用模型，例如简单的模型和复杂的模型
    skill可以设定模型。
      子代理模式，但目前似乎还没发挥子代理的优势，因为spec和plan还是使用haiku，而且superpowers作用不大，头脑风暴似乎没什么意义。

4. 花很多时间在决策和修正AI，AI的方案总是不理想，需要给它提供正确的信息和方案。所以所有时间都在给AI做决策和修正Ai的错误方案上。
      我觉得有几个问题，1是弱AI会有很大问题，得不断加无穷无尽的限制，还是不够聪明。2.即使聪明的，也还是会给出错误决策。就像重建新的Tutorial系统，说了旧系统会删掉，他还是用了旧系统的功能。说了需要跑完全部测试才能停下来，还是跑不完测的（是否没有测试的规矩）
      * ✅ 或者能否不要把AI当做一次性完成的工具，他就是新人，需要不断纠正。而你的工作就是让AI新人帮你干活。不要期待他在没有约束和纠正的情况下干好。你的工作就是约束和纠正AI新人。不要为此生气，气坏了没得赔。
      * ⬜️ Unity似乎没有正确的测试方法。

      * 我觉得应该Antigravity给计划，讲这个计划落盘。
        我审核通过后没问题再让gpt-5.4-mini进行落盘。

5. 没有免费AI
    想法1：使用Gemma 31B + ds2API测试效果解决5
    想法2：使用Gemma 31B + GPT5.4-mini合作检查方案并且一个执行一个审查看效果解决4
    目前使用 gpt-5.4-mini然后用coder-spark做审查，还行但是还是用不到coder-spark的能力，推理思考还是很弱。还可以使用gemma31b做审查。

    想法3: 使用类似<https://github.com/ypollak2/llm-router的功能来做匹配模型，解决3>
    想法4：RAG本地向量索引是否能优化2。解决2。trae是如何实现的？
    想法5： 如何解决1？C# .Net是否有好的方案。

6. AI还是不够聪明，我让他重新做一个系统，他确在做一个系统去引用旧系统，或者把旧系统作为入口引导到新系统。

备用：
改进Todo：
<https://github.com/lethain/library-mcp/tree/main>
以下两个不确定是否有作用：
<https://github.com/Horilla/claudectx>
<https://github.com/azkhh/cchubber> 可以作为开启启动
