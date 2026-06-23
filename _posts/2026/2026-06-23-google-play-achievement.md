---
layout:     post
title:      "Google Play 成就导入失败的坑"
subtitle:   "Google Play Achievement Import Failed"
date:       2026-06-23 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - Game Development
---

最近为公司通用游戏框架增加成就的功能，想着更新到旧游戏中进行猜测。
无奈，无论如何导入，GooglePlay后台都会提示：

```
语言区域不受支持 - 请使用游戏支持的语言区域。请修改以下成就的语言区域值：xxx
```

检查测试了多次，最开始猜测是官方文档的语言代码与后台语言代码不同，全部改为后台配置时显示的语言代码，例如ru改为ru-RU。还是错误。
即使将语言减少为只有一种zh-CN一种也不行。

在最后发现坑呀，虽然游戏详情中设置设置了多种语言，但实际上导入成就时，是根据Play Service中配置的语言的。果断删了AchievementsLocalizations.csv中的所有多语言，成功导入。

嗯，坑。
