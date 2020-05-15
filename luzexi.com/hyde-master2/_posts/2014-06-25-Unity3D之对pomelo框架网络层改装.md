---
layout: post
status: publish
published: true
title: Unity3D之对pomelo框架网络层改装
description: "unity3d pomelo 架构"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 367
wordpress_url: http://www.luzexi.com/?p=367
date: !binary |-
  MjAxNC0wNi0yNSAxNDoyNzoyNSArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wNi0yNSAwNjoyNzoyNSArMDgwMA==
tags:
- Unity3D
- 前端技术
---
最近了解了下网易的开源服务器框架pomelo，github地址：[https://github.com/NetEase/pomelo](https://github.com/NetEase/pomelo)发现其中它封装的u3d的网络层部分有线程安全问题，几乎不能直接u3d项目，所以对其进行了2次封装，让他可以真正用于u3d项目。

封装后的源码也同样放在我的github上：[https://github.com/luzexi](https://github.com/luzexi) 供大家参考。

下面写下pomelo的u3d网络层源码问题和我对源码的封装过程：
pomelo网络通信方式分:原始socket和websocket两种，这两种方式由项目需要而选择其中一种来使用。pomelo种对u3d网络通信封装的最主要问题是线程中逻辑句柄的调用，因为u3d对它本身主线程意外的线程调用本身api是非常排斥，当你在其他线程中运行u3d的api会直接被cut掉，所以我们需要将其调用游戏逻辑句柄的那部分放到u3d主线程中去。

如何将线程通信后的游戏句柄调用部分放到u3d主线程中去：

1.网络通信是由数据包为单位来驱动游戏逻辑句柄的，所以只要加入队列概念，在收到数据包时由原来的直接调用句柄，改为先推入到队列中，等待处理。

``` c#
#if LUZEXI
        private System.Object m_cLock = new System.Object();    //the lock object
        private const int PROCESS_NUM = 5;  //the process handle num per fps
        private Queue<byte[]> m_seqReceiveMsg = new Queue<byte[]>();    //the message queue
        private TranspotUpdate m_cUpdater = null;   //The Updater of the message queue
#endif
```

2.在u3d主逻辑中增加一个专门处理网络通信逻辑句柄的类。这个类在通信开始时就加入到u3d主逻辑中去，当通信结束(无论是正常还是异常结束)时在u3d主逻辑中销毁。在主逻辑中不断的或者说每帧都去查看网络通信层队列中有没有等待处理的句柄，有就取出来处理。这样就由等待队列串联起来了几个线程的共同合作。ps:对队列做防死锁操作

``` c#
#if LUZEXI
        internal void Update()
        {
            for( int i = 0 ; i<PROCESS_NUM && i< this.m_seqReceiveMsg.Count; i++)
            {
                lock(this.m_cLock)
                {
                    byte[] data = this.m_seqReceiveMsg.Dequeue();
                    this.messageProcesser.Invoke(data);
                }
            }
        }
#endif
```

{% include advertisement_content.html %}

3.拆分网络通信事件。将网络通信事件拆分为OnConnect , OnError , OnDisconnect , 这3个事件是最常见的通信事件。这3个事件也必须在主线程中调用，但这3个事件又不是以网络数据包为基础，所以需要在专门处理通信逻辑的类中加入3种状态start , error , disconnect.当网络通信层出现这三种事件时，将状态切过去（ps:而不是直接调用句柄）,然后再由专门处理通信逻辑的类在下一帧去调用对应的事件句柄。

``` c#
using UnityEngine;
using System;
using System.Collections.Generic;
using SimpleJson;

//  TranspotUpdate.cs
//  Author: Lu Zexi
//  2014-6-20
namespace Pomelo.DotNetClient
{
    /// <summary>
    /// Transpot updater.
    /// </summary>
    public class TranspotUpdate : MonoBehaviour
    {
        private enum STATE
        {
            NONE = 0,
            START = 1,
            RUNING = 2,
            CLOSE = 3,
            DESTORY = 4,
        }
        private STATE m_eStat = STATE.NONE; //the state of the transpotUpdate
        private Action m_cUpdate;   //update action
        private List<Action<JsonObject>> m_cOnDisconnect;   //on the disconnect

        /// <summary>
        /// Init this instance.
        /// </summary>
        internal static TranspotUpdate Init()
        {
            GameObject obj = new GameObject(<span class="string">"Socket");
            TranspotUpdate trans = obj.AddComponent<TranspotUpdate>();
            return trans;
        }

        /// <summary>
        /// set the update of the process message action
        /// </summary>
        /// <param name="update">Update.</param>
        internal void SetUpdate( Action update )
        {
            this.m_cUpdate = update;
        }

        /// <summary>
        /// set the disconnect evet.
        /// </summary>
        /// <param name="ondisconnect">Ondisconnect.</param>
        internal void SetOndisconnect( List<Action<JsonObject>> ondisconnect )
        {
            this.m_cOnDisconnect = ondisconnect;
        }

        /// <summary>
        /// close the updater
        /// </summary>
        internal void Close()
        {
            this.m_eStat = STATE.CLOSE;
        }

        /// <summary>
        /// Start this update.
        /// </summary>
        internal void _Start()
        {
            this.m_eStat = STATE.START;
        }

        /// <summary>
        /// destory the gameobject.
        /// </summary>
        internal void _Destory()
        {
            if(this.m_eStat != STATE.CLOSE)
                this.m_eStat = STATE.DESTORY;
        }

        /// <summary>
        /// Fixeds the update.
        /// </summary>
        void FixedUpdate()
        {
            switch(this.m_eStat)
            {
            case STATE.START:
            case STATE.RUNING:
                if(this.m_cUpdate != null )
                {
                    this.m_cUpdate();
                }
                break;
            case STATE.CLOSE:
                if(this.m_cOnDisconnect != null )
                {
                    foreach(Action<JsonObject> action in this.m_cOnDisconnect)
                        action.Invoke(null);
                }
                this.m_cOnDisconnect = null;
                this.m_eStat = STATE.DESTORY;
                break;
            case STATE.DESTORY:
                this.m_eStat = STATE.NONE;
                GameObject.Destroy(gameObject);
                break;
            }
        }
    }
}
```

转载请注明出处：http://www.luzexi.com
