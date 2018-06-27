## RTR
---
## 基于物理着色（Physically Based Shading）
- 就是计算机图形学中用数学建模的方式，模拟物体表面各种材质散射光线的属性
  - 从而渲染照片真实图片的技术
## 图形渲染管线
# 架构
  - 应用，几何，光栅化
  - 最慢的管线阶段决定绘制速度
# 应用阶段
  - 通过软件方式来实现的阶段
  - 用户可完全掌控，运行在CPU
  - 可以在几个并行处理器上同时执行
  - 应用程序阶段虽然是一个单独的过程，但是依然可以对之进行管线化或者并行化处理
  - 有碰撞检测、加速算法、输入检测，动画，力反馈以及纹理动画，变换仿真、几何变形   
    - 以及一些不在其他阶段执行的计算，如层次视锥裁剪等加速算法就可以在这里实现
      - 对应虚幻的 InitView 可见性判断
  - 阶段的末端，将需要在屏幕上（具体形式取决于具体输入设备）显示出来绘制的几何体
    - 也就是绘制图元，rendering  primitives，如点、线、矩形等 输入到绘制管线的下一个阶段
  - 对于被渲染的每一帧，应用程序阶段将摄像机位置，光照和模型的图元输出到管线的下一个主要阶段——几何阶段。
# 几何阶段
  - 一些情况下，一系列连续的功能阶段可以形成单个管线阶段（和其他管线阶段并行运行）
    - 在另外情况下，一个功能阶段可以划分成其他更细小的管线阶段
  - 几何阶段执行的是计算量非常高的任务，在只有一个光源的情况下
    - 每个顶点大约需要100次左右的精确的浮点运算操作
  - 模型变换 视图变换
    - 模型变换的目的是将模型变换到适合渲染的空间当中
    - 视图变换的目的是将摄像机放置于坐标原点，方便后续步骤的操作
    - 在屏幕上的显示过程中，模型通常需要变换到若干不同的空间或坐标系中
      - 模型变换的变换对象一般是模型的顶点和法线
      - 物体的坐标称为模型坐标
      - 世界空间是唯一的，所有的模型经过变换后都位于同一个空间中
    - 不难理解，应该仅对相机（或者视点）可以看到的模型进行绘制
      - 相机在世界空间中有一个位置方向，用来放置和校准相机
    - 为了便于投影和裁剪，必须对相机和所有的模型进行视点变换
      - 变换的目的就是要把相机放在原点，然后进行视点校准
        - 使其朝向Z轴负方向，y轴指向上方,x轴指向右边
        - 在视点变换后，实际位置和方向就依赖于当前的API
        - 称上述空间为相机空间或者观察空间
  - 顶点着色
    - 为了产生逼真的场景，渲染形状和位置是远远不够的，我们需要对物体的外观进行建模
    - 物体经过建模，会得到对每个对象的材质
      - 照射在对象上的任何光源的效果在内的一些描述
      - 且光照和材质可以用任意数量的方式，从简单的颜色描述到复杂的物理描述来模拟
    - 确定材质上的光照效果的这种操作被称为着色（shading）
      - 着色过程涉及在对象上的各个点处计算着色方程（shadingequation）
    - 这些计算中的一些在几何阶段期间在模型的顶点上执行（vertexshading）
    - 其他计算可以在每像素光栅化（per-pixelrasterization）期间执行
    - 也可以在每个顶点处存储各种材料数据，诸如点的位置，法线，颜色或计算着色方程所需的任何其它数字信息
    - 顶点着色的结果（其可以是颜色，向量，纹理坐标或任何其他种类的阴着色数据）计算完成后，会被发送到光栅化阶段以进行插值操作

    - 着色计算通常认为是在世界空间中进行的
    - 有时需要将相关实体（诸如相机和光源）转换到一些其它空间（诸如模型或观察空间）并在那里执行计算
    - 这是因为如果着色过程中所有的实体变换到了相同的空间
      - 着色计算中需要的诸如光源，相机和模型之间的相对关系是不会变的。
    - 顶点着色阶段的目的在于确定模型上顶点处材质的光照效果
  - 投影变换
    - 在光照处理之后，就开始进行投影操作
      - 即将视体变换到一个对角顶点分别是(-1,-1,-1)和(1,1,1)单位立方体（unitcube）内
      - 这个单位立方体通常也被称为规范立方体（Canonical View Volume，CVV）

    - 目前，主要有两种投影方法，即：
    - 正交投影
      - 可视体通常是一个矩形，正交投影可以把这个视体变换为单位立方体
      - 正交投影的主要特性是平行线在变换之后彼此之间仍然保持平行，这种变换是平移与缩放的组合
    - 透视投影
      - 透视投影比正交投影复杂一些。在这种投影中，越远离摄像机的物体，它在投影后看起来越小。
      - 更进一步来说，平行线将在地平线处会聚。透视投影的变换其实就是模拟人类感知物体的方式。

    - 正交投影和透视投影都可以通过4x4的矩阵来实现
      - 在任何一种变换之后，都可以认为模型位于归一化处理之后的设备坐标系中

    - 虽然这些矩阵变换是从一个可视体变换到另一个，但它们仍被称为投影
      - 因为在完成显示后，Z坐标将不会再保存于的得到的投影图片中
    - 通过这样的投影方法，就将模型从三维空间投影到了二维的空间中

  - 裁剪
    - 裁剪阶段的目的，就是对部分位于视体内部的图元进行裁剪操作
    - 只有当图元完全或部分存在于视体（也就是上文的规范立方体，CVV）内部的时候，
      - 才需要将其发送到光栅化阶段，这个阶段可以把这些图元在屏幕上绘制出来。

    - 一个图元相对视体内部的位置，分为三种情况：完全位于内部、完全位于外部、部分位于内部
      - 所以就要分情况进行处理：
        -   当图元完全位于视体内部，那么它可以直接进行下一个阶段。
        -   当图元完全位于视体外部，不会进入下一个阶段，可直接丢弃，因为它们无需进行渲染。
        -   当图元部分位于视体内部，则需要对那些部分位于视体内的图元进行裁剪处理
  - 屏幕映射
    - 只有在视体内部经过裁剪的图元，才可以进入到屏幕映射阶段
      - 进入到这个阶段时，坐标仍然是三维的（但显示状态在经过投影阶段后已经成了二维）
      - 每个图元的x和y坐标变换到了屏幕坐标系中，屏幕坐标系连同z坐标一起称为窗口坐标系

    - 假定在一个窗口里对场景进行绘制，窗口的最小坐标为（x1，y1），最大坐标为（x2，y2），其中x1\<x2，y1\<y2。屏幕映射首先进行平移，随后进行缩放，在映射过程中z坐标不受影响。新的x和y坐标称为屏幕坐标系，与z坐标一起（-1≦
    z ≦ 1）进入光栅化阶段。

    - 经过投影变换，图元全部位于单位立方体之内，而屏幕映射主要目的就是找到屏幕上对应的坐标

    - 屏幕映射阶段的一个常见困惑是整型和浮点型的点值如何与像素坐标（或纹理坐标）进行关联。可以使用Heckbert[书后参考文献第520篇]的策略，用一个转换公式进行解决

    - 屏幕映射阶段的主要目的，就是将之前步骤得到的坐标映射到对应的屏幕坐标系上。
# 光栅化
  - 用经过变换和投影之后的顶点，颜色以及纹理坐标（均来自于几何阶段）给每个像素（Pixel）正确配色，正确绘制整幅图像
    - 这个过个过程叫光珊化（rasterization）或扫描变换（scanconversion）
    - 即从二维顶点所处的屏幕空间（所有顶点都包含Z值即深度值，及各种与相关的着色信息）到屏幕上的像素的转换
  - 三角形Setup
    - 三角形设定阶段主要用来计算，三角形表面的差异，和三角形表面的其他相关数据
    - 该数据主要用于扫描转换scanconversion
      - 以及由几何阶段处理的各种着色数据的插值操作所用
    - 该过程在专门为其设计的硬件上执行
  - 三角形遍历/扫描转换/TriangleTraversal
    - 逐像素检查操作，检查该像素处的像素中心是否由三角形覆盖
      - 而对于有三角形部分重合的像素，将在其重合部分生成片段（fragment）
    - 找到采样点或像素在三角形中的过程通常叫三角形遍历TriangleTraversal或扫描转换
    - 每个三角形片段的属性均由三个三角形顶点的数据插值而生成
      - 这些属性包括片段的深度，以及来自几何阶段的着色数据

  - 像素着色
    - 所有逐像素的着色计算都在像素着色阶段进行，使用插值得来的着色数据作为输入
    - 输出结果为一种或多种将被传送到下一阶段的颜色信息
    - 纹理贴图操作就是在这阶段进行的

    - 像素着色阶段是在可编程GPU内执行的，在这一阶段有大量的技术可以使用
    - 其中最常见，最重要的技术之一就是纹理贴图（Texturing）
      - 纹理贴图就是将指定图片“贴”到指定物体上的过程
      - 而指定的图片可以是一维，二维，或者三维的，其中，自然是二维图片最为常见
    - 像素着色阶段的主要目的是计算所有需逐像素操作的过程
  - 融合
    - 主要任务是合成当前储存于缓冲器中像素着色阶段产生的片段颜色
    - 每个像素的信息都储存在颜色缓冲器中，而颜色缓冲器是一个颜色的矩阵
      - 每种颜色包含红、绿、蓝三个分量
    - 运行该阶段的GPU子单元并非完全可编程的，但其高度可配置，可支持多种特效

    - 这个阶段还负责可见性问题的处理
      - 当绘制完整场景的时候，颜色缓冲器还包含从相机视点处可以观察到的场景图元
      - 大多数图形硬件是通过Z缓冲（也称深度缓冲器）算法来实现的
        - Z缓冲算法非常简单，具有O(n)复杂度（n是需要绘制的像素数量）
          - 只要对每个图元计算出相应的像素z值，就可以使用这种方法，大概内容是：

          - Z缓冲器器和颜色缓冲器形状大小一样，每个像素都存储着一个z值，这个z值是从相机到最近图元之间的距离
          - 每次将一个图元绘制为相应像素时，需要计算像素位置处图元的z值，并与同一像素处的z缓冲器内容进行比较
          - 如果新计算出的z值，远远小于z缓冲器中的z值，那么说明即将绘制的图元与相机的距离比原来距离相机最近的图元还要近
          - 这样，像素的z值和颜色就由当前图元对应的值和颜色进行更新
          - 反之，若计算出的z值远远大于z缓冲器中的z值，那么z缓冲器和颜色缓冲器中的值就无需改变

    - 颜色缓冲器用来存储颜色，z缓冲器用来存储每个像素的z值
    - 还有其他缓冲器可以用来过滤和捕获片段信息
      - 比如alpha通道（alphachannel）和颜色缓冲器联系在一起可以存储一个与每个像素相关的不透明值
        - 可选alpha测试可在，深度测试执行前在传入片段上运行
        - 片段的alpha值与参考值作某些特定的测试（如等于，大于等）
          - 如果片断未能通过测试，它将不再进行进一步的处理
        - alpha测试经常用于，不影响深度缓存的全透明片段的处理
    - 模板缓冲器（stencilbuffer）是用于记录所呈现图元位置的离屏缓存
      - 每个像素通常与占用8个位 unit8
      - 图元可使用各种方法渲染到模板缓冲器中
        - 而缓冲器中的内容可以控制颜色缓存和Z缓存的渲染
        - 举个例子，假设在模版缓冲器中绘制出了一个实心圆形
          - 那么可以使用一系列操作符来将后续的图元仅在圆形所出现的像素处绘制
            - 类似一个mask的操作
        - 模板缓冲器是制作特效的强大工具
          - 而在管线末端的所有这些功能都叫做光栅操作（ROP）
            - 或混合操作（blend operations）
      - 帧缓冲器（framebuffer）通常包含一个系统所具有的所有缓冲器
        - 但有时也可以认为是颜色缓冲器和z缓冲器的组合
      - 累计缓冲器（accumulationbuffer）
        - 是1990年，Haeberli和Akeley提出的一种缓冲器
        - 是对帧缓冲器的补充。这个缓冲器可以用一组操作符对图像进行累积
        - 为了产生运动模糊（motionblur.，可以对一系列物体运动的图像进行累积和平均
          - 此外，其他的一些可产生的效果包括景深depth of field
            - 反走样antialiasing和软阴影

    - 当图元通过光栅化阶段之后，从相机视点处看到的东西就可以在荧幕上显示出来
      - 为了避免观察者体验到对图元进行处理并发送到屏幕的过程图形系统一般使用了双缓冲doublebuffering   
        - 这意味着屏幕绘制是在一个后置缓冲器（backbuffer）中以离屏的方式进行的
        - 一旦屏幕已在后置缓冲器中绘制，后置缓冲器中的内容就不断与已经在屏幕上显示过的前置缓冲器中的内容进行交换，只有当不影响显示的时候，才进行交换 SwapChain
---
## GPU渲染管线与可编程着色器
# GPU管线 概述
  - 第一个包含顶点处理，面向消费者的图形芯片（NVIDIA GeForce256）发布于1999年
    - 且NVIDIA提出了图形处理单元（Graphics Processing Unit，GPU）这一术语
      - 将GeForce256和之前的只能进行光栅化处理的图形芯片相区分。
      - 在接下来的几年中，GPU从可配置的固定功能管线演变到了支持高度可编程的管线。
      - 直到如今，各种可编程着色器依然是控制GPU的主要手段。
      - 为了提高效率，GPU管线的一部分仍然保持着可配置
        - 但不是可编程的，但大趋势依然是朝着可编程和更具灵活性的方向在发展。

  - GPU实现了第二章中描述的几何和光栅化概念管线阶段
    - 其被分为一些不同程度的可配置性和可编程性的硬件阶段

  - GPU实现的渲染管线和第二章中描述的渲染管线的功能阶段在结构上略有不同

  -  顶点着色器（The Vertex Shader）是完全可编程的阶段
    - 顶点着色器可以对每个顶点进行诸如变换和变形在内的很多操作，提供了修改/创建/忽略顶点相关属性的功能，这些顶点属性包括颜色、法线、纹理坐标和位置。顶点着色器的必须完成的任务是将顶点从模型空间转换到齐次裁剪空间。

  - 几何着色器（The Geometry Shader）位于顶点着色器之后
    - 允许GPU高效地创建和销毁几何图元
  - 几何着色器是可选的，完全可编程的阶段
    - 主要对图元（点、线、三角形）的顶点进行操作。
  - 几何着色器接收顶点着色器的输出作为输入，通过高效的几何运算，将数据输出
    - 数据随后经过几何阶段和光栅化阶段的其他处理后，会发送给片段着色器

  -  像素着色器（Pixel Shader，Direct3D中的叫法）常常又称为片断着色器
    - 片元着色器Fragment Shader是完全可编程的阶段
    - 主要作用是进行像素的处理，让复杂的着色方程在每一个像素上执行。
  -  裁剪（Clipping）属于可配置的功能阶段，在此阶段可选运行的裁剪方式
    - 以及添加自定义的裁剪面。
  -  屏幕映射（Screen Mapping）、三角形设置（Triangle Setup）和三角形遍历（Triangle Traversal）阶段是固定功能阶段
  -  合并阶段（The Merger Stage）处于完全可编程和固定功能之间
    - 尽管不能编程，但是高度可配置
    - 其除了进行合并操作，还分管颜色修改（Color Modifying），Z缓冲（Z-buffer），混合（Blend），模板（Stencil）和相关缓存的处理

  - GPU管线已经远离硬编码的运算操作，而朝着提高灵活性和控制性改进


# 可编程着色模型
  - 现代着色阶段（比如支持shader model 4.0，DirectX 10以及之后）使用了通用着色核心（common-shader core），这就表明顶点，片段，几何着色器共享一套编程模型
  - 早期的着色模型可以用汇编语言直接编程，但DX10之后，汇编就只在调试输出阶段可见，改用高级着色语言
  - 目前的着色语言都是C-like的着色语言，比如HLSL,CG和GLSL，其被编译成独立于机器的汇编语言，也称为中间语言（IL）。
    - 这些汇编语言在单独的阶段，通常是在驱动中，被转化成实际的机器语言
    - 这样的安排可以兼容不同的硬件实现。
    - 这些汇编语言可以被看做是定义一个作为着色语言编译器的虚拟机
      - 这个虚拟机是一个处理多种类型寄存器和数据源、预编了一系列指令的处理器
  - 着色语言虚拟机可以理解为一个处理多种类型寄存器和数据源、预编了一系列指令的处理器
    - 考虑到很多图形操作都使用短矢量（最高四位），处理器拥有4路SIMD（single-instruction multiple-data，单指令多数据）兼容性
    - 每个寄存器包含四个独立的值。32位单精度浮点的标量和矢量是其基本数据类型；也随后支持32位整型。
    - 浮点矢量通常包含数据如位置（xyzw），法线，矩阵行，颜色（rgba），或者纹理坐标（uvwq）
    - 而整型通常用来表示，计数器，索引，或者位掩码。也支持综合数据类型比如结构体，数组，和矩阵
    - 而为了便于使用向量，向量操作如调和（swizzling，也就是向量分量的重新排序或复制），和屏蔽（masking，只使用指定的矢量元素），也能够支持


    - 图3.2 DX 10下的通用Shader核心虚拟机架构以及寄存器布局。每个资源旁边显示最大可用编号。其中，用两个斜杠分开的三个数值，分别是顶点，几何、像素着色器对应的可用最大值。

  - 一个绘制调用（Draw Call）会调用图形API来绘制一系列的图元
    - 会驱使图形管线的运行
  - 每个可编程着色阶段拥有两种类型的输入：
    - uniform输入，在一个draw call中保持不变的值（但在不同draw call之间可以更改）
    - varying输入，shader里对每个顶点和像素的处理都不同的值
      - 纹理是特殊类型的uniform输入，曾经一直是一张应用到表面的彩色图片
        - 但现在可以认为是存储着大量数据的数组。

  - 在现代GPU上 ，图形运算中常见的运算操作执行速度非常快。
    - 通常情况下，最快的操作是标量和向量的乘法和加法，以及他们的组合
      - 如乘加运算（multiply-add）和点乘 （dot-product）运算
      - 其他操作，比如倒数（reciprocal）, 平方根（square root），正弦（sine），余弦（cosine），指数（exponentiation）、对数（logarithm）运算，往往会稍微更加昂贵，但依然相当快捷
      - 纹理操作非常高效，但他们的性能可能受到诸如等待检索结果的时间等因素的限制
  - 着色语言表示出了大多数场常见的操作（比如加法和乘法通过运算符+和\*来表示）
    - 其余的操作用固有的函数，比如atan() , dot() , log(),等
    - 更复杂的操作也存在内建函数，比如矢量归一化（vector normalization）、反射（reflection）、叉乘（cross products）、矩阵的转置（matrix transpose）和行列式（determinant）等

  - 流控制（flow control）是指使用分支指令来改变代码执行流程的操作
    - 这些指令用于实现高级语言结构，如“if”和“case”语句，以及各种类型的循环
    - Shader支持两种类型的流控制
      - 静态流控制（Static flow control）是基于统一输入的值的
        - 这意味着代码的流在调用时是恒定的
        - 静态流控制的主要好处是允许在不同的情况下使用相同的着色器（例如，不同数量的光源）
      - 动态流控制（Dynamic flow control）基于不同的输入值。
        - 但动态流控制远比静态流量控制更强大但同时也需更高的开销，特别是在调用shader之间，代码流不规律改变的时候
        - 评估一个shader的性能，是评估其在一段时间内处理顶点或像素的个数
        - 如果流对某些元素选择“if”分支，而对其他元素选择“else”分支，这两个分支必须对所有元素进行评估（并且每个元素的未使用分支将被丢弃）

  - Shader程序可以在程序加载或运行时离线编译
    - 和任何编译器一样，有生成不同输出文件和使用不同优化级别的选项。
    - 一个编译过的Shader作为字符串或者文本来存储，并通过驱动程序传递给GPU
# 顶点着色器 Vertext Shader
  - 它是流水线上的第一个阶段，可选是在GPU还是CPU上实现
    - 而在CPU上实现的话，需将CPU中的输出数据发送到GPU进行光栅化
    - 目前几乎所有的GPU都支持顶点着色
  - 顶点着色器是完全可编程的阶段，是专门处理传入的顶点信息的着色器
  - 顶点着色器可以对每个顶点进行诸如变换和变形在内的很多操作
  - 顶点着色器一般不处理附加信息
    - 顶点着色器提供了修改，创建，或者忽略与每个 多边形顶点 相关的值
      - 例如其颜色，法线，纹理坐标和位置。
  - 顶点着色器程序将顶点从模型空间（ModelSpace）变换到齐次裁剪空间（Homogeneous ClipSpace）
    - 一个顶点着色器至少且必须输出此变换位置

  - 顶点着色阶段之前发生了一些数据操作
    - 比如在DirectX中叫做输入装配（InputAssembler）的阶段
      - 会将一些数据流组织在一起，以形成顶点和基元的集合，发送到管线
      - 在input assembler阶段就可以创建构成物体的triangles（或lines，或points）
        - 本质上是创建带有位置和颜色信息的顶点
        - 可以使用同一个位置数组（并使用一个不同的模型变换矩阵）和一个不同的颜色值数组表示第二个物体
      - 在input assembler阶段还支持执行instancing（同一个物体的多个实例）
        - 这种方法支持通过一次简单的draw call，就可以对同一个物体绘制多次，每次绘制实例包含一些与其他实例不同的数据

  - 顶点着色器本身通用核心虚拟机（Common-Shader Core Virtual  Machine）非常相似
      - 传入的每个顶点由顶点着色器程序处理
        - 然后输出一些在三角形或直线上进行插值后获得的值
      - 顶点着色器既不能创建也不能消除顶点
        - 并且由一个顶点生成的结果不能传递到另一个顶点。
      - 由于每个顶点都被独立处理
        - 所以GPU上的任何数量的着色器处理器都可以并行地应用到传入的顶点流上

  - 顶点着色器的输出可以以许多不同的方式来使用
    - 通常是用于每个实例三角形的生成和光栅化
    - 然后各个像素片段被发送到像素着色器，以便继续处理
    - 而在ShaderModel 4.0中，数据也可以发送到几何着色器（Geometry Shader）
      - 或输出流（StreamedOutput）或同时发动到像素着色器和几何着色器两者中
# 几何着色器 The Geometry Shader
  - 几何着色器（Geometry Shader）是顶点和片段着色器之间一个可选的着色器
  - 几何着色器的输入是单个对象及对象相关的顶点，而对象通常是网格中的三角形，线段或简单的点
    - 另外，扩展的图元可以由几何着色器定义和处理

  - 几何着色器程序的输入是一个单独的类型：点，线段，三角形
    - 两个最右边的图元，包括与线和三角形对象相邻的顶点也可被使用。

  - 几何着色器可以不产生任何的输出
    - 通过这种方法，可以对一个mesh选择性地进行修改，比如通过编辑顶点，增加新的图元，以及删除其他的图元

  - 几何着色器可以改变新传递进来的图元的拓扑结构，且几何着色器可以接收任何拓扑类型的图元，但是只能输出点、折线（line  strip）和三角形条（triangle strips）

  - 几何着色器需要图元作为输入，在处理过程中他可以将这个图元整个丢弃或者输出一个或更多的图元（也就是说它可以产生比它得到的更多或更少的顶点）。
    - 这个能力被叫做几何增长（growing geometry）
    - 如上所述，几何着色器输出的形式只能是点，折线和三角形条。

  - 当我们未添加几何着色器时，默认的行为是将输入的三角形直接输出。
    - 我们添加了几何着色器之后，可以在几何着色器中修改输出的图形，我们可以输出我们想要输出的任何图形



    图
    3.7.一些几何着色器的应用。左图，使用几何着色器实现元球的等值面曲面细分（metaball
    isosurface
    tessellation）。中图，使用了几何着色器和流输出进行线段细分的分形（fractal
    subdivision of line
    segments）。右图，使用顶点和几何着色器的流输出进行布料模拟。（图片来自NVIDIA SDK
    10的示例）
# 流输出 Stream Output
  - GPU的管线的标准使用方式是发送数据到顶点着色器
    - 然后对所得到的三角形进行光栅化处理，并在像素着色器中处理它们
  - 数据总是通过管线传递，无法访问中间结果
    - 流输出的想法在Shader Model4.0中被引入
    - 在顶点着色器（以及可选的几何着色器中）处理顶点之后，除了将数据发送到光栅化阶段之外
      - 也可以输出到流，也就是一个有序数组中进行处理,除此之外还要发送到rasterization阶
    - 可以完全关掉光栅化，然后管线纯粹作为非图形流处理器来使用  
      - 以这种方式处理的数据可以通过管线回传，从而允许迭代处理
      - 这种操作特别适用于模拟流动的水或其他粒子特效
# 像素着色器 Pixel Shader
  - 像素着色器(PixelShader，Direct3D中的叫法)
    - 常常又称为片断着色器,片元着色器(Fragment Shader,OpenGL中的叫法)
    - 用于进行逐像素计算颜色的操作，让复杂的着色方程在每一个像素上执行
    - 像素着色器是光栅化阶段的主要步骤之一
    - 在顶点和几何着色器执行完其操作之后
      - 图元会被裁剪、屏幕映射，结束几何阶段，到达光栅化阶段
      - 在光栅化阶段中先经历三角形设定和三角形遍历，之后来到像素着色阶段

  - 像素着色器常用来处理场景光照和与之相关的效果，如凸凹纹理映射和调色
    - 名称片断着色器似乎更为准确，因为对于着色器的调用和屏幕上像素的显示并非一一对应
    - 举个例子，对于一个像素，片断着色器可能会被调用若干次来决定它最终的颜色
      - 那些被遮挡的物体也会被计算，直到最后的深度缓冲才将各物体前后排序。

  - 像素着色程序通常在最终合并阶段设置片段颜色以进行合并
    - 而深度值也可以由像素着色器修改
    - 模板缓冲（stencil buﬀer ）值是不可修改的
      - 而是将其传递到合并阶段（merge stage）
    - 在SM2.0以及以上版本，像素着色器也可以丢弃（discard）传入的片段数据，即不产生输出
      - 这样的操作会消耗性能，因为通常在这种情况下不能使用由GPU执行的优化
      - 诸如雾计算和alpha测试的操作已经从合并操作转移到SM 4.0中的像素着色器里计算

  - 顶点着色程序的输出，在经历裁剪、屏幕映射、三角形设定、三角形遍历后，实际上变成了像素着色程序的输入
    - 在ShaderModel4.0中，共有16个向量（每个向量含4个值）可以从顶点着色器传到像素着色器
    - 当使用几何着色器时，可以输出32个向量到像素着色器中
  - 像素着色器的追加输入是在ShaderModel3.0中引入的
    - 例如，三角形的哪一面是可见的是通过输入标志来加入的
      - 这个值对于在单个通道中的正面和背面渲染不同材质十分重要
      - 而且像素着色器也可以获得片段的屏幕位置
# 合并阶段 The Merging Stage
  - 作为光栅化阶段名义上的最后一个阶段，合并阶段（The MergingStage）
    - 是将像素着色器中生成的各个片段的深度和颜色与帧缓冲结合在一起的地方。
    - 这个阶段也就是进行模板缓冲（Stencil-Buffer）和Z缓冲（Z-buffer）操作的地方
    - 最常用于透明处理（Transparency）和合成操作（Compositing）的颜色混合（Color
    Blending）操作也是在这个阶段进行的

  - 虽然合并阶段不可编程，但却是高度可配置的
    - 在合并阶段可以设置颜色混合来执行大量不同的操作
    - 最常见的是涉及颜色和Alpha值的乘法，加法，和减法的组合
    - 其他操作也是可能的，比如最大值，最小值以及按位逻辑运算
# 效果 Effect
  - GPU渲染管线中的可编程阶段有顶点、几何和像素着色器三个部分，他们需要相互结合在一起使用
    - 正因如此，不同的团队研发出了不同的特效语言
    - 例如HLSL，FX，CgFX，以及COLLADA FX，来将他们更好的结合在一起
  - 一个效果文件通常会包含所有执行一种特定图形算法的所有相关信息
    - 通常定义一些可被应用程序赋值的全局参数
    - 例如，一个单独的Effectfile可能定义渲染塑料材质需要的vs(顶点着色器)和ps（像素着色器）
    - 它可能暴露一些参数例如塑料颜色和粗糙度，这样渲染每个模型的时候可以改变效果而仅仅使用同一个特效文件
    - 一个效果文件中能存储很多techniques
      - 这些techniques通常是一个相同effect的变体，每种对应于一个不同的Shader
      Model（SM2.0，SM3.0等等）
---
## 图形渲染与视觉外观
- 渲染三维模型的图像时，模型不仅应该具有正确的几何形状，还应该具有期望的视觉外观

- 三种物理现象：
  - 光线从灯光发出并直接传播给房间里其他物体。
  - 物体表面吸收一些物体，并将一些物体散射到新的方向
  - 没有被吸收的光线继续在环境中移动，遇到其他物体
  - 通过场景的光一小部分光进入用于捕获图像的传感器（如摄像机）
- 所有这三种现象的根据
  - 首先，光是由台灯发出的，并直接传播到房间中的物体上
  - 物体的表面会吸收一部分光线并将一些光线散射到新的方向
  - 没有被物体表面吸收的光线会在环境中继续传播，并照射到其他的物体上
  - 最后，在环境中传播的一小部分光线会进入用于捕获图像的传感器中，在该示例中指数码相机的电子传感器

# 光源
  - 光被不同地模拟为几何光线，电磁波或光子（具有一些波特性的量子粒子）
  - 光都是电磁辐射能-通过空间传播的电磁能
  - 光源发光，而不是散射或吸收光
  - 根据渲染目的,光源可以分为三种不同类型：平行光源、点光源和聚光灯
# 材质
  - 在渲染时，通常使用物体的表面来展现场景。
    - 而物体的表面又是通过将材质附加到场景中的模型上来描绘的
  - 每种材质对应于一组 shader 程序，纹理，以及其他属性
  - 这些属性用于模拟光与物体的交互作用。
  - 描述在现实世界中光与物质如何相互作用，然后提供一个简单的材质模型
  - 在渲染中，通过将材质附加到场景中的模型来描绘对象外观。每个材质都和一系列的Shader代码
# 散射与吸收
  - 光与物质相互作用都是两种现象：
    - 散射（scattering）
      - 当光线遇到任何种类的光学不连续性（opticaldiscontinuity）时部分光线向多方面改变方向的现象
      - 可能存在于具有不同光学性质的两种物质分界之处，晶体结构破裂处，密度的变化处等
      - 光的散射（scattering）一般又分为反射（reflection）和透射（transmission）
      - 散射不会改变光量，它只是使其改变方向
    - 吸收（absorption）
      - 发生在物质内部，其会导致一些光转变成另一种能量并消失
      - 吸收会减少光量，但不会影响其方向

  - 镜面反射 /高光
    - 表示在表面反射的光
    - 光从一种介质射到它和另一中介质的分界面时，一部分光返回到这种介质中的现象
  - 漫反射（diffuse） 包括了 次表面散射
    - 一部分进入物体内部。这部分光要不被物体吸收（通常转化为热能），要不在物体内部散射，其中一部分会从物体表面散射出来而被重新看到
    - 漫反射的每条光线均遵循反射定律
  - 通常把表面着色公式表示为两个单独的计算项
    - specular term（镜面光计算项）表示被表面反射的光
    - diffuse term（漫反射光计算项）表示经过了透射，吸收和散射处理的光
  - 入射光（Incomingillumination）通过
    - 表面辐照度（irradiance）来度量
      - 辐照度（irradiance）是电磁辐射入射于曲面时每单位面积的功率
  - 出射光（outgoinglight）通过
    - 辐射出射度（exitance）来度量
      - 定义为离开表面一点处的面元的辐射能通量除以该面元面积
    - 辐射出射度 除以 辐照度 可以作为材质的衡量特性
      - 对于不发光的表面，该比率为0到1之间
      - 辐射出射度 和 辐照度 的比率对于不同的光颜色是不同的
        - 所以其表示为RGB矢量或者颜色，也就是我们通常说的表面颜色c
    - 光物质相互作用是线性的，使辐照度加倍将会使 辐射出射度 增加一倍
# 表面粗糙度 smoothness
  - 镜面反射的方向分布取决于表面粗糙度（smoothness，又译作光滑度）
    - 反射光线对于更平滑的表面更加紧密，并且对于较粗糙的表面更加分散
# 着色与着色方程
  - 着色（Shade）是使用方程式根据材质属性和光源，计算沿着视线v的出射光亮度*Lo*的过程
    - 着色方程具有漫反射和镜面反射分量
# 着色方程的漫反射分量
  - L diff （出射光亮度）=C diff（漫反射color） /3.14* EL（辐照度） cosθi
  - 这种类型的漫反射着色也被叫做兰伯特（Lambertian）着色
    - 兰伯特定律指出，对于理想的漫反射表面，出射光亮度与cosθi成正比。
    - 注意，这种夹紧型cos因子（clamped dot product，可写作max(n·l, 0)，通常称为n点乘l因子）
      - 不是兰伯特表面的特征;
    - 正如我们所见，它一般适用于辐照度（irradiance）的度量。
    - 兰伯特表面的决定性特征是出射光亮度（radiance）和辐照度（irradiance）成正比
# 着色方程
  - 组合漫反射和镜面反射/高光两个项，得到完整的着色方程，总出射光亮度Lo：

# 三种着色处理方法
  - 着色处理是计算光照，并由此，决定像素颜色的过程
  - 存在3种常见的着色处理方法：
    - 平滑着色（Flatshading）：
      - 一个三角面用同一个颜色
      - 一个三角面的代表顶点(也许是按在index中的第一个顶点)
        - 恰好被光照成了白色，那么整个面都会是白的

    - 高洛德着色（Gouraudshading）：
      - 每顶点求值后的线性插值,结果通常称为高洛德着色
      - 在高洛德着色的实现中，顶点着色器传递世界空间的顶点法线和位置到Shade（）
    (首先确保法线矢量长度为1），然后将结果写入内插值
      - 像素着色器将获取内插值并将其直接写入输出
      - 高洛德着色可以为无光泽表面产生合理的结果，但是对于强高光反射的表面

    - 冯氏着色（Phongshading）：
      - 对像素求值
      - 在冯氏着色实现中，顶点着色器将世界空间法线和位置写入内插值
        - 此值通过像素着色器传递给Shade()函数
      - 而将Shade()函数返回值写入到输出中
        - 请注意，即使表面法线在顶点着色器中缩放为长度1，插值也可以改变其长度
        - 因此可能需要在像素着色器中再次执行此归一化操作。

  - 注意Phong Shading和Phong LightingModel的区别
    - 前者是考虑如何在三个顶点中填充颜色，而后者表示的是物体被光照产生的效果。

  - 注意冯氏着色可以说是三者中最接近真实的着色效果，当然开销也是最大的
    - 因为高洛德着色是每个顶点(vertex)计算一次光照
    - 冯氏着色是每个片元(fragment)或者说每像素计算一次光照
    - 点的法向量是通过顶点的法向量插值得到的
    - 所以说不会出现高洛德着色也许会遇到的失真问题
# 抗锯齿与常见抗锯齿类型总结
  - 抗锯齿（英语：anti-aliasing，简称AA），也译为边缘柔化、消除混叠、抗图像折叠有损，反走样
    - 它是一种消除显示器输出的画面中图物边缘出现凹凸锯齿的技术
    - 那些凹凸的锯齿通常因为高分辨率的信号以低分辨率表示或无法准确运算出3D图形坐标定位时所导致的图形混叠（aliasing）而产生的，抗锯齿技术能有效地解决这些问题

# 超级采样抗锯齿（SSAA）
  - 超级采样抗锯齿（Super-Sampling Anti-Aliasing）是比较早期的抗锯齿方法，比较消耗资源但简单直接
    - 这种抗锯齿方法先把图像映射到缓存并把它放大，再用超级采样把放大后的图像像素进行采样
    - 一般选取2个或4个邻近像素，把这些采样混合起来后，生成的最终像素，令每个像素拥有邻近像素的特征
    - 像素与像素之间的过渡色彩，就变得近似，令图形的边缘色彩过渡趋于平滑
    - 再把最终像素还原回原来大小的图像，并保存到帧缓存也就是显存中，替代原图像存储起来，最后输出到显示器，显示出一帧画面
    - 这样就等于把一幅模糊的大图，通过细腻化后再缩小成清晰的小图
    - 如果每帧都进行抗锯齿处理，游戏或视频中的所有画面都带有抗锯齿效果。
  - 超级采样抗锯齿中使用的采样法一般有两种：
    - OGSS，顺序栅格超级采样（Ordered GridSuper-Sampling，简称OGSS），采样时选取2个邻近像素。
    - RGSS，旋转栅格超级采样（Rotated GridSuper-Sampling，简称RGSS），采样时选取4个邻近像素。

  - 作为概念上最简单的一种超采样方法
    - 全场景抗锯齿（Full-SceneAntialiasing,FSAA）以较高的分辨率对场景进行绘制
      - 然后对相邻的采样样本进行平均，从而生成一幅新的图像

# 多重采样抗锯齿（MSAA）
  - 多重采样抗锯齿（Multi SamplingAnti-Aliasing，简称MSAA）
    - 是一种特殊的超级采样抗锯齿（SSAA）
    - MSAA首先来自于OpenGL
      - 具体是MSAA只对Z缓存（Z-Buffer）和模板缓存(StencilBuffer)中的数据进行超级采样抗锯齿的处理
    - 可以简单理解为只对多边形的边缘进行抗锯齿处理
      - 这样的话，相比SSAA对画面中所有数据进行处理，MSAA对资源的消耗需求大大减弱，不过在画质上可能稍有不如SSAA

# 覆盖采样抗锯齿（CSAA）
  - 覆盖采样抗锯齿（Coverage SamplingAnti-Aliasing，简称CSAA）
    - 是NVIDIA在G80及其衍生产品首次推向实用化的AA技术
    - 也是目前NVIDIAGeForce8/9/G200系列独享的AA技术
  - CSAA就是在MSAA基础上更进一步的节省显存使用量及带宽
    - 简单说CSAA就是将边缘多边形里需要取样的子像素坐标覆盖掉
    - 把原像素坐标强制安置在硬件和驱动程序预先算好的坐标中
    - 这就好比取样标准统一的MSAA，能够最高效率的执行边缘取样，效能提升非常的显著。
    - 比方说16xCSAA取样性能下降幅度仅比4xMSAA略高一点，处理效果却几乎和8xMSAA一样。
    - 8xCSAA有着4xMSAA的处理效果，性能消耗却和2xMSAA相同。

# 高分辨率抗锯齿（HRAA）

  - 高分辨率抗锯齿方法(High ResolutionAnti-Aliasing，简称HRAA)，也称Quincunx方法，也出自NVIDIA公司。
  - “Quincunx”意思是5个物体的排列方式，其中4个在正方形角上，第五个在正方形中心，也就是梅花形，很像六边模型上的五点图案模式。
  - 此方法中，采样模式是五点梅花状，其中四个样本在像素单元的角上，最后一个在中心。

# 可编程过滤抗锯齿（CFAA）

  - 可编程过滤抗锯齿（Custom FilterAnti-Aliasing，简称CFAA）技术起源于AMD-ATI的R600家庭
    - 简单地说CFAA就是扩大取样面积的MSAA，比方说之前的MSAA是严格选取物体边缘像素进行缩放的
    - 而CFAA则可以通过驱动和谐灵活地选择对影响锯齿效果较大的像素进行缩放
    - 以较少的性能牺牲换取平滑效果。显卡资源占用也比较小。

# 形态抗锯齿（MLAA）
  - 形态抗锯齿（MorphologicalAnti-Aliasing，简称MLAA）
    - 是AMD推出的完全基于CPU处理的抗锯齿解决方案
    - 与MSAA不同，MLAA将跨越边缘像素的前景和背景色进行混合
      - 用第2种颜色来填充该像素,从而更有效地改进图像边缘的变现效果。

# 快速近似抗锯齿（FXAA）
  - 快速近似抗锯齿(Fast Approximate Anti-Aliasing，简称FXAA)
    - 是传统MSAA(多重采样抗锯齿)效果的一种高性能近似
    - 它是一种单程像素着色器，和MLAA一样运行于目标游戏渲染管线的后期处理阶段，但不像后者那样使用DirectCompute，而只是单纯的后期处理着色器，不依赖于任何GPU计算API
    - 正因为如此，FXAA技术对显卡没有特殊要求，完全兼容NVIDIA、AMD的不同显卡(MLAA仅支持A卡)和DirectX
9.0、DirectX 10、DirectX 11。

# 时间性抗锯齿（TXAA）
  - 时间性抗锯齿（Temporal Anti-Aliasing，简称TXAA）
    - 将MSAA、时间滤波以及后期处理相结合，用于呈现更高的视觉保真度
    - 与CG电影中所采用的技术类似，TXAA集MSAA的强大功能与复杂的解析滤镜于一身，可呈现出更加平滑的图像效果
    - TXAA还能够对帧之间的整个场景进行抖动采样，以减少闪烁情形，闪烁情形在技术上又称作时间性锯齿
    - 目前，TXAA有两种模式：TXAA2X和TXAA 4X
      - TXAA 2X可提供堪比8X MSAA的视觉保真度，然而所需性能却与2XMSAA相类似；
      - TXAA 4X的图像保真度胜过8XMSAA，所需性能仅仅与4X MSAA相当。

# 多帧采样抗锯齿（MFAA）
  - 多帧采样抗锯齿（Multi-Frame Sampled Anti-Aliasing，MFAA）是NVIDIA公司根据MSAA改进出的一种抗锯齿技术
  - 目前仅搭载 Maxwell架构GPU的显卡才能使用
  - 可以将MFAA理解为MSAA的优化版，能够在得到几乎相同效果的同时提升性能上的表现
  - MFAA与MSAA最大的差别就在于在同样开启4倍效果的时候MSAA是真正的针对每个边缘像素周围的4个像素进行采样
    - MFAA则是仅仅只是采用交错的方式采样边缘某个像素周围的两个像素


# 透明渲染
  - 透明渲染是是图形学里面的常见问题之一，可以从《Real-Time Rendering3rd》中总结出如下两个算法：
    - Screen-DoorTransparency方法
      - 基本思想是用棋盘格填充模式来绘制透明多边形
        - 以每隔一个像素绘制一点方式的来绘制一个多边形，这样会使在其后面的物体部分可见
        - 通常情况下，屏幕上的像素比较紧凑，以至于棋盘格的这种绘制方式并不会露馅
        - 同样的想法也用于剪切纹理的抗锯齿边缘
          - 但是在子像素级别中的，这是一种称为alpha覆盖（alphato coverage）的特征
        - screen-doortransparency方法的优点就是简单
          - 可以在任何时间任何顺序绘制透明物体，并不需要特殊的硬件支持（只要支持填充模式）
          - 缺点是透明度效果仅在50%时最好，且屏幕的每个区域中只能绘制一个透明物体。
    - Alpha混合（AlphaBlending）方法
      - 这个方法比较常见，其实就是按照Alpha混合向量的值来混合源像素和目标像素
      - 当在屏幕上绘制某个物体时，与每个像素相关联的值有RGB颜色和Z缓冲深度值
        - 以及另外一个成分alpha分量，这个alpha值也可以根据需要生成并存储
        - 它描述的是给定像素的对象片段的不透明度的值。
      - alpha为1.0表示对象不透明，完全覆盖像素所在区域;
        - 0.0表示像素完全透明。
      - 为了使对象透明，在现有场景的上方，以小于1的透明度进行绘制即可。
        - 每个像素将从渲染管线接收到一个RGBA结果，并将这个值和原始像素颜色相混合。


# 透明排序
  - 要将透明对象正确地渲染到场景中，通常需要对物体进行排序
    - 下面分别介绍两种比较基本的透明排序方法（深度缓存和油画家算法）和两种高级别的透明排序算法（加权平均值算法和深度剥离）

# 深度缓存（Z-Buffer）
  - Z-Buffer也称深度缓冲
    - 在计算机图形学中，深度缓冲是在三维图形中处理图像深度坐标的过程
      - 这个过程通常在硬件中完成，它也可以在软件中完成，它是可见性问题的一个解决方法
    - 可见性问题是确定渲染场景中哪部分可见、哪部分不可见的问题

  - Z-buffer的限制是每像素只存储一个对象
    - 如果一些透明对象与同一个像素重叠，那么单独的Z-buffer就不能存储并且稍后再解析出所有可见对象的效果
    - 这个问题是通过改变加速器架构来解决的，比如用A-buffer。
      - A-buffer具有“深度像素（deeppixels）”
      - 其可以在单个像素中存储一系列呈现在所有对象之后被解析为单个像素颜色的多个片段
      - 但需注意，Z-buffer是市场的主流选择

# 画家算法（Painter's Algorithm）
  - 画家算法也称优先填充算法，效率虽然较低，但还是可以有效处理透明排序的问题
    - 其基本思想是按照画家在绘制一幅画作时，首先绘制距离较远的场景
      - 然后用绘制距离较近的场景覆盖较远的部分的思想
    - 画家算法首先将场景中的多边形根据深度进行排序，然后按照顺序进行描绘
      - 这种方法通常会将不可见的部分覆盖，这样就可以解决可见性问题。

# 加权平均值算法（Weighted Average）
  - 使用简单的透明混合公式来实现无序透明渲染的算法
    - 它通过扩展透明混合公式，来实现无序透明物件的渲染，从而得到一定程度上逼真的结果。

# 深度剥离算法（Depth Peeling）
  - 深度剥离是一种对深度值进行排序的技术。
    - 标准的深度检测使场景中的Z值最小的点输出到屏幕上，就是离我们最近的顶点。
    - 但还有离我们第二近的顶点，第三近的顶点存在。
    - 要想显示它们，可以用多遍渲染的方法。
      - 第一遍渲染时，按照正常方式处理，这样就得到了离我们最近的表面中的每个顶点的z值。
      - 在第二遍渲染时，把现在每个顶点的深度值和刚才的那个深度值进行比较，
        - 凡是小于等于第一遍得到的z值，把它们剥离，后面的过程依次类推即可。

    - 每个深度剥离通道渲染特定的一层透明通道。左侧是第一个Pass，直接显示眼睛可见的层，中间的图显示了第二层，显示了每个像素处第二靠近透明表面的像素。右边的图是第三层，每个像素处第三靠近透明表面的像素。

# 伽玛校正
  - 伽马校正（Gamma correction） 又叫伽马非线性化（gammanonlinearity），伽马编码（gamma encoding）
  - 是用来对光线的辐照度（luminance）或是三色刺激值（tristimulusvalues）
    - 所进行非线性的运算或反运算的一种操作
  - 为图像进行伽马编码的目的是用来对人类视觉的特性进行补偿，从而根据人类对光线或者黑白的感知
    - 最大化地利用表示黑白的数据位或带宽

---
## 纹理贴图及相关技术
# 纹理管线 The Texturing Pipeline
  - 纹理（Texturing）是一种针对，物体表面属性进行，“建模”的高效技术
    - 图像纹理中的像素通常被称为纹素（Texels），区别于屏幕上的像素 根据Kershaw的术语
    - 通过将投影方程（projectorfunction）运用于空间中的点
      - 从而得到一组称为参数空间值（parameter-spacevalues）的关于纹理的数值
      - 这个过程就称为贴图（Mapping，也称映射）
        - 也就是纹理贴图（Texture Mapping，也称纹理映射）这个词的由来
    - 纹理贴图可以用一个通用的纹理管线来进行描述
    - 纹理贴图过程的初始点是空间中的一个位置
      - 这个位置可以基于世界空间，但是更常见的是基于模型空间
        - 因为若此位置是基于模型空间的，当模型移动时，其纹理才会随之移动。

  图2 单个纹理的通用纹理管线

  - 纹理管线的分步概述：
    - 第一步。空间中的点通过将投影方程（projector function）计算
      - 从而得到一组称为参数空间值（parameter-space values）的关于纹理的数值
    - 第二步。在使用这些新值访问纹理之前，可以使用一个或者多个映射函corresponderfunction
      - 将参数空间值（parameter-space values ）转换到纹理空间
    - 第三步。使用这些纹理空间值（texture-spacelocations）从纹理中获取相应的值（obtain
      value）
        - 例如，可以使用图像纹理的数组索引来检索像素值。
    - 第四步。再使用值变换函数（value transform function）对检索结果进行值变换
      - 最后使用得到的新值来改变表面属性，如材质或者着色法线等等。

      - 而如下这个例子应该对理解纹理管线有所帮助。下例将描述出使用纹理管线，一个多边形在给定一张砖块纹理时在其表面上生成样本时（如图3）发生了哪些过程。


  - 图3 一个砖墙的纹理管线过程

    - 如图3所示，在具体的参考帧画面中找到物体空间中的位置(x,y,z)
    - 如图中点（-2.3,7.1,88.2），然后对该位置运用投影函数。
      - 这个投影函数通常将向量(x,y,z)转换为一个二元向量(u,v)。
      - 在此示例中使用的投影函数是一个正交投影，类似一个投影仪，将具有光泽的砖墙图像投影到多边形表面上。
      - 再考虑砖墙这边，其实这个投影过程就是将砖墙平面上的点变换为值域为0到1之间的一对(u,v)值
      - 如图，(0.32,0.29)就是这个我们通过投影函数得到的uv值。
        - 而我们图像的分辨率是256x 256
        - 所以，将256分别乘以(0.32,0.29)，去掉小数点，得到纹理坐标(81,74)
    - 通过这个纹理坐标，可以在纹理贴图上查找到坐标对应的颜色值
      - 所以，我们接着找到砖块图像上像素位置为(81,74)处的点 得到颜色(0.9,0.8,0.7)
      - 而由于原始砖墙的颜色太暗，因此可以使用一个值变换函数
        - 给每个向量乘以1.1，就可以得到我们纹理管线过程的结果——颜色值(0.99,0.88,0.77)
      - 随后，我们就可以将此值用于着色方程，作为物体的漫反射颜色值，替换掉之前的漫反射颜色

# 投影函数 The Projector Function
  - 作为纹理管线的第一步，投影函数的功能就是将空间中的三维点转化为纹理坐标
    - 也就是获取表面的位置并将其投影到参数空间中。

  - 投影函数通常在美术建模阶段使用，并将投影结果存储于顶点数据中
    - 也就是说，在软件开发过程中，我们一般不会去用投影函数去计算得到投影结果
      - 而是直接使用在美术建模过程中，已经存储在模型顶点数据中的投影结果
  - 但有一些特殊情况，例如：
    - 1、OpenGL的glTexGen函数提供了一些不同的投影函数，包括球形函数和平面函数。
      - 利用空闲时间可以让图形加速器来执行投影过程
        - 而这样做的优点是不需要将纹理坐标送往图形加速器，从而可以节省带宽。
    - 2、可以在顶点或者像素着色器中使用投影函数，这可以实现各种效果
      - 包括一些动画和一些渲染方法（比如如环境贴图，environmentmapping，
        - 有自身特定的投影函数，可以针对每个顶点或者每个像素进行计算）。

  - 通常在建模中使用的投影函数有球形、圆柱、以及平面投影，也可以选其他一些输入作为投影函数

  - 图4 不同的纹理坐标，上面一行从左到右分别为球形、圆柱、平面，以及自然uv投影：
    - 下面一行所示为把不同的投影运用于同一个物体的情形。

  - 非交互式渲染器（Noninteractiverenderers）
    - 通常将这些投影方程称为渲染过程本身的一部分
    - 一个单独的投影方程就有可能适用于整个模型
      - 实际上美术同学不得不使用各种各样的工具将模型进行分割
        - 针对不同的部分，分别使用不同的投影函数。如图5所示。
    - 图5 使用不同的投影函数将纹理以不同的方式投射到同一个模型上

  - 各种常见投影的不同要点：
      - 球形投影（The sphericalprojection）
        - 球形投影将点投射到一个中心位于某个点的虚拟球体上
          - 这个投影与Blinn与Newell的环境贴图方法相同

      - 圆柱投影（Cylindricalprojection）。
        - 与球体投影一样，圆柱投影计算的是纹理坐标u，而计算得到的另一个纹理坐标v是沿该圆柱轴线的距离。这种投影方法对具有自然轴的物体比较适用，比如旋转表面，如果表面与圆柱体轴线接近垂直时，就会出现变形。

      - 平面投影（The planarprojection）。
        - 平面投影非常类似于x-射线幻灯片投影，它沿着一个方向进行投影，并将纹理应用到物体的所有表面上。这种方法通常使用正交投影，用来将纹理图应用到人物上，其把模型看作一个用纸做的娃娃，将不同的纹理粘贴到该模型的前后。

    - 图6 雕塑模型上的多个较小纹理，保存在两个较大的纹理上
      - 右图显示了多边形网格如何展开并显示在纹理上的
# 映射函数 The Corresponder Function
  - 映射函数（The Corresponder Function）的作用是
    - 将参数空间坐标（parameter-space coordinates）转换为纹理空间位置（texture space locations）
  - 我们知道图像会出现在物体表面的(u,v)位置上，且uv值的正常范围在[0,1)范围内
    - 超出这个值域的纹理，其显示方式便可以由映射函数（TheCorresponder Function）来决定
  - 在OpenGL中，这类映射函数称为“封装模式（WarappingMode）”
  - 在Direct3D中，这类函数叫做“寻址模式（Texture AddressingMode）”
  - 最常见的映射函数有以下几种：
    - 重复寻址模式，Wrap (DirectX), Repeat (OpenGL)
      - 图像在表面上重复出现。
    - 镜像寻址模式，Mirror
      - 图像在物体表面上不断重复，但每次重复时对图像进行镜像或者反转。
    - 夹取寻址模式，Clamp (DirectX) ,Clamp to edge(OpenGL)
      - 夹取纹理寻址模式将纹理坐标夹取在[0.0，1.0]之间
        - 也就是说，在[0.0，1.0]之间就是把纹理复制一遍
        - 然后对于[0.0，1.0]之外的内容，将边缘的内容沿着u轴和v轴进行延伸
    - 边框颜色寻址模式，Corder (DirectX) ,clamp to border(OpenGL)
      - 边框颜色寻址模式就是在[0.0，1.0]之间绘制纹理
      - 然后[0.0，1.0]之外的内容就用边框颜色填充

  - 图7 图像寻址模式，从左到右分别是重复寻址、镜像寻址、夹取寻址、边框颜色寻址
  - 另外，每个纹理轴可以使用不同的映射函数
    - 例如在u轴使用重复寻址模式，在v轴使用夹取寻址模式

# 体纹理 Volume Texture
  - 什么是三维纹理（3D texture），即体纹理（volume texture）？？
    - 是传统二维纹理（2Dtexture）在逻辑上的扩展
    - 二维纹理是一张简单的位图图片，用于为三维模型提供表面点的颜色值；
    - 而一个三维纹理，可以被认为由很多张2D 纹理组成，用于描述三维空间数据的图片
    - 三维纹理通过三维纹理坐标进行访问
- 三维纹理的优势是什么？？
  - 虽然体纹理具有更高的储存要求，并且滤波成本更高，但它们具有一些独特的优势：
    - 使用体纹理，可以跳过，为三维网格确定二维参数的复杂过程
      - 因为三维位置可以直接用作纹理坐标，从而避免了二维参数化中通常会发生的变形和接缝问题
- 三维纹理的用法？？
  - 体纹理也可用于表示诸如木材或大理石的材料的体积结构
    - 使用三维纹理实现出的这些模型，看起来会很逼真，浑然天成

- 三维纹理的劣势？？
  - 使用体纹理作为表面纹理会非常低效，因为三维纹理中的绝大多数样本都没起到作用

# 立方体贴图 Cube Map
  - 立方体纹理（cube texture）或立方体贴图（cubemap）是一种特殊的纹理技术
    - 它用6幅二维纹理图像构成一个以原点为中心的纹理立方体
      - 这每个2D纹理是一个立方体（cube）的一个面
      - 对于每个片段，纹理坐标(s,t, r)被当作方向向量看待
        - 每个纹素(texel)都表示从原点所看到的纹理立方体上的图像。


  - 图8 Cube Map图示1
  - 图9 Cube Map图示2

  - 可以使用三分量纹理坐标向量来访问立方体贴图中的数据
    - 获取方向向量碰触到立方体表面上的相应的纹理像素，这样就返回了正确的纹理采样值
    - 该矢量指定了从立方体中心向外指向的光线的方向
  - 选择具有最大绝对值的纹理坐标对应的相应的面。
    - （例如：对于给定的矢量(−3.2,5.1, −8.4)，就选择-Z面）
      - 而对剩下的两个坐标除以最大绝对值坐标的绝对值，即8.4。
      - 那么就将剩下的两个坐标的范围转换到了-1到1
      - 然后重映射到[0，1]范围，以方便纹理坐标的计算
      - 例如，坐标（-3.2,5.1）映射到（（-3.2/ 8.4 + 1）/ 2，（5.1/ 8.4 + 1）/ 2）≈（0.31,0.80）

  - 立方体贴图支持双线性滤波以及mipmapping，但问题可以可能会在贴图接缝处出现
    - 有一些处理立方体贴图专业的工具在滤波时考虑到了可能的各种因素
      - 如ATI公司的CubeMapGen，采用来自其他面的相邻样本创建mipmap链
        - 并考虑每个纹素的角度范围，可以得到比较不错的效果。如图10。

  - 图10 立方体贴图过滤
  - 最左边两个图像使用2 x 2和4 x 4的立方体贴图的纹理层次
    - 采用标准立方体贴图生成mipmap链
    - 因接缝显而易见，除了极端细化的情况，这些mipmap级别并不可用
    - 两个最右边的图像使用相同分辨率的mipmap级别，通过在立方体面和采用角度范围进行采样生成
    - 由于没有接缝，不易失真，这些mipmap甚至可以用于显示在很大的屏幕区域的对象

# 纹理缓存 Texture Caching
  - 一个复杂的应用程序可能需要相当数量的纹理
    - 快速纹理存储器的数量因系统而异，但你会发现它们永远不够用
    - 有各种各样的纹理缓存（texturecaching）技术
      - 但我们在，上传纹理到内存的开销，和纹理单次消耗的内存量，之间寻求一个好的平衡点
      - 比如，一个由纹理贴图的多边形对象，初始化在离相机很远的位置
        - 程序也许会只加载mipmap中更小的子纹理，就可以很完美的完成这个对象的显示了

  - 一些基本的建议是——保持纹理在不需要放大再用的前提下尽可能小
    - 并尝试基于多边形将纹理分组
    - 即便所有纹理都一直存储在内存中，这种预防措施也可能会提高处理器的缓存性能

  - 以下是一些常见的纹理缓存使用策略
    - 1 最近最少使用策略（Least Recently Used ,LRU）
      - 最近最少使用（Least Recently Used,LRU）策略是纹理缓存方案中常用的一种策略
        - 加载到图形加速器的内存中的每个纹理都被给出一个时间戳，用于最后一次访问以渲染图像时
          - 当需要空间来加载新的纹理时，首先卸载最旧时间戳的纹理
            - 一些API还允许为每个纹理设置一个优先级
        - 如果两个纹理的时间戳相同，则优先级较低的纹理首先被卸载
        - 设置优先级可以帮助避免不必要的纹理交换。

    - 2 最近最常使用策略（Most Recently Used，MRU）
      - Carmack（就是那个游戏界大名鼎鼎的卡马克）提出了一种非常有用的策略
        - 也就是对交换出缓冲器的纹理进行核查
      - [具体可见原书参考文献374]。思想是这样：
        - 鉴于如果在当前帧中载入纹理，会发生抖动（Thrashing）的情况。
        - 这种情况下，LRU策略是一种非常不好的策略
          - 因为在每帧画面中会对每张纹理图像进行交换。
        - 可以采用最近最常使用（MostRecently Used，MRU）策略
          - 直到在画面中没有纹理交换时为止，再然后切换回LRU

    - 3 预取策略（Prefetching）
      - 加载纹理花费显着的时间，特别是在需将纹理转换为硬件原生格式时
        - 纹理加载在每个框架可以有很大的不同
        - 在单个帧中加载大量纹理使得难以保持恒定的帧速率
        - 一种解决方案是使用预取（prefetching）
          - 在将来需要预期的情况下，预计未来的需求然后加载纹理，将加载过程分摊在多帧中

    - 4 裁剪图策略（Clipmap）
      - 对于飞行模拟和地型模拟系统，图像数据集可能会非常巨大。
        - 传统的方法是将这些图像分解成更小的硬件可以处理的瓦片地图（tiles）
        - Tanner等人提出了一种一种称为裁剪图（clipmap）的改进数据结构
          - 其思想是，将整个数据集视为一个mipmap，
          - 但是对于任何特定视图，只需要mipmap的较低级别的一小部分即可。
          - 使用clipmapp技术可以减少在同一时间所需数据量
          - 支持DirectX10的GPU就能够实现clipmap技术。
          - 用这种技术制作的图像如图11所示
          - 高分辨率地形图访问海量图像数据库

# 纹理压缩 Texture Compression
  - 直接解决内存和带宽问题和缓存问题的一个解决方案是固定速率纹理压缩
    - （Fixed-rateTextureCompression）
    - 通过硬件解码压缩纹理，纹理可以需要更少的纹理内存，从而增加有效的高速缓存大小
      - 至少这样的纹理使用起来更高效，因为他们在访问时消耗更少的内存带宽

  - 有多种图像压缩方法用于图像文件格式，如JPEG和PNG，但在硬件上对其实现解码会非常昂贵
    - S3公司开发一种名为S3纹理压缩（S3TextureCompression，S3TC）的方法
      - 目前已经被选为DirectX中的标准压缩模式，称为DXTC
        - 在DirectX10中，这种方法称为BC (BlockCompression)
        - 其优点是创建了一个固定大小，具有独立的编码片段，并且解码简单，同时速度也很快
        - 每个压缩部分的图像可以独立处理，没有共享查找表（look-uptables）或其他依赖关系
          - 这同样地简化了解码过程

  - 还有几种S3/DXTC/BC压缩方案的变种存在，他们有一些共同的特征
    - 把纹理按4x4个单元（纹素）大小划分为块
    - 每个块对应一张四色查找表，表中存有两个标准RGB565格式表示的16位颜色
      - 另外使用标准插入算法在插入两个新的颜色值，由此构成四色查找表
      - 4x4大小的纹理块中每个单元（像素点）用两个bit表示
        - 每一个都代表四色查找表中的一种颜色
      - 可以看出，实质上是利用每个单元（像素点）中的两个bit来索引四色查找表中的颜色值
    - 这些压缩技术可以应用于立方体或体积图，以及二维纹理
      - 而其主要缺点是它们是有损的压缩。
      - 也就是说，原始图像通常不能从压缩版本检索。
    - 仅使用四个或八个内插值来表示16个像素。
      - 如果一个瓦片贴图有更大的数值，相较压缩前就会有一些损失。
      - 在实践中，如果正常使用这些压缩方案，一般需给出可接受的图像保真度。

  - DXTC的一个问题是用于块的所有颜色都位于RGB空间的直线上
    - 例如，DXTC不能在一个块中同时表示红色，绿色和蓝色

  - 下面对几种不同纹理压缩变体(S3/DXTC/BC以及ETC)在编码上的异同点分节概述。

  - 1 DXT1
  --------

  DXT1（DirectX 9.0）或BC1（DirectX 10.0及更高版本） -
  每个块具有两个16位参考RGB值（5位红，6绿，5蓝）的纹素，而每个纹素具有2位插值因子，以便从一个参考值或两个中间值之间选择。DXT1作为五种变体中最精简的版本，块占用为8个字节，即每个纹素占用4位。
  与未压缩的24位RGB纹理相比，有着6：1的纹理压缩率。

  - 2 DXT3
  --------

  DXT3（DirectX 9.0）或BC2（DirectX 10.0及更高版本） -
  每个块都具有与DXT1块相同的RGB数据编码。另外，每个纹素都具有单独存储4位alpha值（这是唯一的直接存储数据的形式，而不是用插值的形式）。DXT3块占用16个字节，或每个纹理元素8位。与未压缩的32位RGBA纹理相比，有着4：1的纹理压缩率。

  - 3 DXT5
  --------

  DXT5（DirectX 9.0）或BC3（DirectX 10.0及更高版本） -
  每个块都具有与DXT1块相同的RGB数据编码。此外，alpha数据使用两个8位参考值和一个每纹素3位的插值因子进行编码。每个纹素可以选择参考alpha值之一或六个中间值之一作为其值。DXT5块具有与DXT3块相同的存储要求，也就是DXT3块占用16个字节，即每个纹理元素8位。与未压缩的32位RGBA纹理相比，有着4：1的纹理压缩率。

  - 4 ATI1
  --------

  ATI1（ATI公司的特定扩展名）或BC4（DirectX 10.0及更高版本）-
  每个块存储单个颜色的数据通道，以与DXT5中的alpha数据相同的方式进行编码。
  BC4块占用8个字节，即每个纹素占用4位。与未压缩的8位单通道纹理相比，有着4：1的纹理压缩率。仅在较新的ATI的硬件或任意供应商的DirectX
  10.0硬件上才支持此格式。

  - 5 ATI2
  --------

  ATI2（ATI公司的特定扩展名，也称为3Dc）或BC5（DirectX10.0及更高版本） -
  每个块存储两个颜色通道的数据，以与BC4块或DXT5中的alpha数据相同的方式进行编码。
  BC5块占用16个字节，即每个纹理元素8位。与未压缩的16位双通道纹理相比，有着4：1的纹理压缩率。也仅在较新的ATI的硬件或任意供应商的DirectX
  10.0硬件上才支持此格式。

  - 6 ETC
  -------

  对于OpenGL ES，选择了另一种称为ETC（Ericsson texture
  compression，ETC）的压缩算法。方案与S3TC具有相同的特点，即快速解码，随机访问，无间接查找，速率固定。ETC算法将4×4纹素的块编码为64位，即每个纹理元素4位。基本思想如图12所示。


  图12 ETC算法对像素块的颜色进行编码，然后修改每像素的亮度以创建最终的纹理颜色

  每个2×4块（或4×2，取决于哪个质量更佳）存储基本颜色。每个块还从一个小的静态查找表中选择一组四个常量，并且块中的每个纹素可以选择在选定的查找表中添加一个值，添加的这个值就可以是每像素的亮度。

  也可以这样理解：ETC压缩算法将图像中的chromatic和luminance分开存储的方式，而在解码时使用luminance对chromatic进行调制进而重现原始图像信息。

  - 两个要点：
    -   ETC的图片压缩的质量和DXTC相当。
    -   ETC也主要有两种方法：ETC1和改进后的ETC2

# 程序贴图纹理 Procedural Texturing
  - 程序贴图纹理（ProceduralTexturing，也可译为过程纹理）是用计算机算法生成的
    - 旨在创建用于纹理映射的自然元素（例如木材，大理石，花岗岩，金属，石头等）的真实表面或三维物体而创建的纹理图像
    - 通常，会使用分形噪声（fractalnoise）和湍流扰动函数（turbulencefunctions）
      - 这类“随机性”的函数来生成程序贴图纹理
      - 给定纹理空间位置，进行图像查找是生成纹理值的一种方法
      - 另一种方法是对函数进行求值，从而得到一个程序贴图纹理（proceduraltexture）。

  - 过程纹理主要用于模拟自然界中常见的Marble,Stone,Wood,Cloud等纹理
  - 大多数的过程纹理都是基于某类噪声函数（NoiseFunction）
    - 比如说perlinnoise。
      - 在过去，由于过程纹理计算量很大，在实时绘制中很少使用
      - 但是GPU的出现，促进了过程纹理在实时渲染中的广泛应用


  - 图13 程序贴图纹理示例
    - 程序贴图纹理通常用于离线渲染应用程序，而图像纹理在实时渲染中更为常见
      - 这是由于在现代GPU中的图像纹理硬件有着极高效率，其可以在一秒钟内执行数十亿个纹理访问
      - 然而，GPU架构正在朝着更便宜的计算能力和（相对）更昂贵的存储器访问而发展
      - 这将使程序纹理在实时应用程序中更常见，尽管它们不可能完全替代图像纹理
    - 考虑到体积图像纹理的高存储成本，体积纹理用于程序贴图纹理，是一项特别有吸引力的应用
      - 这样的纹理可以通过各种技术来合成，最常见的是使用一个或多个噪声函数来产生纹理的值

# 纹理动画 Texture Animation
  - 贴图偏移量滚动一个材质
  - 使用贴图偏移量为材质设置动画
# 凹凸贴图与其改进
  - 凹凸贴图（Bump Mapping）思想最早是由图形学届大牛中的大牛JimBlinn提出
    - 后来的Normal Mapping，Parallax Mapping，Parallax OcculisionMapping，Relief
  Mapping等
    - 均是基于同样的思想，只是考虑得越来越全面，效果也越来越逼真

  - 以下是几种凹凸贴图与其改进方法的总结对比

  - 除了DisplacementMapping方法以外，其他的几种改进一般都是通过修改每像素着色方程来实现
    - 关键思想是访问纹理来修改表面的法线，而不是改变光照方程中的颜色分量
    - 物体表面的几何法线保持不变，我们修改的只是照明方程中使用的法线值
    - 他们比单独的纹理有更好的三维感官，但是显然还是比不上实际的三维几何体。

  - 以下是各个方法分别的原理和特性说明
    - 1 凹凸贴图 Bump Mapping
      - 凹凸贴图是指计算机图形学中在三维环境中通过纹理方法来产生表面凹凸不平的视觉效果
        - 它主要的原理是通过改变表面光照方程的法线，而不是表面的几何法线
        - 或对每个待渲染的像素在计算照明之前都要加上一个从高度图中找到的扰动
          - 来模拟凹凸不平的视觉特征，如褶皱、波浪等等。

      - Blinn于1978年提出了凹凸贴图方法
        - 使用凹凸贴图，是为了给光滑的平面，在不增加顶点的情况下，增加一些凹凸的变化
        - 他的原理是通过法向量的变化，来产生光影的变化，从而产生凹凸感
        - 实际上并没有顶点（即Geometry）的变化。

      - 表示凹凸效果的另一种方法是使用高度图来修改表面法线的方向
        - 每个单色纹理值代表一个高度，所以在纹理中，白色表示高区域，黑色是低的区域（反之亦然）
        - 图14 波浪高度凹凸贴图以及其在材质上的使用

    - 2 移位贴图 Displacement Mapping
      - 也有人称为置换贴图或称高度纹理贴图（HeightfieldTexturing）
        - 这种方法类似于法线贴图，移位贴图的每一个纹素中存储了一个向量
          - 这个向量代表了对应顶点的位移
          - 注意，此处的纹素并不是与像素一一对应，而是与顶点一一对应
            - 因此，纹理的纹素个数与网格的顶点个数是相等的
          - 在VS阶段，获取每个顶点对应的纹素中的位移向量
            - 施加到局部坐标系下的顶点上，然后进行世界视点投影变换即可

    - 3 法线贴图 Normal Mapping
      - 法线贴图（Normal mapping）是凸凹贴图（Bumpmapping）技术的一种应用
        - 法线贴图有时也称为“Dot3（仿立体）凸凹纹理贴图”
        - 凸凹与纹理贴图通常是在现有的模型法线添加扰动不同，法线贴图要完全更新法线
        - 与凸凹贴图类似的是，它也是用来在不增加多边形的情况下在浓淡效果中添加细节
          - 但是凸凹贴图通常根据一个单独的灰度图像通道进行计算
            - 而法线贴图的数据源图像通常是从更加细致版本的物体得到的多通道图像
            - 即红、绿、蓝通道都是作为一个单独的颜色对待
      - 简单来说，NormalMap直接将正确的Normal值保存到一张纹理中去，那么在使用的时候直接从贴图中取即可。

      - 图15 基于法线贴图的凹凸映射，每个颜色通道实际上是表面法线坐标
        - 红色通道是x偏差;红色越多，正常点越多。
        - 绿色是y偏差，蓝色是z。 右边是使用法线贴图生成的图像。
        - 请注意立方体顶部的扁平外观。

    - 4 视差贴图 Parallax Mapping
      - 又称为 Offset Mapping以及virtual displacementmapping)，于2001年由Kaneko引入，由Welsh进行了改进和推广
      - 视差贴图是一种改进的BumpMapping技术，相较于普通的凹凸贴图，视差贴图技术得到凹凸效果得会更具真实感（如石墙的纹理将有更明显的深度）
      - 视差贴图是通过替换渲染多边形上的顶点处的纹理坐标来实现的
        - 而这个替换依赖于一个关于切线空间中的视角（相对于表面法线的角度）和在该点上的高度图的方程
      - 简单来说，ParallaxMapping利用Height Map进行了近似的Texture Offset
        - 如图 6.32。
      - 图16 视差贴图

    - 5 浮雕贴图 Relief Mapping
      - 关于浮雕贴图（Relief Mapping），有人把它誉为凹凸贴图的极致
        - 我们知道，ParallaxMapping是针对Normal Mapping的改进
          - 利用HeightMap进行了近似的Texture Offset
          - 而Relief Mapping是精确的Texture Offset，所以在表现力上比较完美

    - 图17 法线贴图和浮雕贴图的对比。法线贴图不发生自遮挡。
    - 图18 相较于视差贴图（左），浮雕贴图（右）可以实现更深的凹凸深度。

    - Parallax Mapping能够提供比BumpMapping更多的深度,尤其相比于小视角下
    - 但是如果想提供更深的深度,ParallaxMapping就无能为力了
      - Relief Mapping则可以很好的胜任
    - 相较于ParallaxMapping，浮雕贴图（ReliefMapping）可以实现更深的凹凸深度
    - 浮雕贴图方法不仅更容易提供更深的深度,还可以做出自阴影和闭塞效果,当然算法也稍稍有点复杂
    - 具体细节可以参考这篇中文文献：[Relief  mapping:凹凸贴图的极致](https://link.zhihu.com/?target=http%3A//www.ixueshu.com/document/3dc4369a761ca0d6318947a18e7f9386.html)
    - 而如果要用一句话概括ReliefMapping，将会是：“在Shader里做光线追踪”。
      - 图19 浮雕贴图，使石块看起来更逼真

---