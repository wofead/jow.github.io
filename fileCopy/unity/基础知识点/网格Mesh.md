---
layout:     post
title:      Unity Grid
subtitle:   Grid
date:       2021-3-16
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - Unity

---

### 目录
1. 物体的渲染
2. 创建网格顶点
3. 创建网格
4. 生成额外的顶点数据

> Study Hard, record and review.

> 可参照文章： https://mp.weixin.qq.com/s/jIKG2rpNkgVQx2BIWjmYvg

## 物体的渲染

如果你想在Unity中显示物体，你就需要一个网格。它可以是一个从其它3D建模软件导出来的3D模型，也可以是程序生成的网格。它也可以是一个精灵、UI元素或者粒子系统等一系列Unity中使用的网格。甚至屏幕特效也是通过网格来渲染的。

那个，网格究竟是什么呢?理论上说，网格是图形硬件用来绘制复杂东西的一种构造。它至少包含一组定义三维空间中的点，以及一组连接这些点的最基本二维形状三角形。三角形构成网格所代表的表面。

因为三角形是平的，有直的边，它们可以用来完美地描述平的和直的东西，比如一个立方体的面。曲面或圆面只能用许多小三角形来近似。如果三角形看起来足够小，不大于一个像素，那么你不会注意到近似值。通常情况下，这对于实时性能来说是不可行的，所以表面总是在某种程度上呈现锯齿状。

> Unity中怎么展示这些线框呢？视图左上角选择Display Mode，前三个选项分别是Shaded（着色）、Wrieframe（线框）和Shaded Wrieframe着色并带着线框。

如果你想要展示一个3D游戏物体，它必须拥有两个组件。

1. MeshFilter 这个组件记录了你想要展示的网格数据。
2. MeshRenderer 使用这个组件告诉网格如何渲染，比如使用哪个材质球，是否接受阴影和其他设置。

![](https://catlikecoding.com/unity/tutorials/procedural-grid/01-cube-object.png)

​																	***Unity's default cube game object.***

> 为什么这里的Materials是一个数组呢？
>
> 一个网格渲染组件可以用拥有多个材质，它通常被用来渲染多组三角面，也称为子网格。它通常在外界导入的模型中使用，本篇文章不使用多个材质球。

你可以完全改变网格的显示效果通过调整材质球。Unity默认的材质球是简单的白色立体。你可以自己创建一个新的材质球通过Assets->Create->Material并拖拽到你的游戏物体上来代替它。新的材质球默认为Unity Standard Shader，它给你一组控件来调整你想要模型表现。

一个快速添加细节的方法是给你的网格提供一个反射贴图，这个纹理描绘了材质球的基础颜色。当然我们需要知道如何投射纹理到网格的三角面上。这需要添加2D纹理坐标到顶点上。这二维纹理空间被称为U和V，也是常说的UV坐标。UV坐标通常在(0,0)到(1,1)之间，它覆盖了整个纹理。超出范围的坐标将造成clamped或者Tiling平铺的效果，这去取决于纹理设置。

## 创建网格顶点

那么我们这么创建自己的网格呢？通过生成一个简单的矩形网格来了解一下。这个网格将包含方形瓷砖（单位长度的四边形）。创建一个新的C#脚本并加入水平和垂直尺寸。

```c#
using UnityEngine;
using System.Collections;

public class Grid : MonoBehaviour {

	public int xSize, ySize;
}
```

当我们给游戏物体挂上这个脚本，我们需要给它同时添加MeshFilter和MeshRenderer组建。我们可以添加属性在类的前面，Unity将自动为们添加这些类。

```c#
[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class Grid : MonoBehaviour {

	public int xSize, ySize;
}
```

现在创建一个空物体并挂载组建，它将自动同时添加MeshFilter和MeshRenderer组建。设置MeshRenderer的材质并确保MeshFilter的mesh属性为未定义。并设置网格尺寸为10和5。

我们生成真实的网格在物理被唤醒的时候，Awake方法将在我们游戏游戏模式时自动调用。

```c#
private void Awake () {
    Generate();
}
```

现在我们先考虑顶点位置稍后处理三角面序号。我们需要一个数组存储3D顶点位置。顶点的数量取决于网格的尺寸。我们需要获取每个四边形每个角的顶点，但是相邻的四边形可以共享相同的顶点。所以每个维度商定点数比网格数多一个。

```c#
private Vector3[] vertices;
private void Generate () {
    vertices = new Vector3[(xSize + 1) * (ySize + 1)];
    for (int i = 0, y = 0; y <= ySize; y++) {
        for (int x = 0; x <= xSize; x++, i++) {
            vertices[i] = new Vector3(x, y);
        }
    }
}
```

我们在场景中绘制这些顶点，这样我们就可以正确地核对它们的位置。利用OnDrawGizmos方法来绘制顶点的位置，利用OnDrawGizmos在每个顶点绘制一个小球。

```c#
private void OnDrawGizmos () {
    if (vertices == null) {
			return;
		}
    Gizmos.color = Color.black;
    for (int i = 0; i < vertices.Length; i++) {
        Gizmos.DrawSphere(vertices[i], 0.1f);
    }
}
```



> 什么是Gizmos？
>
> - 在编辑模式下Gizmos可以提供可视化的提示。它们只在Scene场景中显示不在Play模式下显示，但是你可以通过工具栏调整它们。Gizmos公共类允许你绘制图标，线条，和其他的东西。
> - Gizmos在OnDrawGizmos方法中执行绘制，它被Unity编辑器自动调用。另一个可选的方法是OnDrwaGizmosSelected，它只能被可选的对象调用。

> 为什么在Gizmos绘制不能被移动?
>
> Gizmos直接使用世界坐标绘制，不是使用对象的本地坐标系，如果你想它们遵循对象的Transform组件，你必须声明使用transform.TransformPoint(vertices[i]) 代替 vertices[i].

现在我们可以看见这些顶点，但是他们生成的顺序我们不能明显地看出来。我们可以使用颜色来标识，但是我们也可以使用协程减慢这个过程的速度。这就是为什么我在这个脚本中引用了System.Collections命名空间。

```c#
private void Awake () {
    StartCoroutine(Generate());
}

private IEnumerator Generate () {
    WaitForSeconds wait = new WaitForSeconds(0.05f);
    vertices = new Vector3[(xSize + 1) * (ySize + 1)];
    for (int i = 0, y = 0; y <= ySize; y++) {
        for (int x = 0; x <= xSize; x++, i++) {
            vertices[i] = new Vector3(x, y);
            yield return wait;
        }
    }
}
```

## 创建网格

现在我们知道顶点的位置是正确的，接下来我们就处理真正的网格。除此之外我们想要我们的组件持有这些顶点，我们必须把顶点指定到MeshFilter.mesh中。我们处理过顶点之后，就可以网格存储在MeshFilter中。

```c#
private Mesh mesh;

private IEnumerator Generate () {
    WaitForSeconds wait = new WaitForSeconds(0.05f);

    GetComponent<MeshFilter>().mesh = mesh = new Mesh();
    mesh.name = "Procedural Grid";

    vertices = new Vector3[(xSize + 1) * (ySize + 1)];
    …
        mesh.vertices = vertices;
}
```

> 我们的组件需要保持网格吗？
>
> 我们只需要在Generate方法中引用网格。因为网格过滤器也有一个引用，它会一直留在周围。我让它成为一个全局变量，因为本教程的下一个逻辑步骤是动画网格，我鼓励你尝试一下。

到目前，在Play模式下我们可以持有一个网格，但是它依旧是不可见的，因为我们并没有给它任何三角面。三角面由顶点数组索引决定。每一个三角面有三个顶点，三个连续的顶点描绘了一个三角形，让我们来先绘制第一个三角面。

```c#
private IEnumerator Generate () {
    …

        int[] triangles = new int[3];
    triangles[0] = 0;
    triangles[1] = 1;
    triangles[2] = 2;
    mesh.triangles = triangles;
}
```

好的，目前我们已经绘制了一个三角面，但是由于三个顶点在一条直线上，所以生成了一个失败的三角形，它是不可见的。前两个顶点是正确的，第三个顶点应该跳转到下一行的第一个顶点。

```c#
triangles[0] = 0;
triangles[1] = 1;
triangles[2] = xSize + 1;
```

通过以上操作，我们绘制了一个三角形，但是它只能在一个方向可见。这种情况下，只有Z轴的反方向可见，所以你可能需要旋转视角才能看到它。

三角形的哪一面可见是由顶点序号的方向来确定的。默认情况下，如果顶点顺序是顺时针方向的话那么三角形是正面可见。逆时针（就是逆屏幕方向）的三角面是被抛弃的，所以我们不必花费时间去渲染这部分顶点，以为他们通常都是不可见的。

所以为了实线从Z轴负方向到正方向可见，我们必须改变顺序相反的顶点的位置。我们交换后两个顶点的序号即可。
```c#
triangles[0] = 0;
triangles[1] = xSize + 1;
triangles[2] = 1;
```

现在我们绘制了一个三角面只覆盖了一个四方瓦片的一半，为了覆盖整个瓦片，我们需要第二个三角面。

```c#
int[] triangles = new int[6];
triangles[0] = 0;
triangles[1] = xSize + 1;
triangles[2] = 1;
triangles[3] = 1;
triangles[4] = xSize + 1;
triangles[5] = xSize + 2;
```

**一个四边形由两个三角面组成**

既然这些顶点共用两个顶点，我们就可以减少我们代码的行数，明确地提到每个顶点索引只有一次。

```c#
triangles[0] = 0;
triangles[3] = triangles[2] = 1;
triangles[4] = triangles[1] = xSize + 1;
triangles[5] = xSize + 2;
```

我们可以通过循环来创建剩余第一行的瓦片。尽管我们遍历所有的顶点和三角面序号，但是我们必须要保证顶点和三角面序号是按照顺序的。我们把yield代码的声明放到循环里，我们就不需要等待顶点的出现了。

```c#
int[] triangles = new int[xSize * 6];
for (int ti = 0, vi = 0, x = 0; x < xSize; x++, ti += 6, vi++)
{
    triangles[ti] = vi;
    triangles[ti + 3] = triangles[ti + 2] = vi + 1;
    triangles[ti + 4] = triangles[ti + 1] = vi + xSize + 1;
    triangles[ti + 5] = vi + xSize + 2;
    yield return wait;
}
```

现在，Gizmos可以立刻渲染出顶点，并且所有的三角面在一段时间后统一出现。要想看到瓦片一个接一个的出现，我们必须每次循环都刷新网格代替掉执行完所有循环刷新。

```c#
mesh.triangles = triangles;
yield return wait;
```

现在我们来填充剩余所有的网格通过将单体循环转变为双重循环。一定要注意移动到下一行的时一定要增加一次顶点序号，因为每一行顶点比列数多一。

```c#
int[] triangles = new int[xSize * ySize * 6];
for (int ti = 0, vi = 0, y = 0; y < ySize; y++, vi++) 
{
    for (int x = 0; x < xSize; x++, ti += 6, vi++)
    {
        …
    }
}
```

正如你所能看到的，所有的网格都被三角面填充，每一行都是同时填充，因为我们使用了协程。如果你对想过感到满意，你可以移除掉所有的协程代码，这样网格创建就没有任何的延迟了。

```c#
private void Awake () 
{
    Generate();
}

private void Generate () 
{
    GetComponent<MeshFilter>().mesh = mesh = new Mesh();
    mesh.name = "Procedural Grid";

    vertices = new Vector3[(xSize + 1) * (ySize + 1)];
    for (int i = 0, y = 0; y <= ySize; y++) {
        for (int x = 0; x <= xSize; x++, i++) {
            vertices[i] = new Vector3(x, y);
        }
    }
    mesh.vertices = vertices;

    int[] triangles = new int[xSize * ySize * 6];
    for (int ti = 0, vi = 0, y = 0; y < ySize; y++, vi++) {
        for (int x = 0; x < xSize; x++, ti += 6, vi++) {
            triangles[ti] = vi;
            triangles[ti + 3] = triangles[ti + 2] = vi + 1;
            triangles[ti + 4] = triangles[ti + 1] = vi + xSize + 1;
            triangles[ti + 5] = vi + xSize + 2;
        }
    }
    mesh.triangles = triangles;
}
```

## 生成额外的顶点数据

我们的网格目前处于一种特殊的情况下。因为我们没有直到目前还没有给他们法线向量，默认的发现向量是(0,0,1)（垂直于屏幕向里），而我们需要的正好相反。

> 法线工作原理是什么呢？
>
> 法线是垂直于面的向量。我们通常使用单位长度的法向量，并向量指向面的外部，而不是内部。
>
> 法线可以用于确定光线与顶点的夹角。这个细节的使用取决于Shader。
>
> 作为三角面它永远是平的，因此它不应该需要被提供一个单独的法线信息。然而，我们需要造假。在现实中，顶点时不存在法线的，三角面才有。通过附加自定义顶点法线和三角面插着，我们可以假装我们有一个平滑的曲面代替一堆平的三角面。这个错觉是令人信服的，只要你不去注意网格锋利的轮廓（锯齿）。

法线用于规定每个顶点，所以我们必须填充另一个向量数组。另一种选择，我们可以依据网格的三角面来计算出法线。我们也可以偷懒，向下面这样做。

```c#
private void Generate () 
{
    …
    mesh.triangles = triangles;
    // 网格自动计算法线向量
    mesh.RecalculateNormals();
}
```

> 法线是如何重新计算的？
>
> Mesh.RecalculateNormals方法利用与顶点相连的三角面计算出每一个顶点的法线。计算平面三角形法线平均值，然后使用normalize方法单位化。

接下来是UV坐标。你可能注意到网格目前是颜色统一的，即便是我们给它赋值一个带反射贴图的材质球。这很容易理解，因为我们没有自行给它提供UV坐标，他们默认为(0,0)。

想要纹理适应我们的网格，简单地划分顶点的位置通过网格的大小。

```C#
vertices = new Vector3[(xSize + 1) * (ySize + 1)];
Vector2[] uv = new Vector2[vertices.Length];
for (int i = 0, y = 0; y <= ySize; y++) 
{
    for (int x = 0; x <= xSize; x++, i++) 
    {
        vertices[i] = new Vector3(x, y);
        uv[i] = new Vector2(x / xSize, y / ySize);
    }
}
mesh.vertices = vertices;
mesh.uv = uv;
```

纹理现在显示了，但是它并没有覆盖整个网格。它的真实外观取决于纹理模式是Clamp模式或者是Repeat模式。产生这种现象是因为当前我们是通过整数划分的，UV坐标的计算结果是整数。为了得到正确的在0到1之间的坐标，我们必须使用浮点数。

```c#
uv[i] = new Vector2((float)x / xSize, (float)y / ySize);
```

纹理现在被投影到了整个纹理上。当我设置网格尺寸为(5,10)，这个网格纹理将出现横向拉伸，可以自动反向适应材质的纹理的平铺设置。设置X的坐标为(2,1)将产生双倍。如果把纹理模式设置为Repeat模式，我们可以看到两个四方瓷砖网格。

纹理现在被投影到了整个纹理上。当我设置网格尺寸为(5,10)，这个网格纹理将出现横向拉伸，可以自动反向适应材质的纹理的平铺设置。设置X的坐标为(2,1)将产生双倍。如果把纹理模式设置为Repeat模式，我们可以看到两个四方瓷砖网格。

另外一种添加表面细节的方法是使用法线贴图。这个贴图使用颜色值记录了法相向量。应用这个这个纹理到网格上将产生更多的灯光细节效果，它是单独使用顶点法线产生的。

目前应用这个材质球到我们的网格将不会产生任何的凹凸效果。我们需要给我们的网格添加切线向量。

> 切线的工作原理是什么？
>
> 法线贴图在切线空间中定义。这是一个流动在物理表面的3D空间。这个方法使我们能够使用相同的法线贴图在不同的空间和方向。
>
> 在这个空间中，表面法线代表向上，但哪个方向是正确的呢?它由切线决定。理想情况下，这两个向量的夹角是90度。它们的外积产生了定义三维空间所需的第三个方向。在现实中，角度往往不是90度，但结果仍然足够好。
>
> 所以切线是一个三维向量，但是在Unity中它是使用四维向量定义的。第四个值通常是1或者-1，用于控制第三切线空间唯独方向-朝前或朝后,这有助于展示法线贴图，通常用于左右对称的3D模型，像人一样。Untiy的shader执行此计算要求我们使用-1。

当我们有一个平面，所有的切线仅仅指向相同的方向，是正确的。

```c#
vertices = new Vector3[(xSize + 1) * (ySize + 1)];
Vector2[] uv = new Vector2[vertices.Length];
Vector4[] tangents = new Vector4[vertices.Length];
Vector4 tangent = new Vector4(1f, 0f, 0f, -1f);
for (int i = 0, y = 0; y <= ySize; y++)
{
    for (int x = 0; x <= xSize; x++, i++) 
    {
        vertices[i] = new Vector3(x, y);
        uv[i] = new Vector2((float)x / xSize, (float)y / ySize);
        tangents[i] = tangent;
    }
}
mesh.vertices = vertices;
mesh.uv = uv;
mesh.tangents = tangents;
```

现在你知道如何创建一个简单的网格并使得它在使用材质球的情况下看起来更加复杂。**网格需要顶点坐标，三角面，通常还需要UV坐标，经常也需要法线和切线。**你也可以添加顶点颜色，尽管Unity标准着色器不使用这个属性。但是你可以自己创建Shader去使用这个颜色属性，如果你想了解如何使用使用自己的Shader使用这个颜色属性，情况另一篇文章。