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
> 看了ET和Geek两个双端框架，ET和Geek都并不太符合要求。猜测ET是用于mmo类的。这里是一些注释笔记。

## ET框架

### 问题

- Watcher进程在哪监听其他Server如失败就重启?

  - 没有找到
- X2X是什么?

  - 服务器名称
    - Manager : 对服务器进程进行管理
    - Realm : 登录服务器 ( 验证账号密码 相当于LoginServer 祖传叫法,你想叫什么随你)
    - Gate : 网关服务器
    - DB : 数据库服务器
    - Location : 位置服务器
    - Map : 地图服务器
    - Client : 客户端
    - All Server: 所有服务器集合
  - 消息命名( 端到端的命名方式 )
    - C2R_Ping = Client to Realm
    - R2G_GetLoginKey = Realm to Gate
    - M一般情况下代表map, 特殊代表Manager ,具体代表什么看消息协议内容
    - G2M_CreateUnit = Gate to Map
    - C2M_TestRequest = Client to Map
    - M2C_CreateUnits = Map to Client
    - M2A_Reload = Manager to AllServer 通知全部服务器热更新
  - 各服务器的作用(摘录自文档ET框架笔记):
    - Manager：连接客户端的外网和连接内部服务器的内网，对服务器进程进行管理，自动检测和启动服务器进程。加载有内网组件NetInnerComponent，外网组件NetOuterComponent，服务器进程管理组件。自动启动突然停止运行的服务器，保证此服务器管理的其它服务器崩溃后能及时自动启动运行。
    - Realm：对Actor消息进行管理（添加、移除、分发等），连接内网和外网，对内网服务器进程进行操作，随机分配Gate服务器地址。内网组件NetInnerComponent，外网组件NetOuterComponent，Gate服务器随机分发组件。客户端登录时连接的第一个服务器，也可称为登录服务器。
    - Gate：对玩家进行管理，对Actor消息进行管理（添加、移除、分发等），连接内网和外网，对内网服务器进程进行操作，随机分配Gate服务器地址，对Actor消息进程进行管理，对玩家ID登录后的Key进行管理。加载有玩家管理组件PlayerComponent，管理登陆时联网的Key组件GateSessionKeyComponent。
    - Location：连接内网，服务器进程状态集中管理（Actor消息IP管理服务器）。加载有内网组件NetInnerComponent，服务器消息处理状态存储组件LocationComponent。对客户端的登录信息进行验证和客户端登录后连接的服务器，登录后通过此服务器进行消息互动，也可称为验证服务器。
    - Map：连接内网，对ActorMessage消息进行管理（添加、移除、分发等），对场景内现在活动物体存储管理，对内网服务器进程进行操作，对Actor消息进程进行管理，对Actor消息进行管理（添加、移除、分发等），服务器帧率管理。服务器帧率管理组件ServerFrameComponent。
    - AllServer：将以上服务器功能集中合并成一个服务器。另外增加DB连接组件DBComponent
    - Benchmark：连接内网和测试服务器承受力。加载有内网组件NetInnerComponent，服务器承受力测试组件BenchmarkComponent。

### 目录

#### Codes

##### Hotfix

###### Client

- Demo
  - AI

    - AI_XunLuo.cs 继承于AAIHandler,AI巡逻
    - XunLuoPathComponent.cs AI巡逻
  - Login
  - Move

    - M2C_PathfindingResultHandler.cs   继承于AMHandler,消息事件
    - M2C_StopHandler.cs                继承于AMHandler消息事件
  - Ping

    - PingComponentSystem.cs
  - Router

    - RouterAddressComponentSystem.cs   进入游戏时，先获取AllRouter,然后随机分配一个Router。而且每十分钟更新一次
    - RouterHelper.cs
      - CreateRouterSession
        - GetRouterAddress
          - Connect
  - Scene

    - M2C_StartSceneChangeHandler.cs    继承于AMHandler,消息事件
  - Session

    - SessionStreamDispatcherClientOuter.cs 继承于IAction,是个CallBack
  - Unit

    - M2C_CreateMyUnitHandler/M2C_CreateUnitsHandler/M2C_RemoveUnitsHandler.cs 继承于AMHandler,消息事件
    - R2G_GetLoginKeyHandler            继承于AMActorRpcHandler,消息事件
- Module
  - Message
    - NetClientComponentSystem 使用NetService进行监听

###### Server

- Demo
  - Gate
    - C2G_EnterMapHandler/C2G_LoginGateHandler/C2G_PingHandler.cs 继承于AMRpcHandler,消息事件
  - Helper
  - Map
  - Realm
    - C2R_LoginHandler.cs Client客户端 to Realm登录服务器的消息,在收到客户端发来的C2R_LoginHandler消息以后，随机挑选一个Gate，让其加入。
  - Robot
  - Watcher
- Module
  - Actor
    - AMActorHandler 处理进程间通信send消息(不需要返回值)的继承AMActorHandler
    - AMActorRpcHandler 处理进程间通信rpc消息(需要返回值)的继承AMActorRpcHandler
  - ActorLocation
    - AMActorLocationHandler 与AMActorHandler类似，
    - AMActorLocationRpcHandler 与AMActorRpcHandler类似
  - AOI
  - Console
  - DB
  - Http
  - MessageInner
  - RobotCase
  - Router

###### Share

- Demo
  - InitShare.cs
- Module
  - AI
    - AIComponentSystem.cs
      - class AITimer
      - class AIComponentXXXSystem    Awake和Destroy行为
      - Check(this AIComponent self); 调用AAIHandler
      - Cancle(this AIComponent self);调用AAIHandler
  - Config
  - Message
  - Move
    - MoveComponentSystem.cs
  - Numeric
  - Recast
  - Scene
  - Unit

##### HotfixView

###### Client

- Demo
  - Camera
  - Global
  - Opera
  - Scene
  - UI
  - Unit
- Module
  - Resource
  - UI

##### Model

###### Client

- Demo
  - AI
  - Helper
  - Ping
    - PingComponent.cs
  - Router
  - Session
  - Unit

###### Generate          工具自动生成的代码

- Client
  - Config    工具根据策划配置数据自动生成的代码
  - Message   工具根据/Proto/OuterMessage.proto生成的
- Server
  - Config    工具根据策划配置+服务器配置数据自动生成的代码
  - ConfigPartial 为Config手写的处理
  - Message   工具根据/Proto下的所有Message生成的

###### Server

- Demo

  - Gate
    - PlayerComponent.cs 用于Gate网关服务器,保存玩家信息(Player:Account、UnitId)
    - GateSessionComponent.cs用于Gate网关服务器,保存所有Gate里的玩家的Session的Key
  - Map
  - Robot
  - Watcher
- Module

  - Actor
  - ActorLocation
    - ActorLocationSenderComponent.cs 用于Gate网关服务器,向Map内的指定玩家发送消息，如果发送失败，则向Location服务器索要新的地址。
    - LocationComponent.cs 用于Location地址服务器，保存了所有玩家的地址（Key是玩家的Id，Value是玩家的InstanceId），如果玩家在切换Map的时候，要把这里锁住。
  - AOI
  - Console
  - DB
  - Http
  - Message
    - AMRpcHandler 处理需要返回的请求集成AMRpcHandler
    - NetInnerComponent.cs Gate网关服务器,与Realm和Map服务器通讯,注意，Map并不与玩家直接通讯，全都由Gate转发。
  - RobotCase
  - Router

###### Share

- Module
  - Actor
    - ActorMessageSenderComponent.cs s用于Map常见服务器与Gate通讯。这里可以获得ActorId，而ActorId是找到对应Map的关键信息：IdGenerater.AppId。对于开房间的游戏来说，一个Map服务器可能会有很多个房间。
  - ActorLocation
  - AI
    - AIComponent.cs    客户端挂在ClientScene上，服务端挂在Unit上
  - Config
  - CoroutineLock
  - ETTask
  - Log
  - Message
    - AMHandler.cs	不需要返回的继承AMHandler
    - Session.cs Session.send/call后会调用AMRpchandler.Handler，然后通过NetService.SendMessage发送请求。而通过NetService.UpdateInMainThread接收返回值，然后调用NetClientComponent的OnRead，通知NetClientComponentOnRead，继续通知Session.Response，在session.OnResponse设置Task.tcs.SetResult，这时候await会结束并且获得返回值。
  - Move
  - Numeric
  - ObjectWait
  - Recast
  - Scene
  - Timer
  - Unit

##### ModelView

###### Client

- Demo
  - Canera
  - Config
  - Global
  - Opera
  - UI
  - Unit
- Module
  - Resource
  - UI

<!-- #### Proto 消息类
  - InnerMessage.proto
  - MongoMessage.proto
  - OuterMessage.proto

- Unity/Assets/Script -->

#### Core

##### Analyzer

##### Helper

其实融合了Utility和Extension扩展方法,如果是我就拆了它,放到Tools

- ProcessHelper.cs   根据路径和参数，运行一个进程

##### Method

为了运行ET.Client.Entry而专门实现的。也应该放到Tools比较合适

##### Module

###### Config

###### CoroutineLock

###### Entity

- Entity 其实目录应该叫Scene,即唯一的Entity
  - Scene.cs

###### EventSystem

- Event

  - IEvent.cs
  - AEvent.cs
- EventSystem.cs    实际包含Event、CallBack、Feature(管理System)

  - allTypes      即Hotfix.dll的所有类型
  - types         即有XXAttribute对应的类型

  ---


  - twoQueues     继承Load、Update、LateUpdateSystem中的一个;
  - allEntities   这部分与其他数据有所不同，不是LoadHotfix的时候,而是手动RegisterSystem的时候添加
  - RegisterSystem()添加到AllEntities和twoQueues
  - typesystems   含有ObjectSystemAttribute特性的XXXComponentSystem,这些会继承LoadSystem、AwakeSystem、DestroySystem、UpdateSystem、LateUpdateSystem、GetComponentSystem、DestroySystem、DeserializeSystem、AddComponentSystem中的一个,typesystems的结构是：Dictionary<typeof(XXXComponent), Dictionary<typeof(ObjectSystem), List `<XXXComponentSystem>`>>
  - Deserialize() 运行Component的含有ObjectSystemAttribute特性的DeserializeSystem.Run()
  - GetComponent()运行Component的含有ObjectSystemAttribute特性的GetComponentSystem.Run()
  - AddComponent()运行Component的含有ObjectSystemAttribute特性的AddComponentSystem.Run()
  - Awake()       运行Component的含有ObjectSystemAttribute特性的AwakeSystem.Run()
  - Load()        运行Component的含有ObjectSystemAttribute特性的LoadSystem.Run()
  - Destroy()     运行Component的含有ObjectSystemAttribute特性的DestroySystem.Run()
  - Update()      运行Component的含有ObjectSystemAttribute特性的UpdateSystem.Run()
  - LateUpdate()  运行Component的含有ObjectSystemAttribute特性的LateUpdateSystem.Run()

  ---

  - allEvents      含有EventAttribute特性的,即所有XXXChange_XXX
  - PublishAsync() 同步函数,事件通知,触发所有继承AEvent的Run()方法
  - Publish()      异步函数

  ---

  - allCallbacks   含CallbackAttribute特性的
  - Callback()     触发回调的Run()方法

###### IdGenerator

###### Log

Log里面又调用了Game.ILog,正常会用GameFramework的Helper那样耦合会更低一些,也是放到Tools比较合适

###### Network

A是基类, T是TCP, K是KCP, W是WebSocket

- AChannel.cs
- KChannel.cs
- TChannel.cs
- WChannel.cs
- AService.cs
- KService.cs
- TService.cs       监听客户端的连接请求，有客户端请求过来是，建立新的socket保持客户端与服务端之间通信
- WService.cs

###### ObjectPool

###### Options

###### Synchronization

###### Time

###### Timer

##### Object

##### Serialize

##### Singleton

##### Other

- ThreadSynchronizationContext.cs 将其他线程中的回调统一放在主线程中进行处理,只在TChannel、TService上用到,因为SocketAsyncEventArgs的OnComplete回调是在新线程上处理，需要压回主线程处理。

#### ThirdParty

##### ETTask

##### Kcp

##### protobuf-net

##### Recast

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
