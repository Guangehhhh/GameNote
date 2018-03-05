## RHI
- RHI.h :主要定义一些硬件平台的公共变量
  -  硬件支持项,如是否支持PF_FloatRGBA格式渲染目标,手机平台是否支持FrameBuffer拾取,支持体纹理,支持硬件合并渲染等等.
　-  硬件变量,如最大Cube纹理数,阴影贴图长宽最大值等等.
  -  常见渲染定义,如FSamplerStateInitializerRHI纹理采样
    - FRasterizerStateInitializer
      - RHI栅栏化(填充格式,正方向定义,MSAA)
    - FDepthStencilStateInitializer
      - RHI逐片断处理中的模板与深度
    - FBlendStateInitializer
      - RHI逐片断处理中的混合.FRHIDrawIndirectParameters/FRHIDrawIndexedIndirectParameters DrawCall中相关参数
- DynamicRHI.h
  - 包含FDynamicRHI接口定义，渲染所需求所有接口
    - 创建buffer，创建纹理,设置着色器参数，UAV等
  - 简单来说，对应opengl，dx提供的渲染API
    - 其DynamicRHI.cpp文件会根据平台(Windows,apple,android等等)来选择加载合适的渲染平台(如Opengl,Dx,Vulkan等)
    - 在RHI模块的private文件夹下,可能看到各个系统会如何选择相应的渲染平台.





# FVertexStream

# FRenderResource
- 定义接口如InitDynamicRHI /ReleaseDynamicRHI /InitRHI /ReleaseRHI
- InitResource /ReleaseResource
- UpdateRHI等渲染资源选择实现函数

# FRHICommandBase
- 主要定义一个函数指针，一个执行方法调用函数指针指向的函数
- 函数指针：二个参数(FRHICommandListBase,FRHICommandBase)CallExecuteAndDestruct:传入自己FRHICommandBase到时函数指针指向的方向.
# template<typename TCmd> FRHICommand : public FRHICommandBase
- FRHICommandBase的一个模板子类,模板需要定义Execute方法,其方法只需要FRHICommandListBase
  - 其会退化上面CallExecuteAndDestruct的FRHICommandBase参数,默认为自己.
- FRHICommand的模板具体化,对应SetRasterizerState/SetDepthStencilState/SetShaderParameter等等
  - 几乎所有渲染API都有对应的FRHICommand的模板具体化实现

# FRHICommandListBase
- 有相关和RHI线程交互的API
- FRHICommandBase的链表实现,定义一些上下文如IRHICommandContext ,IRHIComputeContext
- RHI本身相应的FRHICommandBase与List都是存放在渲染线程中
- RHI线程用于在渲染线程中同步执行异步的复杂操作
  - 如压入很多FRHICommandBase到渲染线程中执行
- 有些操作可以放入RHI线程中与渲染线程一起执行
  - 在某段FRHICommandBase前,调用WaitForTasks等同步渲染线程与RHI线程
- RHI线程对于渲染线程就相当于渲染线程与游戏线程的关系
  - 看到如何在渲染线程里压入RHI线程,如何用WaitForTasks与渲染线程同步等

# FRHICommandList : public FRHICommandListBase
-


- 简单来说,所有用于渲染API几乎都有二种方法,一种是插入FRHICommandListBase链表
  - 一种是直接调用相应渲染平台对应FDynamicRHI的实现
  - OpenGLDrv相应的FDynamicRHI实现,相应API如SetShaderParameter, SetDepthStencilState等
  - 并没有直接调用相应的OpenGL的API,而是把相关改动放入一个FOpenGLRHIState的结构中保存起来
  - 等到DrawCall(如RHIDrawPrimitiveIndirect等)相关命令调用后,才把各个改动对应opengl的API调用起来,如上的glProgramUniform等

# FRHIAsyncComputeCommandList
- 上下文和IRHIComputeContext相关，并且提供一些与RHI线程交互的方法

# FRHICommandListImmediate
- 直接调用相应渲染平台对应FDynamicRHI的实现,对比FRHICommandList，不会有链表的实现

# FRHICommandListExecutor
- 简单来说，管理FRHICommandListBase的几个子类单例实现
  - 方便查找到如上的FRHICommandListImmediate 与FRHIAsyncComputeCommandListImmediate 单例实现
- 渲染代码里常见的如FRHICommandList/RHICmdList就是指的是FRHICommandListExecutor::GetImmediateCommandList()



- RHI主要调用都在渲染线程中，不过也可以使用FRHICommandListBase链表与RHI线程来实现一些同步异步操作
- 其中渲染模块中FRHICommandList/RHICmdList一般是FRHICommandListExecutor::GetImmediateCommandList()
  - 这个是直接调用相关FDynamicRHI实现，一般并不与RHI线程交互

- UE4中大量用到C++ 的模版，除开自动生成各个分支代码 ，还有二点
  - 一是代替部分接口类，减少如虚函数表的性能，二是减少一些分支判断，还是提高性能
  - 但是会造成阅读代码比C#等语言验证，主要在于有些模板你都不知道是那些类可以用
  - 还好UE4里一般这种模板使用类都有相同的前缀或是后缀，我们可以记一些相同的前缀或后缀转化成自己认为的接口实现

# FOpenGLDynamicRHI是在DrawCall时，才把各个改动对应opengl的API调用起来
- 所以在这可以看到一个渲染的完整过程，当然大家使用过Opengl或是DX直接写过程序也是一样
  - 首先设定渲染目标混合，设定viewport,设定栅栏化，设定逐片断处理(深度，模板),绑定Shader程序
  - 设定shader纹理，设置shader参数，绑定VAO，设定VAO,DrawCall
  - 嗯就是这么个过程，无论UE4如何包装，每次DrawCall就是如上顺序处理




# Parameters:
- 二个主要方法
  - 一是Bind ，对应一个或多个参数Parameter与Shader代码里参数绑定,对应opengl里的API就是如glGetUniformLocation
  - 二是Set,简单来说，上面绑定后，我们就可以传入参数的值到GPU里，对应opengl里的API就是如glUniform等等
- 模板类里的模板如果是后缀ParametersType,一般主要是指各个后缀为Parameters的类

# FVertexFactory:
- 用来表示顶点数据格式，顶点分布结构，顶点元素Buffer，DeclarationElementList数组
  - 相关opengl的API如glVertexAttribPointer
  - 从opengl3+来说，一般虽然可能有多个buffer,但是应该是在一个glgenbuffer中对应不同的区段而已

# Set:
- 只是告诉对应opengl里各个buffer的起点与终点，相应的如OffsetInstanceStreams/SetPositionStream都是类似

# FVertexFactoryType:
- 表示网格类型，如 Local/Particle(三种sprite/beamtrail,mesh)/Landscape/GPUSkin等

# FMeshBatch:
- 是一组相同顶点格式，相同材质的模型，一般可以使用GPU的实例渲染，减少DrawCall.

# FShaderType:
- Global/Material/MeshMaterial (vertex/hull/demain/geomerty/pixel 一种)

# FGlobalShader:
- 全局shader,简单来说,不和mesh与Material关联，一般用于后处理,固定画个方块啥的，如处理特效这种。

# SetParameters:
- 设定Shader里的FViewUniformShaderParameters /FFrameUniformShaderParameters /FBuiltinSamplersParameters 参数。

# FMaterialShader:
- 特定于过程的着色器，它们需要访问材质的某些属性，因此必须针对每个材质进行编译，但不需要访问任何网格属性
  - 如FLightFunctionVS，FLightFunctionPS等　　
  - 对比FGlobalShader，增加一个重载的SetParameters，包含材质对Shader的设置

# FMeshMaterialShader:
- 着色器是特定于过程的着色器，它们依赖于材质的属性和网格类型，因此必须针对每个材质/FVertexFactory组合进行编译。例如，TBasePassVS / TBasePassPS 需要对前向渲染过程中的所有材质输入进行评估。
  - 对比FMaterialShader，增加一个方法SetMesh,添加FMeshBatchElement,FVertexFactory对shader的设置，对应VertexFactory的Parameters针对Mesh填充不同的顶点信息
    - 如GPUSkin,填充骨骼信息到相应的shader参数中， 如MeshParticle，填充动画加速度 ，时间等
    - 以及填充模型本身的FPrimitiveUniformShaderParameters等共有信息，如FPrimitiveUniformShaderParameters:localToworld ,worldTolocal ,objectBounds, LOD，FadeTimeScaleBias等。

# 如下这些类表示渲染主要思路，预先一些相同的渲染方式，可以先缓存起来。

# FMeshDrawingPolicy:
- 整合渲染模型过程，从绑定Shader到调用DrawCall,各个子类对应不同的独立着色器程序。
  - 1 初始化，根据需要生成或绑定各个对应的Shader.
　- 2 SetSharedState,设定和Mesh无关的Shader变量。
　- 3 SetMeshRenderState,设定和Mesh相关的Shader变量。
　- 4 DrawMesh 调用DrawCall.

# 模板类里的模板如果是DrawingPolicyType
- 一般主要是指FMeshDrawingPolicy的各个子类。

# FUniformLightMapPolicy:
- 封装和光照有关渲染的Shader参数设置　

# SetMesh:
- 绑定相应光照计算上Shader的参数，如使用GI预计算产生的间接光照图信息，直接光照图信息，天空图AO等。

# TUniformLightMapPolicy:
- FUniformLightMapPolicy的模版子类，模版为ELightMapPolicyType，表示各种和光照有关，模版预生成多份代码对应不同光照计算表示是否缓存，Shader预编译指令。

# ELightMapPolicyType

# TLightMapPolicy:
- Shader对应预编译指令，是否缓存，模板为ELightmapQuality，有二个值，分别是LQ_LIGHTMAP，HQ_LIGHTMAP。

# 模板类里的模板如果是LightMapPolicyType
- 一般主要是指TUniformLightMapPolicy/TLightMapPolicy的各个子类

# FMeshDrawingPolicy
- 从上面各个类的说明来看，可以看到把所有渲染基本类组合在一起，他的子类简单说几个，BasePassRendering ,CapsuleShadowing ,DepthRendering ,ForwardBasePassRendering ,VelocityRendering等等
  - 还有别的带Rendering的渲染，如DistortionRendering ,DeferredShading ,DecalRendering，ShadowRendering等等
  - 虽然和FMeshDrawingPolicy不同，但是过程其实真差不了多少
  - 在每个Rendering中，都有对应的VS，PS，HS等，这些根据需要分别从上面所说的FGlobalShader /FMaterialShader /FMeshMaterialShader继承
  - 简单来说，后处理特效针对渲染目标的一般从FGlobalShader继承，只针对Material不和具体Mesh有关的用FMaterialShader，最后针对模型渲染的从FMeshMaterialShader继承

# BasePassRendering
- 只简单渲染emissive color与light map，对应的FMeshDrawingPolicy子类为TBasePassDrawingPolicy
- 如上所说，针对Mesh产生的都继承与FMeshMaterialShader生成的VS，PS等，因为光照有影响，我们看到相应的Shader都对应模版LightMapPolicyType，用于生成正确的Shader对应预编译指令，如有无光照，光照质量，静态或动态，阴影类型等。下面还定义一些与BasePassRendering相关的parameters,如天空盒相关参数，如上TBasePassDrawingPolicy在构造函数中得到或是生成上面的VS,PS，然后在SetSharedState时针对VS,PS设定参数，然后调用SetMeshRenderState针对每个FMeshBatch设定和Mesh有关的参数，然后提交DrawCall.

# 每个DrawingPolicy中，对应VS，PS等对应文件可以通过宏IMPLEMENT_SHADER_TYPE查看。
