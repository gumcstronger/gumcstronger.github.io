---
layout:     post
title:      "Claude Code配置"
subtitle:   "Claude Code"
date:       2026-04-09 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - AI
---
## 安装

### 安装[Claude Code](https://github.com/anthropics/claude-code)

  备注：npm安装方式已弃用

```bash
  # 管理员权限运行powershell
  irm https://claude.ai/install.ps1 | iex

  # 配置关闭claude code遥感
  # 永久生效：
  [Environment]::SetEnvironmentVariable("CLAUDE_TELEMETRY", "off", "User")
```

### 安装[Claude Code for VS Code](vscode:extension/anthropic.claude-code)

  VS Code插件

### 安装[CC Switch](https://github.com/farion1231/cc-switch)

用于：

1. 切换第三方API
2. 添加管理MCP服务器
3. 添加管理Skill

## CALUDE.md

## MCP

### [Superpowers](https://github.com/obra/superpowers)

### claude-md-management

### ~~planning-with-files~~

### Remember

### LSP

每个 MCP 工具都需要用详尽的 JSON Schema 来描述。一个 LSP MCP 服务器哪怕只暴露几个功能，每次新会话都会产生数千个 Token 的 Schema 开销。每次调用工具，Claude 发出的指令和 MCP 服务器返回的结果，都会被详细记录在对话历史中，并随着对话反复被模型处理，消耗大量的动态 Token。要看下官方的LSP插件是否会数据更少。

#### (弃用)VSC-LSP-MCP

使用VSCode LSP的MCP，MCP（模型上下文协议）客户端能够实时访问丰富的 VSCode 上下文信息

```md
  1. vscode安装vsc-lsp-mcp插件
  2. 通过CC Switch给Claude添加MCP：
  "lsp-mcp": {
    "type": "http",
    "url": "http://127.0.0.1:9527/mcp"
  },
  3. .claude/rules/TOOLS.md要求Claude Code生成对VSCode-MCP的使用。
```

#### (弃用)[VSCode MCP](https://github.com/tjx666/vscode-mcp)

已弃用，因为不支持partial class。

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

#### [弃用]Claude Code官方CSharp-lsp

dotnet安装[csharp-ls](https://github.com/razzmatazz/csharp-language-server)
Claude Code VS Code插件市场安装csharp-ls Plugin

#### [VSCode LSP MCP Server](https://marketplace.visualstudio.com/items?itemName=trademe.vscode-lsp-mcp)(作者：Trad Me)

这个最简单，安装vscode插件。
运行VSCode命令: "LSP MCP: Install for Claude Code"
备注：生成的.mcp.json是错误的，配置应该是用http

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

## Hook

## ThirdParty

### [jq](https://jqlang.org/download/)

下载后添加到PATH

### [rtk](https://github.com/rtk-ai/rtk)

降低token
