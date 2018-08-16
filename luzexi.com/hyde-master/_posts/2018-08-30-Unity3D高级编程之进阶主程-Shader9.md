


3.	有哪些常用的画面风格。
卡通风格，次世代风格，素风格，漫反射风格
https://www.zhihu.com/question/22530411

4.	有哪些固定方法编写图形渲染算法
(https://blog.uwa4d.com/archives/USparkle_Shader.html，https://blog.uwa4d.com/archives/usparkle_cartoonshading.html)
http://www.unity.5helpyou.com/2992.html

5.	实战练习shader编写。
6.	优化
*高光和反射时对镜头做优化
*类型：漫反射，镜面反射，自发光(高光)，法线贴图，凹凸贴图，环境闭塞贴图(光照的吸收情况)，细节图，贴花图，描边，透明（硬裁切Cutout，透明Transparent，虚化Fade），裁切，lightmap，顶点偏移
*算法：加，减，乘，除，点积，叉积，反射，归一化，转换，投影，N幂，对数，指数，alpha混合，屏幕点，世界点，相对物体点，顶点色，顶点等。
*清除，合并多余的编译指令，采样UV定义，变量
*shader提前解析，在初次加载时进行解析
*少用if
*少用高级计算函数如SIN
*memory access越少越好
*计算量越少越好，变量越少越好，可取的资源越少越好
*Unity3D中配备了强大的阴影和材料的语言工具称为ShaderLab，以程式语言来看，它类似于CgFX和Direct3D的效果框架语法，它描述了材质所必须要的一切咨询，而不仅仅局限于平面顶点/像素着色。
Rendering Paths是Unity3D中一个重要的概念，中文翻译就是“渲染通道”。它可以很大程度上影响光线和阴影的渲染效果，但具体要依赖于具体的游戏内容和硬件设备，以及平台。Unity3D中有三种渲染通道类型，从高到低分别为：Deferred Lighting，Forward Rendering，Vertex Lit。如果平台或设备显卡不能支持高级别的通道类型，Unity3D会自动选择稍微低一些的类型。
三种类型的细节比较，详情看参考手册。
渲染通道与shader的关系:
Deferred Lighting通道类型不关心有多少个光源会影响它，每个物体一般都会绘制两次；类似地，Vertex Lit 只绘制一次。所以对于这两种类型来说，shader对表现效果的改变大多在于多重纹理方面。
Forward 通道类型的表现效果要取决于shader和场景中的光源。它有两种基本的计算方式Vertex-Lit 和 Pixel-Lit。可以翻译为逐顶点渲染法和逐像素渲染法吧应该，对应着D3D中的顶点着色和像素着色过程。
Vertex-Lit 用于对网格模型表面顶点进行光照计算，一次性将所有光源的影响都计算在内，所以无论场景中有多少个光源，这种方式绘制的物体只绘制一次。
Pixel-Lit 会计算每个像素上面最终的光照，因此一个物体必须先呗绘制一次来获得环境光和主方向光的光照信息，再绘制一次来获得其他每个额外的光源信息。应用Pixel-Lit的物体的大小也会影响绘制的效率，越大的物体，绘制越慢。
Vertex-Lit 的开销大于Pixel-Lit，但是Pixel-Lit可以提供很多非常好的效果。