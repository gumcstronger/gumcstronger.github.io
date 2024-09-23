---
layout:     post
title:      "C#游戏双端框架(未完待续)"
subtitle:   " \"客户端使用Unity、服务端C#­\""
date:       2022-08-08 18:15:00
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog: true
tags:
    - Server
---
> 看了ET和Geek两个双端框架，ET和Geek都并不太符合要求。Geek更用于轻度些的，这里是一些注释笔记

## Geek

### 目录

- Bedrock.Framework     该库似乎还是Alpha
- GeekServer.App
  - Config
  - Logic
    - Bag
      - BagComp.cs    背包数据,需要存储到DB所以是继承StateComponent
    - Common
    - Login
      - LoginComp.cs  登录数据,需要查询DB所以继承QueryComponent
    - Role
      - RoleComp      玩家基础数据,继承StateComponent
    - Server
      - ServerComp.cs 服务器数据
    - GameLoop.cs       服务器的循环?
- GeekServer.CodeGenerator  代码生成工具
  - Agent
  - Sate
  - Template
  - Utils
- GeekServer.Core
  - Actor
  - Callback
  - Component
  - Entity
  - Events
  - Hotfix
  - Net
    - Http
      - HttpTool.cs  使用System.Net.Http.HttpClient能够同时在客户端与服务端同时使用的 HTTP 组件（比如处理 HTTP 标头和消息）， 为客户端和服务端提供一致的编程模型。主要用来给服务器错误关闭前发送通知。
    - GameServer.cs
  - Serialize
  - Storage
  - Timer
  - Utils
- GeekServer.Generate
- GeekServer.Hotfix
- GeekServer.Proto
- GeekServer.Serialize
- GeekServer.Test
- GeekServer.xUnit
- Tools
  - ExcelGen
  - Geek.MsgPackTool
- UnityDemo/Assets/Scripts
  - Framework
    - Actor
    - Bedrock
    - Event
    - Net
    - Serialize
    - Utils
  - Generate
    - Config
    - Proto
  - Logic
  - MessagePack

### MessagePackTool

> 由于C#的MessagePack通过运行时动态生成IL来序列化自定义对象，从而为每种类型创建自定义的、高度优化的格式化程序。但Unity IL2CPP等严格AOT环境禁止生成运行时代码，所以MessagePackTool提供了一种提前运行代码生成器的方法。
> MessagePackTool核心在于增加项目通过配置读取输入输出配置,然后调用MessagePack提供的mpc工具生成

<!-- 
## ET和Geek对比
### 差异
- ET使用单线程多进程、Geek使用多线程

### 各自优点
- ET
    - 多进程单物理机和多物理机没区别，天然适合分布式。

- Geek
    - 接口的检测处理、客户端和服务端共用的数据模块是双端必须要有的功能
    - 注释很清晰,容易理解

### 各自缺点（个人想法）
- ET
    - 目录、命名及代码分布比较难理解，有些代码也耦合性强。(强迫症看完受不了)
    - 注释几乎没有，且命名糟糕，导致无法理解开发者的意图,例如Model和Hotfix到底谁是数据谁是逻辑。
    - Client和Server混在一起，其实只有数据和逻辑有必要共用，其他的代码应该分开才会清晰。既然Generate的数据都生成两份了，为何不直接把代码生成两份。

- Geek
    - 貌似是多线程，不利于分布式，且多线程时经常会涉及线程问题会将简单问题复杂化。
    - 引用了Bedrock Framework库，看Nuget目前还是Alpha版本。（实际上如果客户端不使用的话应该可以移除）

### 共有缺点
- 目录结构不合理，公司以前使用CocosCreator+Nodejs实现的客户端服务端双端框架目录比较清晰：
    - Client             客户端代码
        - Client         客户端通用代码
        - ClientEx       客户端当前项目代码
        - Common         CS共用通用代码
        - CommonEx       CS共用当前项目代码
    - Server             服务端代码
        - Server         服务端通用代码
        - ServerEx       服务端共用当前项目代码
        - Common         CS共用通用代码
        - CommonEx       CS共用当前项目代码
- 是否需要增加数据缓存后定时入库缓解来缓解MongoDB压力
- 如有像NodeJS一样单线程单进程代码会非常直观，只要手动开多个进程即可，不用处理多线程操作，不用理解为了异步而实现的代码。



## ET7 （看一遍目录就...）

### 问题？
    - 各种伪装成ComponentSystem的ComponentHelper,是为了避免将逻辑放在Component吗？


- 关系图 -->
