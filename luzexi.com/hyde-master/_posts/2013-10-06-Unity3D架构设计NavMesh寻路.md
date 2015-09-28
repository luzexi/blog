---
layout: post
status: publish
published: true
title: Unity3D架构设计NavMesh寻路
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 90
wordpress_url: http://www.luzexi.com/?p=90
date: !binary |-
  MjAxMy0xMC0wNiAxODowNDoyOCArMDgwMA==
date_gmt: !binary |-
  MjAxMy0xMC0wNiAxMDowNDoyOCArMDgwMA==
categories:
- Unity3D
tags:
- a*
- c#
- navmesh
- Unity3D
- 自动寻路
---
国庆闲来没事把NavMesh巩固一下。以Unity3D引擎为例写一个底层c# NavMesh寻路。因为Unity3D中本身自带的NavMesh寻路不能很好的融入到游戏项目当中，所以重写一个NavMesh寻路是个必经之路。NavMesh在很多游戏中应用广泛，不同种类的框架下NavMesh寻路发挥的淋漓尽致。与传统的A星寻路相比，NavMesh不仅减少了内存空间占有量，加快了寻路速度，还可以加入寻路角色的宽高限制，以及动态物体寻路等功能，基本上适应了大部分项目变化多端的需求。

我把写NavMesh的过程分成好几个部分，一一进行描述：

一.首先要理解NavMesh核心算法。NavMesh的核心算法就是用三角形代替传统寻路的方格，用计算拐点优化寻路路径来代替合并路径直线。
如下图1NavMesh寻路:
<img class="alignnone wp-image-91 size-full" src="/assets/uploads/2013/10/2303121.jpg" alt="" width="479" height="312" />
 
以及如下图2传统的方格寻路:
<img class="alignnone wp-image-94 size-full" src="/assets/uploads/2013/10/20131006173445.jpg" alt="" width="621" height="568" />
 
看到两者的差别了吧，NavMesh已三角形为寻路块，而传统以方格为寻路块。其实两者都使用A*寻路，但就是其网格生成不一样，导致当有大范围寻路时，其效率和要求也不一样。

二.NavMesh寻路中的路径优化之拐点计算。其实NavMesh中比较常用的是<strong>光照射线法，</strong>但这里不做详细介绍，光照射浅法详细内容地址:[http://www.cnblogs.com/neoragex2002/archive/2007/09/09/887556.html](http://www.cnblogs.com/neoragex2002/archive/2007/09/09/887556.html)
拐点计算优化路径就是到达目的地需要经过的一堆三角形中计算出最简洁的移动方式。其核心算法就是从当前点到另一个三角形中的点之间的线段，与这条线段相交的线段全部是路径所穿越的线段，就是拐点，把所有的拐点找出来，并得到一条最长的拐点，那个拐点就是最佳的拐点位置。
 
三.NavMesh类设计详解(这里只设计2D的寻路，对于3D方向的寻路，其实是可以2D寻路代替的)：

1.所有类都在同一的命名空间NavMesh内 namespace NavMesh
Triangle 三角形基础类
NavTriangle 寻路三角形类 (继承Triangle)
Line2D 线段类
Polygon 多边形类
Seeker 寻路主算法类

----------------------------------------- 让大家久等了 ------------------------------------

在寻路前，我们需要建立MESH三角形网格，这是NAV_MESH的重点之一。

1.首先我们先要画出一个范围来确定我们的可行走范围。

2.再在可行走范围中去添加不可行走的范围。

3.我们用多个多边形Polygon代替以上的范围，也就是说，一个大的可行走Polygon内包含了若干个小的不可行走的Polygon。

这是生成MESH前我们需要知道的生成范围，然后再由这些多边形的各个顶点来生成三角形网格,三角形网格的生成算法如下：

Step 1 :  用可行走Polygon的任意一条边作为起点，将其推入堆栈列表。到Step2.

Step 2:  从堆栈中推出一条边，在所有三角形中计算出边的DT点，构成约束Delaunay三角形，到Step3。如果没有DT点就重复做Step2，直到堆栈为空就结束整个程序。

Step 3:  将所构成的三角形，另两边做如下处理：检查堆栈中是否已存在，如果存在就删除该边，如果不存在就加入到堆栈中。

生成mesh后如图：（绿色的为多边形边框，蓝色的为寻路路径，红色的为编辑器选中的多边形）
<img class="alignnone size-full wp-image-376" src="/assets/uploads/2013/10/OD04RIKAWEMUTM_3MSBQ.jpg" alt="OD04[RIKAWEMUTM_3MS}B$Q" width="475" height="312" />

核心源码为：

``` c#
        /// <summary>
        /// 创建导航网格
        /// </summary>
        /// 所有阻挡区域</param>
        /// 输出的导航网格</param>
        /// <returns></returns>
        public NavResCode CreateNavMesh(List<Polygon> polyAll , ref int id , int groupid , ref List<Triangle> triAll)
        {
            triAll.Clear();
            List<Line2D> allLines = new List<Line2D>(); //线段堆栈

            //Step1 保存顶点和边
            NavResCode initRes = InitData(polyAll);
            if (initRes != NavResCode.Success)
                return initRes;

            int lastNeighborId = -1;
            Triangle lastTri = null;

            //Step2.遍历边界边作为起点
            {
				Line2D sEdge = startEdge;
                allLines.Add(sEdge);
                Line2D edge = null;

                do
                {
                    //Step3.选出计算出边的DT点，构成约束Delaunay三角形
                    edge = allLines[allLines.Count - 1];
                    allLines.Remove(edge);

                    Vector2 dtPoint;
                    bool isFindDt = FindDT(edge, out dtPoint);
                    if (!isFindDt)
                        continue;
                    Line2D lAD = new Line2D(edge.GetStartPoint(), dtPoint);
                    Line2D lDB = new Line2D(dtPoint, edge.GetEndPoint());

                    //创建三角形
					Triangle delaunayTri = new Triangle(edge.GetStartPoint(), edge.GetEndPoint(), dtPoint, id++ , groupid);
                    // 保存邻居节点
                    // if (lastNeighborId != -1)
                    // {
                    // delaunayTri.SetNeighbor(lastNeighborId);
                    // if(lastTri != null)
                    // lastTri.SetNeighbor(delaunayTri.ID);
                    // }
                    //save result triangle
                    triAll.Add(delaunayTri);

                    // 保存上一次的id和三角形
                    lastNeighborId = delaunayTri.GetID();
                    lastTri = delaunayTri;

                    int lineIndex;
                    //Step4.检测刚创建的的线段ad,db；如果如果它们不是约束边
                    //并且在线段堆栈中，则将其删除，如果不在其中，那么将其放入
                    if (!Line2D.CheckLineIn(allEdges, lAD, out lineIndex))
                    {
                        if (!Line2D.CheckLineIn(allLines, lAD, out lineIndex))
                            allLines.Add(lAD);
                        else
                            allLines.RemoveAt(lineIndex);
                    }

                    if (!Line2D.CheckLineIn(allEdges, lDB, out lineIndex))
                    {
                        if (!Line2D.CheckLineIn(allLines, lDB, out lineIndex))
                            allLines.Add(lDB);
                        else
                            allLines.RemoveAt(lineIndex);
                    }

                    //Step5.如果堆栈不为空，则转到第Step3.否则结束循环
                } while (allLines.Count > 0);
            }

            // 计算邻接边和每边中点距离
            for (int i = 0; i < triAll.Count; i++)
            {
                Triangle tri = triAll[i];
                //// 计算每个三角形每边中点距离
                //tri.calcWallDistance();

                // 计算邻居边
                for (int j = 0; j < triAll.Count; j++)
                {
                    Triangle triNext = triAll[j];
                    if (tri.GetID() == triNext.GetID())
                        continue;

                    int result = tri.isNeighbor(triNext);
                    if (result != -1)
                    {
                        tri.SetNeighbor(result , triNext.GetID() );
                    }
                }
            }

            return NavResCode.Success;
        }
```
 
这里对如何计算DT点进行一个说明：

Step1. 构造三角形的外接圆，以及外接圆的包围盒

Step2. 依次访问网格包围盒内的每个网格单元：

若某个网格单元中存在可见点 p, 并且 &ang;p1pp2 > &ang;p1p3p2，则令 p3=p，转Step1；

否则，转Step3.

Step3. 若当前网格包围盒内所有网格单元都已被处理完,也即C（p1，p2，p3）内无可见点，则 p3 为的 p1p2 的 DT 点.

核心源码为：

``` c#
/// <summary>
/// 找到指定边的约束边DT
/// </summary>
/// <param name="line"></param>
/// <returns></returns>
private bool FindDT(Line2D line, out Vector2 dtPoint)
{
    dtPoint = new Vector2();
    if (line == null)
        return false;

    Vector2 ptA = line.GetStartPoint();
    Vector2 ptB = line.GetEndPoint();

    List<Vector2> visiblePnts = new List<Vector2>();
    foreach (Vector2 point in allPoints)
    {
        if (IsPointVisibleOfLine(line, point))
            visiblePnts.Add(point);
    }

    if (visiblePnts.Count == 0)
        return false;

    bool bContinue = false;
    dtPoint = visiblePnts[0];

    do
    {
        bContinue = false;
        //Step1.构造三角形的外接圆，以及外接圆的包围盒
        Circle circle = NMath.CreateCircle(ptA, ptB, dtPoint);
        Rect boundBox = NMath.GetCircleBoundBox(circle);

        //Step2. 依次访问网格包围盒内的每个网格单元：
        //若某个网格单元中存在可见点 p, 并且 &ang;p1pp2 > &ang;p1p3p2，则令 p3=p，转Step1；
        //否则，转Step3.
        float angOld = (float)Math.Abs(NMath.LineRadian(ptA, dtPoint, ptB));
        foreach (Vector2 pnt in visiblePnts)
        {
            if (pnt == ptA || pnt == ptB || pnt == dtPoint)
                continue;
            if (!boundBox.Contains(pnt))
                continue;

            float angNew = (float)Math.Abs(NMath.LineRadian(ptA, pnt, ptB));
            if (angNew > angOld)
            {
                dtPoint = pnt;
                bContinue = true;
                break;
            }
        }

        //false 转Step3
    } while (bContinue);

    //Step3. 若当前网格包围盒内所有网格单元都已被处理完，
    // 也即C（p1，p2，p3）内无可见点，则 p3 为的 p1p2 的 DT 点
    return true;
}
```

为了让各位能更容易读懂此文，此文仍会继续补充。

现在我将所有源码都存放在了Github上，请各位跟随我到Github去取源码：[https://github.com/luzexi/Unity3DNavMesh](https://github.com/luzexi/Unity3DNavMesh)
 
转载请注明出处：http://www.luzexi.com
