# Toggle

> Toggle Attribute:用于热和一个为bool的属性或者字段，让此bool成员成为一个可以控制开关的toggle。

##### 【Toggle 】构造函数传一个类含有的bool成员名称，作为开关的bool

![img](../image/Toggle/post-698-5fb7dd1334b70.gif)

```cs
using Sirenix.OdinInspector;
using System;
using UnityEngine;

public class ToggleAttributeExample : MonoBehaviour
{
    [Toggle("Enabled")]
    public MyToggleable Toggler = new MyToggleable();

    public ToggleableClass Toggleable = new ToggleableClass();

    [Serializable]
    public class MyToggleable
    {
        public bool Enabled;
        public int MyValue;
    }

    // 您还可以直接在类定义上使用Toggle属性。
    [Serializable, Toggle("Enabled")]
    public class ToggleableClass
    {
        public bool Enabled;
        public string Text;
    }
}
```