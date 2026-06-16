---
layout:     post
title:      "Github Desktop无法打开"
subtitle:   "Github Desktop not open or not work"
date:       2026-06-16 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - System
---

前几天突然无法打开Github Desktop了，查了下有人是安装了Antigravity也无法打开了。真是坑呀谷歌，又垃圾还搞坏环境。

  - 使用powershell查询：

```powershell
# 输入
icacls $env:LOCALAPPDATA
# 应该输出
C:\Users\xxxx\AppData\Local NT AUTHORITY\SYSTEM:(OI)(CI)(F)
                            BUILTIN\Administrators:(OI)(CI)(F)
                            xxxx\xxx:(OI)(CI)(F)
                            xxxx\CodexSandboxUsers:(OI)(CI)(RX)
#实际输出
C:\Users\xxxx\AppData\Local xxxx\CodexSandboxUsers:(OI)(CI)(RX)
                            S-1-15-2-4283406991-4294444083-2861539546-3594594304-3831838268-676399724-3246962861:(F)
                            S-1-15-2-4283406991-4294444083-2861539546-3594594304-3831838268-676399724-3246962861:(OI)(CI)(IO)(F)
                            NT AUTHORITY\SYSTEM:(OI)(CI)(F)
                            BUILTIN\Administrators:(OI)(CI)(F)
                            xxxx\xxxx:(OI)(CI)(F)
# S-1-15-2-*这段就是Antigravity留下的坑了
```

  - 使用Powershell管理员模式运行

```powershell
# 定义路径和 SID
$path = $env:LOCALAPPDATA
$sidString = "S-1-15-2-4283406991-4294444083-2861539546-3594594304-3831838268-676399724-3246962861"

# 获取当前 ACL
$acl = Get-Acl $path

# 将 SID 字符串转换为 SecurityIdentifier 对象
$identity = [System.Security.Principal.SecurityIdentifier]::new($sidString)

# 找出所有匹配该 SID 的访问规则
$rulesToRemove = $acl.Access | Where-Object { $_.IdentityReference -eq $identity }

if ($rulesToRemove) {
    foreach ($rule in $rulesToRemove) {
        # 移除该规则
        $null = $acl.RemoveAccessRule($rule)
        Write-Host "移除了规则：$rule"
    }
    # 应用更改
    Set-Acl -Path $path -AclObject $acl
    Write-Host "权限已更新。"
} else {
    Write-Host "未找到匹配该 SID 的权限规则。"
}

# 验证结果
icacls $path
```
