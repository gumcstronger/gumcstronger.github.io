---
开发layout:     post
title:      "Entitas-Lite常见问题"
subtitle:   " \"Entitas-Lite是没有代码生成器的Entitas\""
date:       2022-10-07 04:03:00
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
