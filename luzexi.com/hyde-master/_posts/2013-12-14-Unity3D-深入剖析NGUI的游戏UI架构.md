---
layout: post
status: publish
published: true
title: Unity3D-深入剖析NGUI的游戏UI架构
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 182
wordpress_url: http://www.luzexi.com/?p=182
date: !binary |-
  MjAxMy0xMi0xNCAxMzowMDo1NCArMDgwMA==
date_gmt: !binary |-
  MjAxMy0xMi0xNCAwNTowMDo1NCArMDgwMA==
categories:
- Unity3D
- 游戏通用模块
- 前端技术
tags:
- NGUI
- UI
- Unity3D
- 客户端架构
- 服务器架构设计
- 游戏架构
---
Unity3D-NGUI分析，使用NGUI做UI需要注意的几个要点在此我想罗列一下，对我在U3D上做UI的一些总结，最后解剖一下NGUI的源代码，它是如果架构和运作的。

在此前我介绍了自己项目的架构方式，所以在NGUI的利用上也是同样的做法，UI逻辑的程序不被绑定在物体上。那么如何做到GUI输入消息的传递呢，答案是：我封装了一个关于NGUI输入消息的类，由于NGUI的输入消息传递方式是U3D中的SendMessage方式，所以在每个需要接入输入的物体上动态的绑定该封装脚本。在这个消息封装类中，加入消息传递的委托方法后，所有关于该物体的输入消息将通过封装类直接传递到方法上，再通过消息类型的识别就可以脱离传统脚本绑定的束缚了。


在用NGUI制作UI时需要注意的几点：

1.每个GUI以1各UIPanel为标准，过多的UIPanel首先会导致DrawCall的增多，其次是导致UI逻辑的混乱。

2.UITexture不能使用的过于平凡，因为每个UITexture都会增加1各DrawCall，所以一般会作为背景图出现在UI上，小背景，大背景都可以。

3.图集不宜过大，过大的图集，不要把很多个GUI都放在一个图集里，在UI显示时加载资源IO速度会非常慢。我尝试了各种方式来管理图集，例如每个GUI一个图集，大雨300*100宽度的图不做图集，抑或一个系统模块2个图集，甚至我有尝试过以整个游戏为单位划分公共图集，按钮图集，头像图集，问题图集，但这种方式最终以图集过大IO过慢而放弃，这些图集的管理方式都是应项目而适应的，并没有固定的方式，最主要是你怎么理解程序读取资源时的IO操作时间。

4.在开发中，尽量用Free分辨率来测试项目的适配效果，不要到上线才发现适配问题。

适配源码：

``` c#
float defaultWHRate = 800f / 480f;
float ScreenWHRate = (float)Screen.width / (float)Screen.height;
bool isUseHResize = defaultWHRate >= ScreenWHRate ? false : true;
UIRoot root = GameObject.Find("ROOT").GetComponent<UIRoot>();
if (!isUseHResize)
{
    float curScreenH = (float)Screen.width / defaultWHRate;
    float Hrate = curScreenH / Screen.height;
    root.manualHeight =(int)(480f / Hrate);
}
else
{
    root.manualHeight = 480;
}
```

{% include advertisement_content.html %}

5.拆分以及固定各个锚点，上，左上，右上，中，左中，右中，下，左下，右下

6.拆分GUI层级，层级越高，显示越靠前。层级的正确拆分能有效管理GUI的显示方式。

``` c#
/// <summary>
/// GUI层级
/// </summary>
public enum GUILAYER
{
    GUI_BACKGROUND = 0, //背景层
    GUI_MENU,           //菜单层0
    GUI_MENU1,           //菜单层1
    GUI_PANEL,          //面板层
    GUI_PANEL1,         //面板1层
    GUI_PANEL2,         //面板2层
    GUI_PANEL3,         //面板3层
    GUI_FULL,           //满屏层
    GUI_MESSAGE,        //消息层
    GUI_MESSAGE1,        //消息层
    GUI_GUIDE,           //引导层
    GUI_LOADING,        //加载层
}
```

8.要充分的管理GUI，不然过多的GUI会导致内存加速增长，而每次都销毁不用的GUI则会让IO过于频繁降低运行速度。我的方法是找到两者间的中间态，给予隐藏的GUI一个缓冲带,当每次某各GUI进行隐藏时判断是否有需要销毁的GUI。或者也可以这么做，每时每刻去监控隐藏的GUI，哪些GUI内存时间驻留过长就销毁。关于内存优化问题，可以参考[《unity3d-texture图片空间和内存占用分析》](/unity3d/前端技术/2014/05/21/Unity3D-Texture图片空间和内存占用分析.html)和 [《unity3d优化之路》](/unity3d/游戏架构/前端技术/2014/02/22/Unity3d优化之路.html)

9.另外关于图标，像头像，物品，数量过多的，可以用打成几个图集，按一定规则进行排列，减小文件大小减少一次性读取的IO时间。

10.尽量减少不必要的UI更改，NGUI一旦有UI进行更改，它就得重新绘制MESH和贴图，比起cocos2d耗得CPU大的多。

11.如果可以不用动态字体就不要用动态字体，因为动态字体每次都会做IO操作读取相应的图片，这个是NGUI一个问题，费cpu，费内存。

12.设置脚本执行次序，在U3D的Project setting->Script Execution Order 中。由于NGUI以UIPanel为主要渲染入口，所以，所有关于游戏渲染处理的程序最好放在渲染之后，也就是UIPanel之后。UIPanel以LateUpdate为接口入口，所以关于渲染方面的程序还得斟酌是否方在LateUpdate里。

13.NGUI对于动态的移动旋转等的UI操作支持性很差，当有这种操作过多的时候，会使得屏幕很卡。解决办法就是，自己用程序生成面片，面片的渲染不再受到NGUI的控制。
 
以上是我能想起来的注意点，若有没想起来的，在以后的时间想到的也将补充进去。口无遮拦的说了这么多，不剖析一下源码怎么说的过去，之前对NGUI输入消息进行了封装，对2D动画序列帧进行了封装，却一直没能完整剖析它的底层源码，着实遗憾。

{% include advertisement_content.html %}

NGUI中UIPanel是渲染的关键，他承载了在他下面的子物体的所有渲染工作，每个渲染元素都是由UIWidget继承而来，每个UI物体的渲染都是由面片、材质球、UV点组成，每个种材质由一个UIDrawCall完成渲染工作，UIDrawCall中自己创建Mesh和MeshRender来进行统一的渲染工作。这些都是对NGUI底层的简单的介绍，下面将进行更加细致的分析。

首先我们来看UIWidget这个组件基类，从它拥有的类内部变量就能知道它承担得怎样的责任:

``` c#
// Cached and saved values
[HideInInspector][SerializeField] protected Material mMat;//材质
[HideInInspector][SerializeField] protected Texture mTex;//贴图
[HideInInspector][SerializeField] Color mColor = Color.white;//颜色
[HideInInspector][SerializeField] Pivot mPivot = Pivot.Center;//对齐位置
[HideInInspector][SerializeField] int mDepth = 0;//深度
protected Transform mTrans;//坐标转换
protected UIPanel mPanel;//相应的UIPanel

protected bool mChanged = true;//是否更改
protected bool mPlayMode = true;//模式

Vector3 mDiffPos;//位置差异
Quaternion mDiffRot;//旋转差异
Vector3 mDiffScale;//缩放差异
int mVisibleFlag = -1;//可见标志

// Widget's generated geometry
UIGeometry mGeom = new UIGeometry();//多变形实例
```

UIWidget承担了存储显示内容，颜色调配，显示深度，显示位置，显示大小，显示角度，显示的多边形形状，归属哪个UIPanel。这就是UIWidget所要承担的内容，在UIWidget的所有子类中都具有以上相同的属性和任务。UIWidget和UIPanel的关系非常密切，因为UIPanel承担了UIWidget的所有渲染工作，而UIWidget只是承担了存储需要渲染数据。所以，在UIWidget在更换贴图，材质球，甚至更换UIPanel父节点时它会及时通知UIPanel说："我更变配置了，你得重新获取我的渲染数据"。

UIWidget中最重要的虚方法为 virtual public void OnFill(BetterList<Vector3> verts, BetterList<Vector2> uvs, BetterList<Color32> cols) { } 它是区分子类的显示内容的重要方法。它的工作就是填写如何显示，显示什么。

UIWidget中在使用OnFill方法的重要的方法是 更新渲染多边型方法：

{% include advertisement_content.html %}

``` c#
public bool UpdateGeometry (ref Matrix4x4 worldToPanel, bool parentMoved, bool generateNormals)
{
  if (material == null) return false;

  if (OnUpdate() || mChanged)
  {
    mChanged = false;
    mGeom.Clear();
    OnFill(mGeom.verts, mGeom.uvs, mGeom.cols);

    if (mGeom.hasVertices)
    {
      Vector3 offset = pivotOffset;
      Vector2 scale = relativeSize;
      offset.x *= scale.x;
      offset.y *= scale.y;

      mGeom.ApplyOffset(offset);
      mGeom.ApplyTransform(worldToPanel * cachedTransform.localToWorldMatrix, generateNormals);
    }
    return true;
  }
  else if (mGeom.hasVertices && parentMoved)
  {
    mGeom.ApplyTransform(worldToPanel * cachedTransform.localToWorldMatrix, generateNormals);
  }
  return false;
}
```

它的作用就是，当需要重新组织多边型展示内容时，进行多边型的重新规划。
 
{% include advertisement_content.html %}

接着，我们来看看UINode，这个类很容易被人忽视，而他的作用也很重要。它是在UIPanel被告知有新的UIWidget显示元素时被创建的，它的创建主要是为了监视被创建的UIWidget的位置，旋转，大小是否被更改，若被更改，将由UIPanel进行重新的渲染工作。
HasChanged这是UINode唯一重要的方法之一，它的作用就是被UIPanel用来监视每个元素是否改变了进而进行重新渲染。

``` c#
public bool HasChanged ()
{
#if UNITY_3 || UNITY_4_0
  bool isActive = NGUITools.GetActive(mGo) && (widget == null || (widget.enabled && widget.isVisible));

  if (lastActive != isActive || (isActive &&
  (lastPos != trans.localPosition ||
  lastRot != trans.localRotation ||
  lastScale != trans.localScale)))
  {
    lastActive = isActive;
    lastPos = trans.localPosition;
    lastRot = trans.localRotation;
    lastScale = trans.localScale;
    return true;
  }
#else
  if (widget != null && widget.finalAlpha != mLastAlpha)
  {
    mLastAlpha = widget.finalAlpha;
    trans.hasChanged = false;
    return true;
  }
  else if (trans.hasChanged)
  {
    trans.hasChanged = false;
    return true;
  }
#endif
  return false;
}
```

接着，来看UIDrawCall，它是被NGUI隐藏起来的类。他的内部变量来看看：

Transform        mTrans;            //坐标转换类

Material        mSharedMat;        // 渲染材质

Mesh            mMesh0;            //首个MESH

Mesh            mMesh1;            //用于更换的Mesh

MeshFilter        mFilter;        //绘制的MeshFilter

MeshRenderer    mRen;            //渲染MeshRender组件

Clipping        mClipping;        //裁剪类型

Vector4            mClipRange;        //裁剪范围

Vector2            mClipSoft;        //裁剪缓冲方位

Material        mMat;            //实例化材质

int[]            mIndices;        //做为Mesh三角型索引点

由这些内部变量可知，UIDrawCall是负责NGUI的最重要的渲染类。他制造Mesh制造Material，设置裁剪范围，为NGUI提供渲染底层。
他最重要的方法是：

{% include advertisement_content.html %}

``` c#
public void Set (BetterList<Vector3> verts, BetterList<Vector3> norms, BetterList<Vector4> tans, BetterList<Vector2> uvs, BetterList<Color32> cols)
{
  int count = verts.size;

  // Safety check to ensure we get valid values
  if (count > 0 && (count == uvs.size && count == cols.size) && (count % 4) == 0)
  {
    // Cache all components
    if (mFilter == null) mFilter = gameObject.GetComponent<MeshFilter>();
    if (mFilter == null) mFilter = gameObject.AddComponent<MeshFilter>();
    if (mRen == null) mRen = gameObject.GetComponent<MeshRenderer>();

    if (mRen == null)
    {
      mRen = gameObject.AddComponent<MeshRenderer>();
      #if UNITY_EDITOR
      mRen.enabled = isActive;
      #endif
      UpdateMaterials();
    }
    else if (mMat != null && mMat.mainTexture != mSharedMat.mainTexture)
    {
      UpdateMaterials();
    }

    if (verts.size < 65000)
    {
      int indexCount = (count >> 1) * 3;
      bool rebuildIndices = (mIndices == null || mIndices.Length != indexCount);

      // Populate the index buffer
      if (rebuildIndices)
      {
        // It takes 6 indices to draw a quad of 4 vertices
        mIndices = new int[indexCount];
        int index = 0;

        for (int i = 0; i < count; i += 4)
        {
          mIndices[index++] = i;
          mIndices[index++] = i + 1;
          mIndices[index++] = i + 2;

          mIndices[index++] = i + 2;
          mIndices[index++] = i + 3;
          mIndices[index++] = i;
        }
      }

      // Set the mesh values
      Mesh mesh = GetMesh(ref rebuildIndices, verts.size);
      mesh.vertices = verts.ToArray();
      if (norms != null) mesh.normals = norms.ToArray();
      if (tans != null) mesh.tangents = tans.ToArray();
      mesh.uv = uvs.ToArray();
      mesh.colors32 = cols.ToArray();
      if (rebuildIndices) mesh.triangles = mIndices;
      mesh.RecalculateBounds();
      mFilter.mesh = mesh;
    }
    else
    {
      if (mFilter.mesh != null) mFilter.mesh.Clear();
      Debug.LogError("Too many vertices on one panel: " + verts.size);
    }
  }
  else
  {
    if (mFilter.mesh != null) mFilter.mesh.Clear();
    Debug.LogError("UIWidgets must fill the buffer with 4 vertices per quad. Found " + count);
  }
}
```

在这个方法里，它制造Mesh,MeshFilter,MeshRender,Materials。
 
最后，我们来说说最重要的UI渲染入口UIPanel。
    UIPanel的渲染步骤：
    1.当有任何形式的UI组件启动渲染时加入UIPanel的渲染队列，当有新的渲染组件需要有新的UIDrawCall时，进行生成新的UIDrawCall.
    2.对所有UIPanel的渲染队列进行检查，是否队列中渲染组件需要重新渲染，包括位移，缩放，更改图片，启用，关闭.
    3.获取渲染组件对应的UIDrawCall，更新Mesh,贴图,UV，位置，大小
    4.对需要更新的UIDrawCall进行重新渲染
    5.最后标记已经渲染的渲染组件，告诉他们已经渲染，为下次判断更新做好准备。删除不再需要渲染的UIDrawCall，销毁渲染冗余。
    注意：所有的渲染都是在LateUpdate下进行，也就是它是进行的延迟渲染。

接口源码：

{% include advertisement_content.html %}

``` c#
void LateUpdate ()
{
  // Only the very first panel should be doing the update logic
  if (list[0] != this) return;

  // Update all panels
  for (int i = 0; i < list.size; ++i)
  {
  UIPanel panel = list[i];
  panel.mUpdateTime = RealTime.time;
  panel.UpdateTransformMatrix();
  panel.UpdateLayers();
  panel.UpdateWidgets();
}

// Fill the draw calls for all of the changed materials
if (mFullRebuild)
{
  UIWidget.list.Sort(UIWidget.CompareFunc);
  Fill();
}
else
{
  for (int i = 0; i < UIDrawCall.list.size; )
  {
    UIDrawCall dc = UIDrawCall.list[i];

    if (dc.isDirty)
    {
      if (!Fill(dc))
      {
        DestroyDrawCall(dc, i);
        continue;
      }
    }
    ++i;
  }
}

  // Update the clipping rects
  for (int i = 0; i < list.size; ++i)
  {
    UIPanel panel = list[i];
    panel.UpdateDrawcalls();
  }
  mFullRebuild = false;
}
```

Fill()接口源码：

``` c#
/// <summary>
/// Fill the geometry fully, processing all widgets and re-creating all draw calls.
/// </summary>
static void Fill ()
{
  for (int i = UIDrawCall.list.size; i > 0; )
  DestroyDrawCall(UIDrawCall.list[--i], i);

  int index = 0;
  UIPanel pan = null;
  Material mat = null;
  UIDrawCall dc = null;

  for (int i = 0; i < UIWidget.list.size; )
  {
    UIWidget w = UIWidget.list[i];

    if (w == null)
    {
      UIWidget.list.RemoveAt(i);
      continue;
    }

    if (w.isVisible && w.hasVertices)
    {
      if (pan != w.panel || mat != w.material)
      {
        if (pan != null && mat != null && mVerts.size != 0)
        {
          pan.SubmitDrawCall(dc);
          dc = null;
        }

        pan = w.panel;
        mat = w.material;
      }

      if (pan != null && mat != null)
      {
        if (dc == null) dc = pan.GetDrawCall(index++, mat);
        w.drawCall = dc;
        if (pan.generateNormals) w.WriteToBuffers(mVerts, mUvs, mCols, mNorms, mTans);
        else w.WriteToBuffers(mVerts, mUvs, mCols, null, null);
      }
    }
    else w.drawCall = null;
    ++i;
  }

  if (mVerts.size != 0)
  pan.SubmitDrawCall(dc);
}
```

转载请注明出处：http://www.luzexi.com
