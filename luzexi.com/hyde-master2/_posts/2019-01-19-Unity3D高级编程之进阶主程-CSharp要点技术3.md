---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(三) 浮点数的精度问题
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### 浮点数的精度问题

===

很多同学在平时的项目中会用double类型来替代float来提高精度，错误的认为double可以解决精度问题。其实平常我在编程中极少使用double类型，浮点数计算在我们大多数项目中并没有使用到特别的科学计算部分，所以float基本都够用，其实double也同样有精度问题，无论怎么样都是无法避免精度导致的在逻辑中的不一致的问题。

我们不妨比较下float 与 double 来来看看它们有什么不同。float和double所占用的比特位数不同会导致精度不同，float是32个位占用4字节而double是64个位占用了8个字节，因此它们在计算时也会引起计算效率的不同。

实际工作中我们很多时候想试图通过使用double替换float来解决精度问题，最后基本都会以失败告终。因此我们要认清精度这个问题的根源，才能真正解决问题，我们先来看下浮点数在内存中到底是如何存储的。

计算机只认识 0 和 1，无论什么形式的数字都是一样的，无论是整数还是小数在计算机中都以二进制的方式储存在内存中的，那么浮点数是以怎样的方式来存储的呢我们来看下。根据 IEEE 754 标准，任意一个二进制浮点数 F 均可表示为：F = (-1 ^ s) * (1.M) * (2 ^ e)。

从公式中可以看出，它被分为了3个部分，符号部分即 s部分 和 尾数部分即 M部分 和 阶码部分即 e部分，s为sign符号位0或1，M为尾数指具体的小数用二进制表示，它完整的表示为1.xxx中的1后面的尾数部分因为称它为尾数，e是比例因子的指数是浮点数的指数。

	(float1.png)

	(float2.png)

上图所示的是32位和64位的浮点数存储结构，即float和double的存储结构。不论是32位浮点数还是64位浮点数，它们都由S、E、M三部分组成并以相同的公式来计算得到最终值。

其中E的阶符采用隐含方式，即采用移码方法来表示正负指数。移码方法对两个指数大小的比较和对阶操作比较方便，因为阶码的值大时其指向的数值也是大的，这样更容易计算和辨认。移码（又叫增码）是符号位取反的补码，例如float的8位价码，应将指数e加上一个固定的偏移值127（01111111），即 e 加上 127才是存储在二进制中的数据。

尾数M则更为简单点，它只表示1后面的小数部分，而且是二进制直译的那种，然后再根据阶码来平移小数点，最后根据小数点的左右部分得出整数部分和小数部分的数据。

以9.625 为例，转换为二进制为 1001.101 可以表达为 1.001101×(2^3)，因此M尾数部分为 00110100000000000000000 去掉了小数点左侧的 1，并用 0 在右侧补齐。

其中表示9.625时的 1001.101 的小数部分为什么是“101”，因为整数部分采用 "除2取余法"来得到从10进制转换到2进制的数字，而小数部分则采用 "乘2取整法"得到二进制。因此这里的小数部分 0.625 乘 2 为 1.25 取整得到 1，继续 0.25 乘以 2 为 0.5 取整得到 0，再继续 0.5 乘以 2 为 1 取整为 1，后面都是 0 不再计算，因此得到 0.101 这个小数点后面的二进制。

再以 198903.19 这个数字为例，先形象直接的转成二进制的数值为，整数部分采用 "除2取余法"，小数部分采用 "乘2取整法"，并把整数和小数部分用点号隔开的到

		110000100011110111.0011000010100011（截取 16 位小数）

以1为首平移小数点后得到 1.100001000111101110011000010100011 * (2 ^ 17)（平移17位）即

		198903.19(10) = (-1 ^ 0) * 1.100001000111101110011000010100011 * (2 ^ 17)。

从结果可以看出，小数部分 0.19 转为二进制后，小数位数超过 16 位（已经手算到小数点后 32 位都还没算完，其实这个位数是无穷尽的），因此这里导致浮点数有诸多精度的问题，它很多时候无法准确的表示数字，甚至非常不准确。

###### 浮点数的精度问题可不只是小数点的精度问题，随着数值越来越大，即使是整数也开始会有相同的问题，因为浮点数本身是一个 1.M * (2 ^ e) 公式形式得到的数字，当数字放大时，M的尾数的存储位数没有变化，能表达的位数有限，自然越来越难以准确表达，特别是数字的末尾部分越来越难以准确表达。

人们总是觉得精度问题看起来好像很难碰到，但其实并非如此，实际开发中常常碰到而且确实很头疼，如果没有这部分的知识结构很容易在查看问题时陷入无尽的问号。那么我们来看看哪些情况会碰到这类问题。

###### 1.数值比较不相等

我们在写程序时经常会遇到到阀值触发某逻辑的情景，比如某个变量，需要从0开始加，每次加某个小于0.01的数，加到刚好0.23时做某事，到0.34时做另外一件事，到0.56时再做另一件。

这种精确定位的问题，就会遇到麻烦。因为浮点数在加减乘数时无法完全准确定位到某个值，就会在出现，要么比0.23小，要么比0.23大，永远不会刚刚与0.23相等的时候，这时我们不得不放弃 ‘==’ 这个等于号而选择‘>’大于号或者‘<’小于号来解决这种问题的出现。

如果一定要用等于来做比较，则需要有一个微小的浮动区间，即 ABS(X-Y) < 0.00001 时认为 X 和 Y 是相等的。

###### 2.数值计算不确定

比如 x = 1f，y = 2f，z = 1f /5555f * 11110f，如果 x / y <= 0.5f 时做某事，那么理论上说 x / z 也能通过这个if，因为在我们看来z 就等于 2 和 y是一样，但实际上未必是这样的。浮点数在计算时由于位数的限制无法得到精确的数值而是一个被截断的数值，因此 z 的计算结果有可能是0.4999999999991，当x / z 时，结果有可能得到大于0.5。

这让我们很头疼，在实际编码中，我们经常会遇到这样的情况，在外圈的if判断成立，理论上同样的结果只是公式不同，它们在内圈的if判断却可能不成立，使得程序就出现异常行为，因为看起来应该是得到同样的数值，但结果却不一样。

###### 3.不同设备计算结果不同

不同平台上的浮点数计算也有所偏差，由于不同设备上CPU的计算方式不同，导致相同的公式在不同的设备上计算出来的结果有略微的偏差。

面对这些精度上的问题我们该怎么办，下面就来看看我们有哪些解决方案。

###### 我们可以简单点，只计算一次，认定这个值为准确值，只用这个变量结果做判断，也省去了多次计算浪费的CPU。

由于多次计算相同的结果可能造成精度问题，不如只计算一次，只用一次计算的结果把它当做唯一确定性结果，而不使用多次计算得到的结果。排除了多次结果不同导致的问题。

由于多次看似相等的计算其实得到的结果有可能不同，使得问题变得更复杂，比如上面所说的，1f / 2f 的结果，用 1f / (1f / 5555f * 11110f ) 来表示得到的结果不一样导致问题变得不可控。不如只使用一次计算，不再进行多次计算，认定这次的结果的数值为准确数值，只用这个浮点数值当做判断的标准。

我在编程中也常用这种方法，多次计算同样的结果也浪费了不少CPU，这种方法简单实用，当前其他方法由于复杂度高，项目需要快速推进，当前遗留的不是很重要的问题的话，可以推延到整体架构演变时再一步步细化。

当然这种方法使用范围比较小，可能不适用你所在的项目我们可以继续看看其他的解决方案。

###### 我们可以改用int或long型来替代浮点数。

浮点数和整数的计算方式都是一样的，只是小数点部分不同而已，那么完全可以把浮点数乘以10的幂次得到更准确的精度部分的数字，把自己需要的精度提上来用整数表示。

比如保留3位精度，所有浮点数都乘以1万来存储(因为第四位不是很准确了)，1.5变成了15000的整数，9.9变成了99000整数存储。

这样整数 15000 乘以 99000 得到的结果，与，整数30000 除以 2 再乘以 99000 得到的结果是完完全全相等的。

再复杂点 原来 2.5 / 3.1 * 5.1 与 0.8064 * 5.1，两者都约等于 4.1126，用整数替代，2500 / 31 * 51 与 80 * 51，等于 4080，把4080看作 4.08 虽然精度出现问题，但是前两者结果不一致，而后两者结果完全相同，使用整数来代替小数使得一致性得到了保证。

如果你觉得用整数做计算精度问题比较大，我们可以再扩大数值上10的幂次，来看看扩大后如果是 250000 / 31 * 51 就等于 411290，是不是发现精度提高了。但问题又来了，乘以10的幂次来提高精度时，当浮点数值比较大时就会超出了整数的最大上限2 ^ 32 - 1或者2 ^ 64 - 1 的问题。

如果你觉得精度可以接受，并且数值计算的范围肯定会被确定在32位或64位整数范围内，则可以用这种int和long的方式来代替浮点数。

###### 用定点数保持一致性并缩小精度问题

浮点数在计算机中的表示方法是用 V = (-1)^s x (1.M) x 2^(e) 这样的公式表示的，也就是说浮点数的表达其实是模糊的，它用了另一个数乘以2的幂次来表示当前的数。

定点数则不同，它把整数部分和小数部分拆分开来，都用整数的形式表示，这样一来计算和表达都使用整数的方式。由于整数的计算是确定的，这样就不会存在误差，缺点是由于拆分了整数和小数，两个部分都要占用空间，所以受到存储位数的限制，占用字节多了通常使用64位的long型整数结构来存储定点数，计算的范围也会相对缩小些。

与浮点数不同的，用定点数来做计算就能保证在各设备上的计算结果一致性，C# 有种整数类型叫 decimal 它并非基础类型，是基础类型的补充类型，是C# 额外造出来的一种类型，可以认为是造了一个类作为数字实例并重载了操作符，它拥有更高的精度却比float范围还要小。它的内部实现就是定点数的实现方式，我们完全可以把它看作定点数来操作。

C# 的 decimal 类型数值有几个特点需要我们重点关注一下，它占用128位的存储空间即一个decimal变量占用16个字节，相当于4个int整数大小，或2个long型长整数大小，比double还要大1倍。它的数值范围在 ±1.0 × 10^28 到 ±7.9 × 10^28之间，这么大的的占用空间却比float的取值范围还小。decimal 精度比较大，精度范围在 28 个有效位，另外任何与它一起计算的数值都必须先转化为 decimal 类型否则就会编译报错，数值不会隐式的自动转换成 decimal。

看起来好用的 decimal 却不是大部分游戏开发者的首选选择。使用 C# 自带的 decimal 定点数在使用时存在诸多问题，最大的问题是无法和浮点数随意的互相转换，因此在计算上也会需要进行一定的封装，要么提前对float处理，要么在 decimal 基础上封装一层外壳(类)以适应所有数值的计算。精度过大导致CPU 计算消耗量大，128位的变量28位的精度范围，计算起来实在是比较大的负荷，如果大量用于程序内的逻辑计算则CPU就会不堪重负。内存也是同样的，大量使用时会使得堆栈内存直线飙升，这也间接的增大了CPU的消耗。因此它只适用于财务和金融领域的软件，对于游戏和其他普通应用来说实在不太合适，其根源是不需要这么高的精度浪费了诸多设备资源。

实际上大部分项目都是自己实现定点数的，具体实现如前面所说的那样：把整数和小数拆开来存储，用两个int整数分别表示整数部分和小数部分，或者用long长整型存储(前32位存储整数，后32位存储浮点数)，long型存储会更好因为这样就确保定点数的内存就是连续的。这样无论整数还是小数部分都用整数表示，并封装在类中，继而我们需要重载(override)所有的基本计算和比较符号，包括+、-、*、/、==、!=、>、<、>=、<=、这些符号都需要重载，重载范围包括 float浮点数、double双精度、int整数、long长整数等。除了以上这些，为了能更好的融合定点数与外部数据的逻辑计算，我们还需要为此写一些额外的定点库，包括定点数坐标类，定点数Quaternion类等用于定点数的扩展。

看起来比较困难其实并不复杂，只要耐下心来写都会发现不是难事，把定点数与其他类型数字的加减乘除重写一下，如果涉及到更多的数学运算，则再建立一个定点数的数学库，存放一些数学运算的函数，再用写好的定点数类去写些用于扩展的逻辑类仅此而已，都是只要花点时间就能搞定的事，github上也有很多定点数开源的代码，大可以下载下来参考着写，或者把它从头到尾看一遍，把它改成适合自己项目的。

###### 最耗的办法，用字符串替代浮点数。

如果想要精确度很高很高，那么就可以用字符串代替浮点数来计算结果。但它的缺点是CPU和内存的消耗特别大，如果只是少量使用高精度计算还是可以的。

我记得以前在大学里做ACM题目时，就有这种方式来检验程序员的逻辑能力和考虑问题的全面性的题目，题目很简单A * B 或 A - B 或 A + B 或 A / B 输出结果，精度要求在小数点后100位。我们把中小学算术的笔算方式写入到程序里去，把字符串转化为整数，并用整数计算当前位置，接着再用字符串形式存储数字，这样的计算方式就完全不需要担心越界问题，还能自由的控制精度。

缺点是很消耗CPU和内存，比如123456.78912345 * 456789.2345678，这种类型的计算使用字符串代替浮点数，计算一次相当于计算好几万次的普通浮点数计算量。所以如果程序中对精度要求很高，且计算的次数不大，这种方式可以放在考虑范围内。

###### 最差的办法，提高期望值。

如果 1.55f / 2f 有可能等于 0.7749999 而无法达到 0.775 的目标值时，我们不妨在计算前多加个0.01，使得 1.56f / 2f，这样就大概率保证超出 0.775 的结果目标了。

###### 怎么想的？为什么能这么做？因为我们做了假设。

我们脑袋中假设结果是模糊的，这个结果范围就是有可能是 0.7751，也有可能是 0.7749。正常情况下，由计算机的寄存器和CPU决定到底会得到什么结果，但我们不想由它来决定。我们的期望是，宁愿高一点点，跨过这道门槛，也不要少一点点，被门槛拦在外面。

如果正常浮点数的精度不能给我们安全感，那么我们就自己给自己安全感。提高自己的数值一点点，从而降低门槛跨过去的门槛，差不多的就差一点点的都当做是跨过门槛的。于是就有了 (X + 1) / Y 或者 (X + 0.001) * Y 的写法来度过'精度危机'。


