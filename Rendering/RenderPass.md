# 虚幻渲染
- ParticleSimulation pass
  - 在GPU上计算了场景里所有的粒子发射器（emitter）的粒子运动以及其他属性
  - 并将结果输出到两个渲染目标（rendertarget）上，一个格式为RGBA32_Float
  - 保存位置，另一个为RGBA16_Float，保存速度以及其他一些和粒子时间/生存周期相关的数据
  - RGBA32_Float格式的渲染目标保存的数据，每一个像素代表了一个sprite的世界坐标
- Z-Prepass
  - 将所有不透明物体渲染到一个R24G8的深度缓冲中
  - UE4使用reverse-Z来保存深度，意味着近裁面的深度值为1，远裁面的深度值为0
  - 这使得深度缓冲的精度更高，避免在远处发生z-fighting的现象
  - 从该pass的名字可以看出这一步是由“DBuffer”触发的
  - DBuffer是UE4用来保存延迟贴花（deferred decal）的缓冲，这一步需要场景深度，所以会启动Z-prepass
  - 这个Z-buffer还会用在其他地方，例如遮挡检测和屏幕空间反射
- BeginOcclusionTests
  - 负责一帧中所有遮挡测试。UE4默认使用硬件遮挡查询（hardware occlusion queries）来进行遮挡测试
  - 分为3步
    - 所有被标记为遮挡体的物体（例如一个较大的solid mesh）渲染进一个深度缓冲
    - 创建一个遮挡查询（occlusion query），提交该查询并渲染那些我们希望查询遮挡情况的模型
      - 这一步使用硬件深度测试（z-test）以及我们在第一步中创建的深度缓冲
      - 遮挡查询将返回通过深度测试的像素数量，如果结果是0就意味着该物体完全被solid mesh遮挡
      - 由于为了深度测试而去把完整的模型渲染一遍的开销很高，这一步我们渲染模型的包围盒，而不是原模型
        - 如果该包围盒不可见（也就是没通过深度测试），那么该包围盒所代表的模型肯定也不可见
    - 将查询结果读回CPU，根据被渲染像素的数量我们决定是否提交模型给GPU渲染
      - （即便是有一小部分像素可见我们也可以不读渲染这个模型）

# 对于CPU与GPU间的同步问题
- E4使用和其他引擎类似的方法：将CPU对数据的读回操作延迟几帧进行
  - 这个方法大部分情况下可行，但在摄像机高速移动的时候可能会导致物体的突然出现（pop in）（
  - 实践中这不是个大问题，因为物体在遮挡剔除时使用包围盒来计算遮挡，这一步是保守，即便完全不可见的物体也可能被标记为可见）
  - 额外的drawcall开销依然存在，但这个问题也是可以解决的，UE4通过以下方法来减轻这个问题的影响：
    - 首先所有物体会被渲染到深度缓冲。（也就是之前提到的这一过程）
    - 对于所有需要遮挡测试的物体向GPU提交一个遮挡查询请求。
    - 在每一帧的最后，CPU从前一帧（或者更加前面的帧）读回物体的可见性结果
      - 如果物体是可见的就将物体标为在下一帧需要渲染
      - 对于不可见的物体，将其加入一个“分组”的查询中，该查询会以批次提交最高8个物体的包围盒组，测试这些物体在下一帧是否可见。
    - 如果整个分组在下一帧变为可见，那么再将整个组重新分离为独立的遮挡查询并提交。
- 如果相机和物体是静止的（或者缓慢移动），这一优化会将必要的遮挡查询数量减少8倍
  - 唯一一个我注意到的奇怪地方是被遮挡物体的批次查询组合方式似乎是随机的，而不是基于物体在空间上的距离

- 这一步对应于上图中的IndividualQueries和GroupedQueries标记
  - GroupedQueries在这一帧是空的，因为UE4没有在前一帧中找到任何需要这一操作的物体

- 在整个遮挡剔除pass的最后，ShadowFrustumQueries提交所有针对本地光源（local light，也就是点光源或者聚光灯）的包围盒的遮挡查询
  - （无论光源是否投影都会提交，和这一步的名字所表达的意思不同），如果某个光源被完全遮挡住了那么就没必要去对该光源进行任何光照/投影计算
  - 值得注意的是我们的示例场景中有4个点光源（每一帧每个光源都需要计算shadowmap）
  - 但是ShadowFrustumQueries这一步提交的查询数量为3
  - 我猜测这是因为其中一个光源的包围盒和相机近裁面相交，因此UE4认为该光源必然可见
  - 另一点值得一提，对于一个需要计算cubemap shadowmap的动态光源，UE4会提交一个球体来进行遮挡测试
  - 对于需要计算逐物体阴影的静止动态光源（之后会有更详细的介绍），UE4会提交一个视锥体来进行遮挡检测
  - 最后对于PlanarReflectionQueries这一步
    - 是指用于计算平面反射（planar reflection）的遮挡剔除计算（方法是将相机变换到渲染平面之后/之下在重新绘制物体）
- Hi-Z缓冲的生成
  - 接下来，UE4会创建一个Hi-Z缓冲（passes HZB SetupMipXX），存储格式为16位浮点数（R16_Float）
  - 这一步将Z-prepass阶段得到的深度缓冲作为输入创建一个深度值的mipmap链（mipmap chain）
  - 这一步还会将深度重新采样为分辨率大小为2的幂次数的纹理，这样用起来更方便
  - 之前提到，由于UE4使用reverse-Z，pixel shader在降采样时使用最小值操作符
    - （注：也就是指每次降采样时选取邻域内深度值最小的像素输出到下一个mipmap）
- 阴影计算render pass(ShadowDepths)
  - 静态（stationary）的平行光，两个可移动（movable）的点光源以及一个静止（static）的点光源
    - 所有光源都会计算阴影
  - 对于静态光源，渲染器会为静态物体烘焙阴影，并为动态物体计算阴影。
  - 对于可移动的光源，每一帧都需要为所有物体计算阴影（完全动态）
    - 最终对于静态物体其阴影会被烘焙入光照贴图（lightmap），所以这些阴影在渲染中不会出现
  - 对于平行光 ，添加了分三个层级的级联阴影（cascaded shadowmaps），以观察UE4是怎么处理这个功能的
    - UE4创建了一个3x1的格式为R16_TYPELESS的纹理（每行3个tile，每层阴影一个）
    - 每一帧清除一次（意味着每一帧所有层都要更新，而不会有隔帧更新之类的优化）
    - 随后，在Atlas0 render pass中所有物体会被渲染进对应的阴影tile中
  - 阴影在渲染时无需pixel shader，这能使得阴影的渲染速度翻倍
  - 值得注意的是无论平行光是静止的还是动态的，渲染器会将所有物体（包括静态物体）都渲染到阴影贴图中
  - 接下来是Atlas1 render pass，这一步将渲染所有静态点光源的阴影
    - 对于静态光源的动态物体，UE4使用逐物体阴影贴图，保存在一个纹理图集（texture atlas）中
      - 意味着对于每一个光源，每一个物体都会渲染一个shadowmap
  - 最后，对于动态光源，UE4使用传统的立方体阴影（cubemap shadowmap，在CubemapXX passes中)
    - 使用一个geometry shader来选择要渲染到cubemap的哪个面上（以减少draw call）
    - 在这一步只渲染动态物体，所有静态物体会被缓存起来
    - CopyCachedShadowMap这一步会把阴影缓存复制进来，然后在此之上渲染动态物体的阴影深度
  - 静态物体的阴影缓存不会再每一帧重新生成，因为渲染器知道（我们场景中的）这一光源没有移动（尽管被标记为动态光源）
    - 如果光源移动了，渲染器会在每一帧渲染动态物体前把所有静态物体重新绘制入阴影缓存中
  - 唯一 一个静态光源（static light）完全没有出现在drawcall列表中，意味着这个光源不会影响动态物体，只会通过光照贴图去影响静态物体





# RenderingThread
    - Run
      - RenderingThreadMain()
        - ProcessThreadUntilRequestReturn
          - ProcessTasksNamedThread
            - FDeferredShadingSceneRenderer::Render

              - FSceneRenderTargets::Get(RHICmdList)
              - GRenderTargetPool.TransitionTargetsWritable(RHICmdList);
              - FRHICommandListExecutor::WaitOnRHIThreadFence(OcclusionSubmittedFence[BlockFrame])
              - RenderOcclusion
              - RenderShadowDepthMaps
              - RenderBasePass
              -

              - InitView
                - ComputeViewVisibility
                  - ComputeAndMarkRelevanceForViewParallel
                    - ComputeRelevance
                      - GetViewRelevance
                        - IsShadowCast
                    - RenderThreadFinalize
                      - UpdateUniformBuffer
                        - UpdateRHI
                          - InitDynamicRHI
                            -
  - RenderTargetPool



# Rendering

# 渲染BOX过程
* 首先在主线程通过调用这个函数向渲染线程发送命令
    * FScene::AddPrimitive
  * 渲染线程会调用这个函数，添加信息到渲染线程可以读取的结构中
    * FScene::AddPrimitiveSceneInfo_RenderThread
  * 在每一帧的render函数中，读取渲染线程可以读取的数据
      * 遍历场景中可见物品，调用到这里
      * 最后调用到D3D的API
        * FMeshDrawingPolicy::DrawMesh
  * 如果一个mesh位置变化了，也会调用FScene里面的更新函数
    * FScene::UpdatePrimitiveTransform
  * 和增加MESH一样，更新也会调用一个渲染线程的代码
      * FScene::UpdatePrimitiveTransform_RenderThread

---
## 渲染StaticMesh过程
* FDeferredShadingSceneRenderer::Render
    * RenderBasePass(RHICmdList)
    * TStaticMeshDrawList<DrawingPolicyType>::DrawVisibleInner
    * FMeshDrawingPolicy::DrawMesh
      * RHICmdList.DrawIndexedPrimitive
    * FD3D11DynamicRHI::RHIDrawIndexedPrimitive


    * 游戏线程是木偶师，渲染线程是木偶
    * 渲染线程具有独立的逻辑系统
    * 渲染线程只负责执行游戏线程的命令
---
## 渲染线程的启动
* 创建渲染线程实例
    * 通过FRunnableThread::Create创建
    * 等待渲染线程准备好从自己的TaskGraph取出任务执行
    * 注册渲染线程
    * 创建渲染线程Tick更新线程

## 渲染线程的运行
  * 渲染线程的主要执行内容在全局函数RenderingThreadMain
    * 渲染线程只是一个机械性地执行任务包的“奴隶”
    * 游戏线程会借助EQUEUE_Render_COMMAND 系列宏
    * 向渲染线程的TaskMap 中添加渲染任务
    * 渲染线程也并非直接向GPU 发送指令
    * 将渲染命令添加到RHICommandList (RHI 命令列表)
    * RHI 线程不断取出指令，向GPU发送，并阻塞等待结果
    * 此时RHI 线程虽然阻塞，但是渲染线程依然正常工作,向RHI 命令列表填充指令
    * 增加CPU 时间的利用率,避免渲染线程凭空等待GPU

## 渲染架构
*  Deferred Rendering
      * 是为了解决场景中由于物体、光照数量带来的计算复杂度问题
        * 延迟渲染则是将“光照渲染”延迟进行
        * BaseColor（基础颜色）、粗糙度、表面法线、像素深度等
        * 分别渲染成图片然后根据这几张图，再逐个计算光照。
      * 对于场景中所有不透明物体的渲染方式



---
## 渲染过程

## 初始化视口
* 此时有必要进行可见性剔除，该函数命名为InitViews
  * 这是为了照顾不止一个视口的情况
* 剔除过程分为三步：预设置可见性，可见性计算，完成可见性计算
* 预设置可见性对应函数为PreVisibilityFrameSetup
    * 设置TemporalAA 的采样方式
    * 设置视口矩阵,视口投影矩阵和转换矩阵
* 可见性计算，对应的函数为ComputViewVisibility
    * 初始化视口的一系列用于可视化检测的缓冲区
      * 排除特殊情况，大多数情况下视口都使用平截头体剔除，FrustumCull
      * 过小的线框直接剔除掉
      * 处于视口范围内、但是被别的对象遮挡的对象进行一次剔除
      * 根据所有的可见性位图，设置每个需要渲染的对象的可见性状况，即Hidden flags
      * 接下来会给每个对象机会来返回自己是否可见
      * 最后一步是获取所有动态对象的渲染信息
        * 对应函数是每个RenderProxy 的Get-DynamicMeshElements 函数
* 完成可见性计算，对应的函数为PostVisibilityFrameSetup。
    * 对半透明的对象进行排序
      * 对每个光照确定当前光照可见的对象列表，这里也是使用平截头体剔除
      * 初始化雾与大气的常量值

## PrePass 预处理阶段
* PrePass 的主要目的是为了降低Base Pass 的渲染工作量
  * 即渲染一次深度信息,如果某个像素点的深度检测失败
    * 即不符合当前深度值，那么这个像素将不会进行像素渲染器计算  
  * PrePass 过程是可选的
    * 取决于NeedsPrePass 函数的返回结果
    * 不是基于分块的GP
    * 渲染器的EarlyZPassMode 参数不为DDM_None，或GEarlyZPassMovable 不为0
#两者必须同时满足，才会进行PrePass 计算**
  * 此时参与渲染的包括不透明对象和Mask 对象
    * 静态网格物体构成了一套列表，即Scene 的DepthDrawList 与MaskedDepthDrawList
  * 对象的渲染始终按照
    * 设置渲染状态
    * 载入着色器
    * 设置渲染参数
    * 提交渲染请求
    * 写入渲染目标缓冲区的步骤进行
  **预处理阶段的工作过程如下**
  * 设置渲染状态 这个步骤对应函数SetupPrePassView
    * 目的在于关闭颜色写入，打开深度测试与深度写入。
  * 渲染三个绘制列表DrawList
    * 只绘制深度的列表PositionOnlyDepthDrawList
      * 这个列表里的对象只在深度渲染过程中起作用
    * 深度绘制列表DepthDrawList
      * 这是最主要的不透明物体渲染列表
    * 带蒙版的深度绘制列表MaskedDepthDrawList
      * 蒙版即对应材质系统中的Mask类型材质
    * 绘制动态的预处理阶段对象
      * 这个阶段会通过ShouldUseAsOccluder 函数询问RenderProxy 是否被当作一个遮挡物体
      * 同时也会配合其他情况（是否为可移动等），决定是否需要在这个阶段绘制

## DrawVisible 绘制可见对象  在哪一步？？
- 绘制可见对象的基础是可见对象列表
  - 尤其是对于静态网格物体而言，是以TStaticMeshDrawList 为单位进行成批绘制的
  - 在绘制之前，每个绘制列表已经进行了排序，尽可能共用同样的绘制状态
  - 每个列表都共用以下着色器状态
    * 顶点描述Vertex Declaration
    * 顶点着色器Vertex Shader
    * 壳着色器Hull Shader
    * 域着色器Domain Shader
    * 像素着色器Pixel Shader
    * 几何着色器Geometry Shader
  - 着色器状态看作是顶点描述和整个渲染管线用到的着色器对象的集合
  - 而同一个列表中，这些东西都是一致的，区别只是在于渲染过程具体参数
  - 如顶点缓冲区不同、索引缓冲区不同

- 载入公共着色器信息
  -  这是逐列表完成的
  - 对应的函数为SetBoundShaderState 和SetSharedState
    - SetBoundShaderState 载入了需要的着色器
    - SetSharedState 则因不同类型而异
      - 比较有代表性的TBasePass 而言，是设置顶点着色器和像素着色器的参数
      - 如果是透明物体
        - 即透明度设置为：Additive、Translucent、Modulate 的，会在此时再次设置混合模式。

  逐元素渲染
    * 需要注意的是，此时的元素并非是一比一对应，而是经过组合的
    * 即并非Element，而是BatchElement。
    * (1) 对每个DrawingPolicy 调用SetMeshRenderState 函数，设置渲染状态
      * 调用每个着色器的SetMesh 函数，以设置与当前Mesh 相关的参数，如变换矩阵
    * (2) 调用当前Batch Element 的DrawMesh 函数，实际完成绘制
      * 顶点缓冲区和索引缓冲区是一开始就已经上传到显存准备好了的
      * 只需要调用RHICmdList 的DrawIndexedPrimitive 函数
        * 指定好顶点缓冲区和索引缓冲区的位置，就可以了
        * 这两个位置是包含在当前Element 持有的信息里面的

## BasePass

* BasePass 是一个极为重要的阶段
  * 这个阶段完成的是通过逐对象的绘制，将每个对象和光照相关的信息都写入到缓冲区中
* BasePass 的过程和PrePass 的过程非常接近
  * 它们都按照设置渲染视口、渲染静态数据、渲染动态数据三个步骤完成
* BasePass 阶段采用了MRT(Multi-Render Target) 多渲染目标技术
* 允许Shader 在渲染过程中向多个渲染目标进行输出
* 这几个渲染目标从哪里来的？答案是由当前请求渲染的视口（Viewport）分配的
  * 并不归属SceneRenderer分配
  * 对应的是FSceneViewport::BeginRenderFramw 函数。
* 如何向多个渲染目标写入？
  * 这个问题的答案并不在C++ 代码中，而是在Shader 着色器代码中
  * 打开Engine/Shader/BasePassPixelShader.usf 文件，就会看到著名的Main 函数
  * 定义了用到的多个输出（不考虑特殊情况）：SV_Target0 输出了颜色数据
  * SV_Target1-3 则作为GBuffer 输出目标
  * 由于这几个渲染目标会一直存在于当前的渲染上下文中，所以在下个阶段可以直接使用

* 换句话说类似于炒菜的时候，最开始下单的那个服务员会给一个盘子，
负责每个步骤的人会从上一个步骤的人那接过盘子，在自己处理过后
继续传递给下个人。这几个渲染目标会放在盘子里，不断向下传递。*

* 设置渲染视口
* (1) 如果PrePass 阶段已经写入深度，则深度写入被关闭，直接使用已经写入的深度
结果进行深度检测。
* 通过RHICmdList . SetBlendState ( TStaticBlendStateWriteMask < CW_RGBA ,
CW_RGBA , CW_RGBA , CW_RGBA >:: GetRHI () );
  * 打开了前4 个渲染目标的RGBA 写入。
  * 设置了视口区域大小。这个大小会因为是否开启InstancedStereoPass 而有所变化


* 渲染静态数据
* 与PrePass 有一定区别的是，如果之前已经进行了深度渲染
* 那么会首先渲染Masked 蒙版对象，然后渲染普通的不透明对象
* 否则就会反过来，先渲染不透明对象，再渲染蒙版对象

* 渲染动态数据
  * 区别不大。

## RenderOcclusion 渲染遮挡
- 虚幻引擎的遮挡计算
  * 实质上是在PrePass 中直接进行基于并行队列的硬件遮挡查询
  * 有些特效需要的情况下，才会开启Hierarchical Z-Buffer Occlusion Culling 用作遮挡查询
* Hierarchical Z-Buffer Occlusion Culling 为关键词搜索
* 这个步骤是为了尽可能剔除处于屏幕内但是被其他对象遮挡的对象
* HZB 渲染遮挡技术被用于解决这个问题
  * 预先准备屏幕的深度缓冲区，这个缓冲区将会作为深度测试的基础数据
  * 逐层创建缓冲区的Mipmap 级联贴图
    * 层级越高，贴图分辨率越低，对应的区域越大
  * 计算所有需要进行测试的对象的包围球半径
  * 根据这个半径，选择对应的深度缓冲区层级进行深度测试，判断是否被遮挡


## 光照渲染

- 对应的函数是RenderLights
  * 光照渲染与阴影渲染是分离的
    * 阴影渲染是在视口初始化阶段完成，处于整个渲染流水线中非常靠前的位置

- 阴影渲染？
  - 步骤？？
  -   收集可见光源  
  -  对收集好的光源进行排序
  -  如果是存在基于图块的光照（TiledDeferred-Lighting）
    - 则通过RenderTiledDeferredLighting 对光照进行计算
    - 如果是PC 平台
      - 大部分光照依然使用RenderLight 函数进行光照计算
  -  4如果当前平台支持着色器模型5（Shader Model 5）
    - 则会计算反射阴影贴图与LPV 信息

- 核心光照渲染部分集中于RenderLight 函数
  - 每个光源都会调用该函数，其遍历所有视口，计算光照强度，并叠加到屏幕颜色上

- RenderLight函数
  * 设置混合模式为叠加
  * 判断光源类型
  * 平行光
    * 载入延迟渲染光照对应的顶点和像素着色器
    （对应为TDeferredLightVS 和TDeferredLightPS）
    * 设置光照相关参数
    * 绘制一个覆盖全屏幕的矩形，调用着色器

  - 非平行光
   * 判断摄像机是否在光照几何体范围内
   * 如果是，关闭深度测试，从而避免背面被遮盖部分不进行光照渲染
   * 否则，打开深度测试以加速渲染
   * 载入着色器
   * 设置光照相关参数
   * 根据是点光源还是聚光灯，绘制一个对应的几何体
   * 从而排除几何体外对象的渲染，加速光照计算


##渲染着色器数据提供

- 虚幻引擎的着色器数量
  * 虚幻引擎的着色器数量高得令人恐怖
  * 单个材质会编译出多个着色器

- 着色器按照什么样的方式被归类？
  * 如果当前着色器类型继承自FMaterialShader
    * 则对每个材质类型会编译出一组对应渲染管线的着色器，一般是顶点着色器、像素着色器组合
  * 如果当前着色器类型继承自FMeshMaterialShader
  * 则对每个材质类型的每个顶点工厂类型编译出一组顶点着色器和像素着色器

- 顶点工厂负责抽象顶点数据以供后面的着色器获取，从而让着色器能够忽略由于顶点类型造成的差异
  - 比如普通的静态网格物体和使用GPU 进行蒙皮的物体，两者顶点数据不同，但是通过顶点工厂进行抽象后，
  - 提供统一的数据获取接口，供后面的着色器调用

- 虚幻引擎的“材质”
  * 实质上是提供了一系列的函数接口，调用这些函数接口就能获得对应材质参数
  * 单个材质不会产生一个完整的、带有具体执行过程的顶点、像素着色器

* 对于继承自FMeshMaterialShader 的着色器类型
  * 着色器不存在“动态链接”的概念，不可能出现动态切换某一部分函数的情况
  * 由于继承自FMeshMaterialShader 的着色器类型需要适配不同的顶点工厂类型
  * 也需要适配不同的材质类型，对于顶点工厂+ 材质类型的组合，
  * 只要有一项不同，就必须产生新的一份着色器代码用于编译

* 因此，我们可以从两个维度来看待这个被虚幻引擎官方文档称为“稀疏矩阵”的着色器方案
  - 从渲染阶段（PrePass、BasePass 等）角度来看
    * 每个渲染阶段对应的着色器（如TBasePassVS/PS）都需要对每个顶点工厂, 每个材质类型产生出一组着色器，放置在对应材质类型的着色器缓存中
  - 从材质类型角度来看
    - 每个材质根据自身参与的渲染阶段不同，需要对每个顶点工厂类型 ,每个自己参与的渲染阶段 产生出一组着色器，放置在自己的着色器缓存中。此时被称为ShaderMap。

* 这就形成一套“三维”的着色器矩阵
  * 你可以想象一个三维魔方
    * 长度为每个材质类型
    * 宽度为每个渲染阶段
    * 高度为每个顶点工厂类型
    * 那么这个魔方的每一个方格都对应了一组着色器组合
    * 当然由于材质不一定参与全部阶段，这个魔方有很多的空缺

## 着色器数据选择
* 如何根据当前阶段、当前材质类型、当前顶点工厂类型，获得需要的着色器组合呢?
  * 以一个静态网格物体渲染为例，对着色器数据选择的过程大体描述如下
    * 渲染线程遍历当前场景，添加静态网格到渲染列表
    * 当一个静态网格FStaticMesh 被添加到渲染列表时
      * 其会根据自己参与的渲染阶段
      * 通过F< 渲染阶段（如BasePass) >DrawingPolicyFactory 的AddStaticMesh 函数
      * 开始选取当前渲染阶段对应的着色器
    * 通过当前静态网格的MaterialRenderProxy 材质渲染代理成员变量的GetMaterial函数
      * 获得对应的FMaterial 指针，该指针将会作为材质筛选的依据
      * 随后调用Process< 渲染阶段>Mesh，例如在BasePass 调用的是ProcessBasePassMesh
    * 该函数的最后一个参数会传入一个结构体，描述了一组渲染动作
      * 其对应的Process 函数会根据传入的光照贴图代理类型来做出不同的渲染方式
      * 在此暂不分析光照贴图相关内容。其调用了AddMesh 函数

# 在参数中创建了最重要的DrawingPolicy 绘制代理
    -  这个代理的类型即对应渲染阶段
    -  传入参数包括顶点工厂指针和材质代理指针。
    -  至此，前文描述中的三个决定因素：渲染阶段、顶点工厂类型、材质类型均齐备
    -  调用Get< 渲染阶段>Shaders 函数：
        * 该函数调用当前传入的材质类型的GetShader 模板函数
          * 从材质对象中抽取出对应的着色器，赋值给对应阶段的着色器变量
      -  材质（FMaterial）的GetShader 函数则首先以当前的顶点工厂类型的id 为索引
          - 通过GetMeshShaderMap 函数从OrderedMeshShaderMaps 成员变量中查询到对应顶点工厂类型的MeshShaderMap
          -  随后，调用当前的MeshShaderMap 的GetShader 函数, 以当前着色器类型为参数，查询到实际对应的着色器
        -  实质上获取一组着色器组合需要的三个变量：
          - 渲染阶段
            - 直接用代码区分，以模板形式完成定义，例如通过：
            - TBasePassPS < TUniformLightMapPolicy < Policy >, true >
          - 顶点工厂类型
            - 获取于当前网格物体的Material 参数
            - 实质上拉取材质就是从需要绘制的网格物体获取对应材质
            - 再直接调用材质对应的GetShader 函数完成。
          - 材质类型
            - 获取于当前网格物体的VertexFactory 参数
            - 传给上文提到的材质对应的GetShader 作为参数



##  FPrimitiveSceneProxy
- 以对象为维度理解渲染过程
- 逻辑的世界与渲染的世界
- 无论是Static Mesh Actor，还是这个Actor 持有的Static Mesh Component 组件
  - 均不会去处理“渲染”相关的代码，而是提供一个用于渲染的“影子”
  - 这个影子来完成具体的顶点构造、三角形填充等任务
- 游戏线程遍历所有的逻辑对象，对于每个逻辑对象中可以显示的组件
  - a. 创建一个FPrimitiveSceneProxy 场景代理，完成场景代理初始化。
  -b. SceneProxy 会根据对象位置等，更新自己用于渲染的信息
- 在合适的时机收集SceneProxy 对象，并创建渲染指令，添加到渲染线程的渲染
指令列表末尾。
- 渲染线程不断从渲染指令列表中取出渲染指令进行渲染
- 这个世界的更新是依赖于当前View 的，也就是观察视角
- 根据观察视角，有些逻辑对象将不会创建自己的渲染内容
- 同样地，根据在屏幕上的大小，逻辑对象可以在诸多LOD 中进行选择，以降低渲染的压力
- 渲染线程只会参考SceneProxy进行渲染，而不会关注原始的对象的情况

## 渲染代理的创建
- virtual FPrimitiveSceneProxy * CreateSceneProxy ()
  - 组件注册到世界中的时候被调用
- 在自己创建的组件类中重载此函数，即可返回自己的SceneProxy
  - 而此时你拥有极大的自由度
  - 你可以引入和提交Shader，你可以手动提交顶点缓冲区和索引缓冲区，手动请求绘制等


## 渲染代理的更新
- 渲染代理的更新分为两个部分进行
  - 一个是静态的部分
    - 通过重载DrawStaticElements函数完成
  - 另一个是动态的部分
    - 通过重载GetDynamicMeshElements 完成
  - 其中静态部分的更新更加节省资源，动态部分则赋予了更高的自由度

- 静态部分更新遵循以下的规则
  - DrawStaticElements 函数只会在场景收集的时候被调用，以收集静态物体
    - 其通常无法在接下来的渲染过程中得到更新
  - 当静态对象被移动、旋转或者缩放的时候，OnTransformChanged 函数会被调用
    - 同时DrawStaticElements 函数会被重新调用以更新
- 动态部分的更新
  - GetDynamicMeshElements 在每次场景开始渲染的时候
    - 就会由FDeferredShading-SceneRenderer 通过FSceneRenderer 调用
  - GetDynmaicMeshElements 会根据当前视角角度，提交需要的Element

- 不管是静态部分的更新，还是动态部分的更新，最终都不是直接对对象进行绘制
  - 而是将Mesh 信息输入到传入的FMeshElementCollector 中
  - 最后会由渲染器统一进行渲染。

---


## FSceneView
- 从场景空间投影到2D屏幕区域
- FViewUniformShaderParameters
  - 视图参数的统一缓冲区 这只会在渲染线程的FSceneView副本中初始化
