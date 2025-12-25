---
layout:     post
title:      "定时关闭Win10实时保护"
subtitle:   "Disable Realtime Monitoring"
date:       2025-12-06 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
tags:
    - System
---
Win10的安全中心-实时保护默认会开启，即使关闭也会不断自动打开，实时保护会在任何代码编译生成时检查是否安全，导致编译前会一致等待实时保护的检查。对于开发来说时致命的，因为每次调试都会生成新的可执行程序，都要等待实时保护，严重降低开发速度。

所以需要写一个程序定时关闭实时保护。

1. 打开安全中心 - 病毒和威胁防护 - “病毒和威胁防护”设置 - 将“篡改防护”关闭。
2. 使用 PowerShell 一键创建任务(打开PowerShell时需要使用管理员权限打开)

```Powershell
# 1. 定义动作：调用 PowerShell 静默关闭实时防护
$Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-WindowStyle Hidden -Command `"Set-MpPreference -DisableRealtimeMonitoring `$true`""

# 2. 定义触发器：
# 先创建一个普通的触发器，设置间隔为1小时。
# 注意：这里我们先不指定 -RepetitionDuration，稍后手动修改为“无限”。
$Trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 1)

# 【关键修正】将重复持续时间设置为 $null，代表“无限期 (Indefinite)”
$Trigger.Repetition.Duration = $null

# 3. 定义权限：使用 SYSTEM 最高权限
$Principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount -RunLevel Highest

# 4. 定义额外设置：
# -StartWhenAvailable: 如果错过计划时间（如关机），开机后立即运行
# -AllowStartIfOnBatteries: 即使没插电源也运行
$Settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable -MultipleInstances Parallel

# 5. 注册（覆盖）任务
Register-ScheduledTask -TaskName "DefenderKiller_Hourly" -Action $Action -Trigger $Trigger -Principal $Principal -Settings $Settings -Force

Write-Host "任务已修正并创建：每隔 1 小时检查一次，永不过期。" -ForegroundColor Green
```

3 检查是否成功

```Powershell
执行完后，重启电脑，然后运行 Get-MpPreference | Select-Object DisableRealtimeMonitoring，如果显示 True，说明任务执行成功
```

4 如果需要关闭任务

```Powershell
Unregister-ScheduledTask -TaskName "AutoDisableDefender" -Confirm:$false
```
