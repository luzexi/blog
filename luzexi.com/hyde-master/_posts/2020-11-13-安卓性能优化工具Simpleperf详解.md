---
layout: post
status: publish
published: false
title: 安卓性能分析工具Simpleperf详解与应用
description: "unity3d gameplay 性能优化 总结"
excerpt_separator: ===
tags:
- Unity3D
- 前端技术
---


### 本文关注三个问题：Simpleperf的工作原始里是什么？Simpleperf该如何使用？它如何在Unity项目上使用？

关注Simpleperf的缘由是，虽然一直使用Xcode Intruments 、 Unity profiler 、Perfdog、以及一些自制的工具（自定义打点+内存快照）来做性能分析，但一直缺少专门针对安卓设备的的函数耗时分析。偶尔间搜到Simpleperf这么好的分析工具，于是开始研究 Simpleperf，最后把它用到项目中去并且建立起日常性能监控流水线。

#### Simpleperf是Android开源项目（AOSP）的一部分， 是一个 CPU 性能剖析工具，可以剖析 Android 客户端 Java 和 C++ 代码，是 Android NDK 工具的一部分。其包含两部分：Simpleperf可执行文件（命令行）和python脚本。

python脚本整合了Simpleperf可执行文件（命令行）和adb的功能，让Simpleperf使用起来更加方便快捷。

Simpleperf中可执行文件：

![1](/assets/simpleperf/image7.png)
![2](/assets/simpleperf/image21.png)

Simpleperf中整合了命令行的Python脚本：

![3](/assets/simpleperf/image23.png)

#### Simpleper可执行文件支持安卓5.0及以上的系统，但python脚本只支持安卓7.0及以上的系统。

这里的Python脚本不仅封装了命令行可执行文件的操作，同时提供了生成测试数据报告，线程消耗图，火焰调用图等功能。由于数据报告生成方面，Simpleperf命令行本身只支持文本数据报告的生成，因此要生成可视化的数据报告，还得依靠python。

我用一个demo做实验，demo的程序就是每帧运行1000w次浮点数和随机数计算，作为实验性能分析数据：

![4](/assets/simpleperf/image3.png)

以下展示的是性能报告生成后的火焰图和文本数据报告：

![5](/assets/simpleperf/image19.png)

![6](/assets/simpleperf/image12.png)

![7](/assets/simpleperf/image9.png)

#### 所以我把数据分析和数据报告生成给拆分开来，直接用可执行文件去执行性能分析的任务，让Simpleperf可以支持更低的安卓系统。性能分析完毕后，再用python脚本对所得的数据文件来生成可视化的数据报告，以及文本数据报告。

#### 其中，数据报告也分成两部分，一部分是文本数据报告，在生成文本数据报告后，对文本数据报告中的数据再加工和筛选，筛选出关键的信息上传到性能数据平台作为每日的性能日常报告，另一部分是火焰图和消耗图的数据报告，用于查看更细致的函数消耗分析。


下面就来详细介绍一下，如何使用Simpleperf来做性能分析。

### 功能概要

Simpleperf主要功能分为事件摘要（stat），记录样本(record)和生成数据报告(report)三个功能。stat功能给出了在一个时间段内被分析的进程中发生了多少事件的摘要。record功能必须在Android系统中运行，当Simpleperf运行分析时会不断将数据写入到性能数据文件，所以它可以随时停止，随时拷贝分析数据文件。分析完毕后我们可以需要将输出数据文件拷贝到PC上，再使用report功能解析成数据报告。

### 前提条件

Simpleperf需要有权限去做性能采样，所以想要使用Simpleperf做性能分析，需要满足4个条件中的一个就够了。

1.debug版本，即在manifest中设置了android::debuggable=”true”，并且允许JNI 测试，并且C/C++没有被编译器优化过。也就是Unity构建的Development Build版本。

2.release版本，如果安卓10以上则需要manifest中加入<profileable android:shell="true" />就可以。

3.release版本，如果是安卓8以上则在manifest中加入<application android::debuggable="true" ...>后，把wrap.sh 放入 lib/arch 文件夹中。 wrap.sh 会在没有debug标志给ART的情况下跑app。

4.release版本，你的手机是root过的，有root权限Simpleperf就会畅通无阻。

### 底层原理

现代CPU具有一个硬件组件，称为性能监控单元(PMU)。PMU具有一些硬件计数器，计数一些诸如经历了多少次CPU周期，执行了多少条指令，或发生了多少次缓存未命中等事件。

Linux内核将这些硬件计数器包装到硬件perf事件 (hardware perf events)中。此外，Linux内核还提供了独立于硬件的软件事件和跟踪点事件。Linux内核通过 perf_event_open 系统调用将这些都暴露给了用户空间，这正是simpleperf所使用的机制。

![8](/assets/simpleperf/image25.png)

linux系统在各个层都封装了一套性能接口。图中perf字样的接口主要位于CPU层，系统调用层，系统库层，调度层，内存层中。我们通过simpleperf list可以查看受支持的事件类型，有硬件事件类型和软件事件类型之分。

![9](/assets/simpleperf/image6.png)

![10](/assets/simpleperf/image1.png)

硬件事件借助现代CPU中的PMU（性能监控单元）部件实现采样，比如cpu-cycle，cache-miss等硬件级数据，kernel会开启PMU的计数器去采集对应进程的数据。

##### PMU是CPU中的部件，专门用于性能监控，CPU在运行时可以收集关于处理器和内存的各种统计信息。对于处理器来说这些统计信息中的事件非常有用，这样我们可以利用它们来调试或者剖析代码。

##### 软件事件则是系统内核kernel层自行实现，统计操作系统相关性能事件在各个功能模块中，如内存对齐断层事件、线程context-switch上下文切换事件、cpu时钟事件、cpu迁移事件、页断层事件等等。

事实上SimplePerf就是通过linux系统的perf_event_open接口来获得这些perf数据。

![11](/assets/simpleperf/image17.png)

linux内核perf_event_open接口 https://www.man7.org/linux/man-pages/man2/perf_event_open.2.html


### 那么stat和record是如何工作的呢？

#### Stat命令给出时

1.simpleperf调用linux系统接口启用perf分析。

2.设置监控事件和周期。

3.Linux 内核在调度到被分析进程时启用计数器。

4.simpleperf从内核读取计数器，并报告计数器摘要。

cmd_stat.cpp 471行 StatCommand::Run

https://android.googlesource.com/platform/system/extras/+/master/simpleperf/cmd_stat.cpp


#### Record 命令给出时

1.Simpleperf通过对linux内核接口调用加入监控，锁定监控进程对象。

2.调用linux系统接口启用perf分析。

3.Simpleperf在simpleperf 和 linux 内核之间创建共享映射缓冲区。

4.设置需要监控的事件和采样频率，告诉linux 内核将样本数据转储到映射缓冲区。

5.开启线程不断监控共享映射缓冲区，从映射缓冲区读取数据并写入到数据文件。

cmd_record.cpp 455行 RecordCommand::PrepareRecording

https://android.googlesource.com/platform/system/extras/+/master/simpleperf/cmd_record.cpp

### 这里有2个重要的节点我们深入了解下，这两个重点能映射出Simpleperf是如何在Linux系统工作的？

### 第一个，它是如何与Linux操作系统通信的？

通常进程之间通信都是通过共享内存进行，Simpleperf通过创建映射缓冲区来与系统共享一块内存。那么它是怎么创建和映射的呢？

我们来看看源码中的关键函数，CreateMappedBuffer创建映射缓冲区，其中mmap为Linux系统接口，调用它使得进程之间通过映射同一块内存(或文件)实现共享内存。
Simpleperf调用mmap创建指定的共享内存大小，设置读写权限，设置共享标记，以及设置通道句柄。

![12](/assets/simpleperf/image13.png)

有了共享内存，接着Simpleperf需要告诉与系统把性能数据放入共享缓冲中去。

其中ioctl为Linux系统接口，是设备驱动程序中对设备的I/O通道进行管理的函数。所谓对I/O通道进行管理，就是对设备的一些特性进行控制，例如串口的 传输波特率、马达的转速等等。

Simpleperf调用ioctl告诉Linux系统将性能数据写入通道缓存。

![13](/assets/simpleperf/image2.png)

ioctl调用格式为：

```	c
int ioctl(int fd, ind cmd, …)；
```

我们可以通过ioctl的命令码(cmd)告诉Linux驱动程序我们想做什么，至于怎么解释这些命令和怎么实现这些命令，这都是驱动程序要做的事情。
现在我们使用cmd命令码PERF_EVENT_IOC_SET_OUTPUT，告诉系统让内核把性能相关事件数据放入指定缓冲中。

![14](/assets/simpleperf/image20.png)

### 第二个，它是如何得到Linux操作系统的性能信息的？

Simpleperf通过创建一个线程来监控数据并记录数据，它就是RecordReadThread。

![14](/assets/simpleperf/image11.png)

这个线程也会不断的接收用户的命令输入，来暂停和恢复数据记录。

创建线程后，接着实例化需要监控的事件，并开始记录和监控。

![15](/assets/simpleperf/image10.png)

此线程不断循环监控共享内存的状况，同时记录性能数据：

![16](/assets/simpleperf/image24.png)

#### 第一个问题中我们说，Simpleperf通过Linux系统接口mmap申请共享内存，再通过Linux系统接口ioctl告诉内核把性能监控的数据放在共享内存中。由于Linux系统在各个层都封装了一套perf性能接口，于是在接下来的程序运行中，Linux内核将这些perf事件 (perf events)数据放入Simpleperf设置的共享内存中。而Simpleperf又开启了一个线程来不断监控是否有数据加入，并将它们放入自己缓存中去，最后再通知主线程适当的时候将这些数据写入本地文件。

### 性能分析执行

### record 命令行说明

![17](/assets/simpleperf/image16.png)


### 性能分析命令执行步骤

现在我们用Simpleperf命令行的形式来对安卓系统进行C/C++的性能分析：

1.将simpleperf可执行文件放入安卓系统

``` sh
adb push .\ndk21\simpleperf\bin\android\arm64\simpleperf /data/local/tmp
```

2.将可执行文件设置为可执行文件

``` sh
adb shell chmod a+x /data/local/tmp/simpleperf
```

3.开始执行分析，这里设定一个99秒的持续时间。也可以设定为99999秒，然后用杀进程的方式来终止分析。因为我们前面说过，simpleperf是逐步写入数据文件的，数据文件随时保持完整性。

``` sh
adb shell /data/local/tmp/simpleperf record -o /data/local/tmp/perf.data -g --app com.xxx.xxx --duration 99 -f 800
```

4.停止分析

``` sh
adb shell pkill -l 2 simpleperf
```

5.将分析数据文件拷贝出来

``` sh
adb pull /data/local/tmp/perf.data .\report_data\report\perf.data
```

### 生成数据报告步骤

有了Simpleperf生成的性能数据后，我们就可以对这份性能数据文件进行分析并生成数据报告。

#### 生成报告前，我们首先需要准备符号表。

#### 为什么要准备符号表呢，因为这样就可以通过，分析数据中调用地址映射到具体的函数名。这些都由Simpleperf帮我们完成，只是我们需要为此准备这个带符号表的so。

Unity引擎带符号表so的放在Unity的安装目录下

![1](/assets/simpleperf/image8.png)

打完apk包后带符号的il2cpp的so在项目的Temp目录下面：

![1](/assets/simpleperf/image5.png)

有了这些准备接下我们来开始讲生成数据报告的操作步骤：

![1](/assets/simpleperf/image14.png)

1.准备符号表

把包地址打印出来，因为真正的包地址可能会在你知道的包名后面加些字符串

``` sh
adb shell pm path com.xxxx.xxx > tmp_appinfo.txt
```

把符号表拷贝过去到./binary_cache/data/app/com.xxx.xxxx/lib/arm(或者arm64)/里去

``` python
python:
cp_src1 = 'libil2cpp.so.debug'
cp_target1 = './binary_cache/data/app/com.xxx.xxxx/lib/arm/libil2cpp.so'
if os.path.exists(cp_src1):
    shutil.copy(cp_src1, cp_target1)
```

这里只拷贝了il2cpp的，至少还需要拷贝带符号的libunity.so。符号表是simpleperf用来查找调用函数名的。

#### Simpleperf是怎么查到函数名的呢，它是通过调用地址与符号表中的函数名对应关系来获得所调用的函数名的。我们可以用Simpleperf解析后数据，打开文本数据获得调用地址，来查看到底是怎么回事。

举个例子，我们查看数据报告中的88.31%的耗时占比函数调用库指令地址为83cb7c：

![1](/assets/simpleperf/image18.png)

在汇编工具IDA中加载libil2cpp带符号的so文件，查看到83cb7c地址恰好是在Random调用地址上：

![1](/assets/simpleperf/image4.png)

也就是说，有88.31%的消耗在Random.Range这个函数上。Simpleperf就是通过指令调用地址来获取符号表中的函数名的，通过调用地址和符号表上的函数名对齐后得到最终的函数名。

2.生成普通数据报告
   python .\ndk21\simpleperf\report.py -i .\perf.data -o .\report.txt -n --full-callgraph --symfs .\binary_cache

3.生成调用者耗时分布数据报告
   python .\ndk21\simpleperf\report.py -i .\perf.data -o .\report.caller.txt -g caller --full-callgraph --symfs .\binary_cache

4.生成被调用者耗时分布数据报告
   python .\ndk21\simpleperf\report.py -i .\perf.data -o .\report.callee.txt -g callee --full-callgraph --symfs .\binary_cache

最后生成的文本数据如下图：

![1](/assets/simpleperf/image9.png)

### 生成火焰图和消耗图

1.用simpleperf提供的report_html.py生成可视图

python .\ndk21\simpleperf\report_html.py -i .\perf.data -o .\report.html --binary_filter .\binary_cache

2.拷贝可视图网页需要用到的js文件

copy  ".\ndk21\simpleperf\report_html.js" ".\report_html.js"

![1](/assets/simpleperf/image12.png)

### 日常性能监控流程建设：

那么我们是否可以将Simpleperf运用到日常性能监控中去呢？

经过Simpleperf的性能分析后，我们得到了性能数据文本数据，我们可以对这个文本类型的数据文件再解析，然后将我们需要的重要的函数性能数据加入到后台数据库中，并在每日自动化性能测试报告中体现出来。

![1](/assets/simpleperf/image22.png)

我们来看看上图中，生成出来的性能数据格式。根据性能数据格式，百分比、采样次数、线程名、库文件名、调用函数名的格式，我们可以将数据解析到内存中，并用http的方式上传到web后台。web后台对数据加工后，可以用趋势图的形式在后台页面上展现出来。例如下面的样例图：

![1](/assets/simpleperf/image15.png)


参考文献：

Simpleperf官方地址：https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/README.md#executable-commands-reference

Simpleperf源码地址：https://android.googlesource.com/platform/system/extras/+/master/simpleperf/

《SimplePerf 安卓客户端性能剖析及自动化性能测试》http://km.oa.com/articles/show/466087?kmref=search&from_page=1&no=1

《Simpleperf介绍》https://blog.csdn.net/tq08g2z/article/details/77311712


