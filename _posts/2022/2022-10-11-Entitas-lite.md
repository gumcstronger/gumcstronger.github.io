---
开发layout:     post
title:      "Entitas-Lite常见问题"
subtitle:   " \"Entitas-Lite是没有代码生成器的Entitas\""
date:       2022-10-11 09:03:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 开发
---
## DebugSystem在Editor查看时崩溃

没有实现特殊TypeDrawer会导致崩溃。

查看下Entitas的TypeDrawer，看看自己的Component是否使用了某些类型，但TypeDrawer并没有实现。参考其他实现自己的TypeDrawer，例如我的PositionComp使用了float3类型，需要自己实现Float3TypeDrawer：

```csharp
using System;
using Unity.Mathematics;
using UnityEditor;
using UnityEngine;
namespace Entitas.VisualDebugging.Unity.Editor {
    public class Float3TypeDrawer : ITypeDrawer {

        public bool HandlesType(Type type) {
            return type == typeof(float3);
        }

        public object DrawAndGetNewValue(Type memberType, string memberName, object value, object target) {
            float3 vector = (float3)value;
            vector.x = EditorGUILayout.FloatField("x", vector.x);
            vector.y = EditorGUILayout.FloatField("y", vector.y);
            vector.z = EditorGUILayout.FloatField("z", vector.z);
            return vector;
        }
    }
}
```

## ReactiveSystem的OnAdded/OnRemoved

查看源码Collector.cs

```csharp
        /// Activates the Collector and will start collecting
        /// changed entities. Collectors are activated by default.
        public void Activate() {
            for (int i = 0; i < _groups.Length; i++) {
                var group = _groups[i];
                var groupEvent = _groupEvents[i];
                switch (groupEvent) {
                    case GroupEvent.Added:
                        group.OnEntityAdded -= _addEntityCache;
                        group.OnEntityAdded += _addEntityCache;
                        break;
                    case GroupEvent.Removed:
                        group.OnEntityRemoved -= _addEntityCache;
                        group.OnEntityRemoved += _addEntityCache;
                        break;
                    case GroupEvent.AddedOrRemoved:
                        group.OnEntityAdded -= _addEntityCache;
                        group.OnEntityAdded += _addEntityCache;
                        group.OnEntityRemoved -= _addEntityCache;
                        group.OnEntityRemoved += _addEntityCache;
                        break;
                }
            }
        }
```

group.OnEntityAdded、Group.OnEntityRemoved都会调用_addEntityCache的方法，Entity无论Add还是Removed Component，Entity都会被添加到缓存池中。

所以，如果添加了OnRemoved，则ReactiveSystem中对应的Execute(entities)传入的Entities会包含Added和Removed的Entity。

这个操作无法理解对吧，理论上应该OnAdded的传入的就是Add了Component的Entity，OnRemoved的传入的就是Remove了Component的Entity，但实际上并不是，真奇葩。

所以又提供了Where方法，通过Where去筛选出我们需要的Entity，最开始还以为Filter是脱了裤子放屁多此一举，但到这里才发现这个无语的设计，实际上可以在Entitas层面就避免这个问题，Where只是在特殊处理的时候再实现，过阵子有空的时候再修改下Entitas的代码实现自己筛选的功能。

## Entity.Remove<>会报错

错误信息：

```csharp
System.InvalidOperationException: Collection was modified; enumeration operation may not execute.
  at System.Collections.Generic.List`1+Enumerator[T].MoveNextRare () [0x00013] in <3dd5df5ef4974f29afeb2d3ba227c5da>:0 
  at System.Collections.Generic.List`1+Enumerator[T].MoveNext () [0x0004a] in <3dd5df5ef4974f29afeb2d3ba227c5da>:0 
  at Entitas.Entity+ListenerSystem`1[T].OnAdded (System.Collections.Generic.List`1[T] entities) [0x00047] in
```

这部分跟我新增的ListenerSystem `<T>`事件有关。

将foreach()改成for()避免在事件时RemoveListener导致错误。

## Context.Remove<>不存在。

Context.Add时，SingleEntity代表一个Comp一个Entity，如果删除Com，Entity不删除，则会导致Entity不断增加。

考虑新增Context.Remove方法，而其中需要调用Context.DestroyEntity

## 使用Entitas使用问题

* System具体做哪些内容(从官方例子Match-One看)
  * System根据Entity对Comp的操作，触发修改其他Comp，也就行根据数据变化修改数据。所以逻辑也写在System，如果该逻辑可能在多处使用，再凑出来叫XXXLogic或XXXUtility.cs（守望先锋开发者用Utility，Entitas官方案例用Logic）静态类静态方法使用。
  * 例如：
    * BoardSystem
      * Initialize 创建随机piece
      * Execute 检测Board超出范围则添加DestroyComp
    * FallSystem
      * Execute 检测有MovableComp，修改PositionComp
    * FillSystem
      * Execute 检测到DestroyComp和BoardComp被添加，则创建随机P
    * InputSystem 根据点击添加InputComp
    * ProcessInputSystem 根据InputComp的值，点击到则为该Entity添加DestroyComp
* Unity的交互放在那里呢？交互后做什么？
  * 放到所有继承MonoBehaviour的交互逻辑处。
  * 例如：
    * BurstModeButtonController 点击则增加或删除BurstModeComp
      * Button.onClick.AddListener(() =>contexts.input.isBurstMode=!contexts.input.isBurstMode);
    * 官方案例将Input的获取放到InputSystem，个人觉得不太合适，这样做不方便服务器代码运行，应该放到MonoBehaviour根据Input修改Input状态。
* Unity的状态修改放在哪里呢？
  * 为所有继承MonoBehaviour的类添加IListener增加监听，根据监听修改状态
  * 例如：
    * BurstModeButtonController 根据InputComp变化修改文字。（此处有InputEntity的疑问）
    * CameraView AnyBoard监听变化，触发OnAnyBoard调用修改camera的orthographicSize和Transform
    * ScoreLabelController 根据ScoreComp的变化修改分数
* 如使用Unity的碰撞交互会导致代码跟Unity挂钩严重，无法实现数据和可视化分离。
  * 直观的想法有两个：
    * 一是需要碰撞检测时再调用Unity的方法，这样代码在服务器就无法运行。
    * 二是实现自己的碰撞检测，但就需要为所有Unity GameObject生成对应的数据。
  * 大概思路是
    * Unity获得数据，例如两个立方体数据or导航地形网格，然后在逻辑中继续计算。这需要在数据层就有碰撞检测等逻辑，不能直接使用Unity的接口。
* 不要为了监听增加没有数据的Comp，监听部分，如果需要绑定Entity使用则使用IListener，不需要则使用EventManager。
  * 例如：UI事件。
