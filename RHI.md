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
# Set:
- 只是告诉对应opengl里各个buffer的起点与终点，相应的如OffsetInstanceStreams/SetPositionStream都是类似
# SetParameters:
- 设定Shader里的FViewUniformShaderParameters /FFrameUniformShaderParameters /FBuiltinSamplersParameters 参数。

# SetMesh:
- 绑定相应光照计算上Shader的参数，如使用GI预计算产生的间接光照图信息，直接光照图信息，天空图AO等



# FMeshBatch:
- 是一组相同顶点格式，相同材质的模型，一般可以使用GPU的实例渲染，减少DrawCall.

# FShaderType:
- Global/Material/MeshMaterial
  - (vertex/hull/demain/geomerty/pixel )
# Shader
  - In Unreal a shader is a combination of HLSL code (in the form
  of .ush/.usf files) and the contents of a material graph
  - FShader is paired with an FShaderResource
    - which keeps track of the resource on the GPU
  - Vertex Output from a FLocalVertexFactory
    - Images haven’t loaded yet.
    - Please exit printing, wait for images to load,
and try to print again.
associated with that particular shader. An FShaderResource can be
shared between multiple FShaders if the compiled output from the
FShader matches an already existing one.

# FGlobalShader:
- 全局shader,简单来说,不和mesh与Material关联，一般用于后处理,固定画个方块啥的，如处理特效这种
- When a Shader class derives from FGlobalShader
  - it marks them to be part of the Global recompile group
    - (which seems to mean that they don’trecompile while the engine is open!)
-  Only One Instance should Exist
  - which means that you can’t have per-instance parameters


# FMaterialShader:
-  these classes allow multiple instances
- each one associated with its own copy of the GPU resource
-  SetParameters function
  - which allows the C++ code of your shader to change the values of bound HLSL parameters.
  -  Parameter binding is accomplished through the
  - FShaderParameter / FShaderResourceParameter classes
    - and can be done in the Constructor of the shader,
    - see FSimpleElementPS for an example.
  -  The SetParameters function
    - is called just before rendering something with
      - that shader and passes along a fair bit of information
      - —including the material
        - which gives you a great deal of information
        - to be part of your calculations for parameters
          - that you want to change
- 特定于过程的着色器，它们需要访问材质的某些属性，因此必须针对每个材质进行编译，但不需要访问任何网格属性
  - 如FLightFunctionVS，FLightFunctionPS等　　
  - 对比FGlobalShader，增加一个重载的SetParameters，包含材质对Shader的设置

# FMeshMaterialShader:
-  these classes allow multiple instances
- each one associated with its own copy of the GPU resource
- This adds the ability for us to set parameters in our shader
  - before we draw each mesh
-  A significant number of shaders derive from this class as it is the base class of all
shaders that need material and vertex factory parameters (according to
the comment left on the file)
- This simply adds a SetMesh function
  - which is called before each mesh is drawn with it
  -  allowing you to modify the parameters on the GPU to suit that specific mesh.
- Examples: TDepthOnlyVS , TBasePassVS , TBasePassPS
- 着色器是特定于过程的着色器，它们依赖于材质的属性和网格类型，因此必须针对每个材质/FVertexFactory组合进行编译。例如，TBasePassVS / TBasePassPS 需要对前向渲染过程中的所有材质输入进行评估。
  - 对比FMaterialShader，增加一个方法SetMesh,添加FMeshBatchElement,FVertexFactory对shader的设置，对应VertexFactory的Parameters针对Mesh填充不同的顶点信息
    - 如GPUSkin,填充骨骼信息到相应的shader参数中， 如MeshParticle，填充动画加速度 ，时间等
    - 以及填充模型本身的FPrimitiveUniformShaderParameters等共有信息，如FPrimitiveUniformShaderParameters:localToworld ,worldTolocal ,objectBounds, LOD，FadeTimeScaleBias等。

# IMPLEMENT_MATERIAL_SHADER_TYPE
- This macro binds the C++ class FDepthOnlyPS to the HLSL code
located in /Engine/Private/DepthOnlyPixelShader.usf . Specifically, it
associates with the entry point “Main”, and a frequency of SF_Pixel.
Now we have an association between our C++ code ( FDepthOnlyPS ),
the HLSL file it exists in ( DepthOnlyPixelShader.usf ) and which
function within that HLSL code to call ( Main ). Unreal uses the term
“Frequency” to specify what type of shader is it—Vertex, Hull,
Domain, Geometry, Pixel, or Compute.
# Caching and Compilation Environments
- ShouldCache
  - Unreal will automatically compile the many many possible permutations of a shader for you when you modify a material.
  - can be implemented in an FShader , FMaterial or FVertexFactory class
- Unreal will only create a particular permutation of a shader if the Shader, Material and Vertex Factory all agree that that particular permutation should be cached
- ModifyCompilationEnvironment/SetupMaterialEnvironment/
  - ModifyCompilationEnvironment
  - These functions are called before the shader is compiled
  - and lets you modify the HLSL preprocessor defines
  -  FMaterial uses this extensively to set shading model related   - defines based on settings within that material to optimize out any unnecessary code
---
# FVertexFactory:
- 用来表示顶点数据格式，顶点分布结构，顶点元素Buffer  
  - DeclarationElementList数组
  - 相关opengl的API如glVertexAttribPointer
- Uses a Vertex Factory to control what data gets uploaded to the GPU for the vertex shader
- A Vertex Factory is a class that encapsulates a vertex data source and is linked to the input on a vertex shader.
- Static Meshes, Skeletal Meshes, and Procedural Mesh Components all use different Vertex Factories.
- FLocalVertexFactory
  -  it provides a simple way to transform explicit vertex  
    - attributes from local to world space.
    - Static meshes use this,
      - but so do cables and procedural meshes.
    - Skeletal Meshes (which need more data) on the other hand use
- FGPUBaseSkinVertexFactory
  - Further down we look at how the shader data
    - that matches these two vertex factories has different data
contained within them


# FVertexFactoryType:
- 表示网格类型，如 Local/Particle
  - 三种sprite/beamtrail,mesh)/Landscape/GPUSkin等


# FPrimitiveSceneProxy
- FPrimitiveSceneProxy is the render thread version of a UPrimitiveComponent
- Unreal has a game thread and a render thread
  - and they shouldn’t touch data that belongs to the other thread
  - (except through a few specific synchronization macros).
  -  To solve this Unreal uses UPrimitiveComponent for the Game thread and it decides
    - which FPrimitiveSceneProxy class it creates by overriding the CreateSceneProxy() function.
  - The FPrimitiveSceneProxy can then query the game thread (at the appropriate time)
    - to Get data from the game thread onto the render thread
    - so it can be processed and placed on the GPU
-  Examples: UCableComponent / FCableSceneProxy
  - UImagePlateFrustrumComponent / FImagePlateFrustrumSceneProxy
  - FCableSceneProxy the render thread looks at the data in the UCableComponent
  - and builds a new mesh (calculating position, color,etc.)
  - which is then associated with with the FLocalVertexFactory from earlier.
  - UImagePlateFrustrumComponent is neat
    - because it doesn’t have a Vertex Factory at all!
    - It just uses the callbacks from the render thread to calculate some data
    -  and then draws lines using that data.
    - There is no shader or vertex factory associated with it,
    - it just uses the GPU callbacks to call some immediate-mode style rendering functions
# IMPLEMENT_VERTEX_FACTORY_TYPE
- (FactoryClass, ShaderFilename,
bUsedWithmaterials, bSupportsStaticLighting,
bSupportsDynamicLighting, bPrecisePrevWorldPos,
bSupportsPositionOnly)
- example  FVertexFactoryInput
- This data structure is defined in LocalVertexFactory.ush to have specific meaning.
- However, GpuSkinVertexFactory.ush also defines this struct!
-  Then, depending on which header gets included, the data being provided to the vertex shader changes.
- This pattern is repeated in other areas and will be covered in more depth in the Shader Architecture article
#  Shader Pipelines
- shader pipeline” where it handles multiple
shaders (vertex, pixel) together in a pipeline so it can look at the
inputs/outputs and optimize them out
- They’re used in three places in
the engine: DepthRendering, MobileTranslucentRendering, and
VelocityRendering
-
# Drawing Policy
-  Drawing Policy figures out which specific shader to use for a given material and vertex factory
-  refer to a class that contains the logic to render specific meshes with specific shaders
- Drawing Policies do not tend to know specifics about the shaders or meshes they’re rendering

# FDepthDrawingPolicy
- The depth drawing policy is a good example of how simple a drawing policy can be
- In its constructor, it asks the material to find it a shader of a specific type for a specific vertex factory
- TDepthOnlyVS is an implementation of the FMeshMaterialShader
  - and uses the appropriate macro to declare itself as a shader
  - Unreal handles trying to compile all possible permutations of material/shader/vertex factory
  - so it should be able to find this
  - It looks at the material it’s supposed to draw to determine
    - if that material has tessellation enabled or not
      - if it does have it enabled then the depth drawing policy looks for a hull and domain shader:
- Drawing Policies also have the ability to set parameters on the shaders
  - through the SetSharedState and SetMeshRenderState functions,
  - though they usually just pass these on to the currently bound shaders
# FBasePassDrawingPolicy
- First thing they do is they make a template FBasePassDrawingPolicy
  -  template<typename LightMapPolicyType>
    - class TBasePassDrawingPolicy : public FBasePassDrawingPolicy
  -  and the constructor simply calls another template function.
  - That in turn calls another template function,
    - but this time with a specific enum for each lighting type
- Now that they know what lighting policy
  - they’re trying to get a shader for
  -  they use the same  InMaterial.GetShader function
    - as the Depth drawing policy does
    - but this time they’re getting a shader class which is templated!
    - VertexShader = InMaterial.GetShader<TBasePassVS<TUniformLightMapPolicy<Policy>,false> >(VertexFactoryType);
- The first macro is IMPLEMENT_BASEPASS_VERTEXSHADER_TYPE
  - which registers vertex, hull and domain material shaders
  -  (using the IMPLEMENT_MATERIAL_SHADER_TYPE macro we talked about in the section on Shaders)
  - for a given LightMapPolicyType, and LightMapPolicyName by creating new typedefs.
  -  So now we know that calling IMPLEMENT_BASEPASS_VERTEXSHADER_TYPE registers vertex shaders for us.
- The second macro is IMPLEMENT_BASEPASS_LIGHTMAPPED_SHADER_TYPE
  - which takes the LightMapPolicyType, and LightMapPolicyName
  - and calls IMPLEMENT_BASEPASS_VERTEXSHADER_TYPE
  -  and IMPLEMENT_BASEPASS_PIXELSHADER_TYPE
    - (which we didn’t talk about, but works in the same way as the vertex one).
  - This macro therefor lets us create a full shader chain (vertex and pixel) for any given LightMap.
  - Finally Unreal calls this macro 16 different times,
    - passing in different combinations of LightMapPolicyTypes and LightMapPolicyNames.
-  At one point during the call of InMaterial.GetShader<…> functions from earlier
  -  one of the functions had a big switch statement for each LightMapPolicyType to return the right one.
  -  So we know that Unreal is declaring all of our variations for us

#  Drawing Policy Factory  
-  Drawing Policy Factory examine/检查 the state of the material or vertex factory
  - and then can create the correct drawing policy
- Drawing Policy Factory is more type specific and handles creating a Drawing Policy
  - for each object that should be rendered and adding them to different lists for later
# FDepthDrawingPolicyFactory
- There’s only three functions
- AddStaticMesh ,DrawDynamicMesh and DrawStaticMesh .
-  When AddStaticMesh is called the Policy Factory looks at settings about the material and asset
  - that is to be drawn and determines the appropriate/适当 Drawing Policy to create.
-  Then, Unreal puts that drawing policy into a list inside of the FScene which is about to be drawn

- For example, the FDepthDrawingPolicyFactory looks at the material to see if it modifies the mesh position.
  - If it modifies the mesh position then it creates a FDepthDrawingPolicy
    - and adds it to the “MaskedDepthDrawList” inside of FScene .
  -  If the material does not modify the mesh position then instead it creates a FPositionOnlyDepthDrawingPolicy
    - (which looks for different shader variations!)
    - and adds it to a different list in the FScene
- The FDepthDrawingPolicyFactory also has the ability to draw a given mesh batch
  - which again examines the settings and creates a drawing policy.
  - However, instead of adding it to a list it instead sets up the state for the GPU via the RHI layer
    - and then calls another drawing policy to actually draw the mesh



# Telling the the Drawing Policy Factory to Draw
- Finally we learn the root of this and see how all of these pieces come
into play. Remember how there was no shared base class for Drawing
Policies, or Drawing Policy Factories? We’ve reached the point where
the code just knows about them specifically and calls them at different
times.
FStaticMesh::AddToDrawLists
Our FDepthDrawingPolicyFactory had a function called
AddStaticMesh so it’s no surprise that the class that creates it is related
to static meshes! When AddToDrawLists gets called it examines the
asset and project settings to decide what to do with it. The first thing it
does is call FHitProxyDrawingPolicyFactory::AddStaticMesh , and then
FShadowDepthDrawingPolicyFactory::AddStaticMesh , and then
FDepthDrawingPolicyFactory::AddStaticMesh and finally
FBasePassOpaqueDrawingPolicyFactory::AddStaticMesh and
FVelocityDrawingPolicyFactory::AddStaticMesh , whew!
So we know when FStaticMesh is marked to be added to draw lists it
creates a wide variety of Drawing Policy Factories (who then create
Drawing Policies and add them to the correct list). The specifics of how
this function is called aren’t terrible important (though see
FPrimitiveSceneInfo::AddStaticMeshes and go up from there), but we
know that something has to tell the depth pass to draw before the base
pass as well as doing shadows, etc.
Enter FDeferredShadingRenderer , a massive class that handles getting
everything drawn in the right order.
FDeferredShadingRenderer::Render kicks off the whole process and
controls the order of the render operations. We’ll look at the base pass
drawing policy factory; The Render function calls
FDeferredShadingSceneRenderer::RenderBasePass which in turn calls
FDeferredShadingSceneRenderer::RenderBasePassView which calls
FDeferredShadingSceneRenderer::RenderBasePassDynamicData which
finally calls our
FBasePassOpaqueDrawingPolicyFactory::DrawDynamicMesh in a loop,
passing a different mesh to it each time



# Changing Input Data
- Different types of meshes will ultimately need different data to accomplish what they do
  - ie: GPU skinned verts need more data than simple static meshes
  -  Unreal handles these differences on the CPU side with FVertexFactory
    - but on the GPU side it’s a little trickier.
    - Because all Vertex Factories share the same vertex shader
      - (for the base  pass at least) they use a generically named input structure FVertexFactoryInput .
    - Because Unreal is using the same vertex shader
      - but are including different code for each vertex factory
    -  Unreal redefines the FVertexFactoryInput structure in each vertex factory.
    -  This struct is uniquely defined in GpuSkinVertexFactory.ush,LandscapeVertexFactory.ush, LocalVertexFactory.ush and several others.
    - Obviously including all of these files isn’t going to work—instead
      - BasePassVertexCommon.ush includes/Engine/Generated/VertexFactory.ush.
        - This is set to the correct Vertex Factory
        - when the shader is compiled which allows the engine to know which implementation of FVertexFactoryInput to use.
      -  We talked briefly about using a macro to declare a vertex factory in part 2
        - and you had to provide a shader file—this is why.
      - So now the data input for our base pass vertex shader matches the type of vertex data we’re uploading.
      -  The next issue is that different vertex factories will need different data interpolated between the VS and PS.
      - Again, the BasePassVertexShader.usf calls generic functions
        - GetVertexFactoryIntermediates , VertexFactoryGetWorldPosition ,GetMaterialVertexParameters .
        - If we do another Find in Files we’ll discover that each * VertexFactory.ush has defined
          - these functions to be unique to their own needs.
# Changing Output Data
- Now we need to look at how we get data from the Vertex Shader to the Pixel Shader.
-  Unsurprisingly, the output from BasePassVertexShader.usf is another generically named struct
( FBasePassVSOutput ) who’s implementation depends on the vertex factory.
-  There’s a little snag here though—If you have Tessellation enabled there’s two stages between the Vertex shader and Pixel shader
(the Hull and Domain stages), and these stages need different data than if it was just the VS to PS.
- Enter Unreal’s next trick. They use #define to change the meaning of FBasePassVSOutput
  - and it can either be defined as the simple
    - FBasePassVSToPS struct, or for tessellation, FBasePassVSToDS (this code can be found in BasePassVertexCommon.ush).
-  The two structures have nearly the same contents, except the Domain Shader version adds a few extra variables.
- Now, what about those unique per-vertex factory interpolations we needed?
  - Unreal solves this by creating FVertexFactoryInterpolantsVSToPS
  - and FBasePassInterpolantsVSToPS as members of the FBasePassVSOutput .
- Surprise! FVertexFactoryInterpolantsVSToPS is defined in each of the * VertexFactory.ush files
  - meaning that we’re still passing the correct data between stages,
  - even if we stop to add a Hull/Domain stage in the middle.
- FBasePassInterpolantsVSToPS isn’t redefined as the stuff stored in this struct doesn’t depend on anything unique to a specific
vertex factory, holding things like VertexFog values,AmbientLightingVector, etc.
- Unreal’s redefinition technique abstracts away most of the differences
in the base pass vertex shader, allowing common code to be used regardless of tessellation or specific vertex factory.

# Base Pass Vertex Shader
-  what each shader in the Deferred Shading pipeline actually do.
- The BasePassVertexShader.usf ends up being pretty simple overall./非常简单
- For the most part the Vertex Shader is simply calculating and assigning /计算和分配the BasePassInterpolants and the VertexFactoryInterpolants,
  - though how these values are calculated gets a bit more complicated
  - there’s lots of special cases /有许多特殊情况
    - where they’ve chosen to only declare certain interpolators under certain preprocessor defines
    - and then only assign those under matching defines. /只分配给匹配的定义
- For example, near the bottom of the Vertex Shader we can see a define
  - #if WRITES_VELOCITY_TO_GBUFFER
    - which calculates the velocity on a per-vertex basis by calculating the difference between its position last frame and this frame.
    -  Once calculated it stores it in the BasePassInterpolants variable,
    - but if you look over there they’ve wrapped the declaration of that variable in a matching
  - #if WRITES_VELOCITY_TO_GBUFFER .
      - This means that only shader variants that write velocity to the GBuffer will calculate it
      - This helps cut down on the amount of data passed between stages,
      - which means less bandwidth which in turn results in faster shaders.

# Material Graph to HLSL
- When we create a material graph inside of Unreal
  - Unreal translates your node network into HLSL code.
-  This code is inserted into the HLSL shaders by the compiler.
-  If we look at MaterialTemplate.ush it contains a number of structures (like FPixelMaterialInputs )
  - that have no body —instead they just have a %s .
- Unreal uses this as a string format and replaces it with the code specific to your material graph.
- This text replacement isn’t limited to just structures,
  - MaterialTemplate.ush also includes several functions which have no implementation.
-  For example, half GetMaterialCustomData0 , half3 Boilerplate Code that you don’t have to touch!
  - GetMaterialBaseColor , half3 GetMaterialNormal are all different functions that have their content filled out based on your material graph.
- This allows you to call these functions from the pixel shader and know that it will execute the calculations you have created in your
Material graph and will return you the resulting value for that pixel.
# The “Primitive” Variable
- Throughout the code you will find references to a variable named “Primitive”
  - —searching for it in the shader files yields no declaration though!
  -  This is because it’s actually declared on the C++ side through some macro magic.
  -  This macro declares a struct that is set by the renderer before each primitive is drawn on the GPU.
- The full list of variables that it supports can be found in PrimitiveUniformShaderParameters.h from the macro at the top.
- By default it includes things like LocalToWorld , WorldToLocal , ObjectWorldPositionAndRadius , LightingChannelMask , etc.
# Creating the GBuffer
- Deferred shading uses the concept of a “GBuffer” (Geometry Buffer)
  - which is a series of render targets that store different bits of information
    - about the geometry such as the world normal, base color,roughness, etc.
  -  Unreal samples these buffers when lighting is calculated to determine the final shading.
  - Before it gets there though, Unreal goes through a few steps to create and fill it.
- The exact contents of the GBuffer can differ/准确内容有可能不同
  - the number of channels and their uses can be shuffled around depending on your project settings.
  -  A common case example is a 5 texture GBuffer, A through E.
  - GBufferA.rgb = World Normal , with PerObjectGBufferData filling the alpha channel.
  - GBufferB.rgba = Metallic, Specular, Roughness, ShadingModelID .
  - GBufferC.rgb is the BaseColor with GBufferAO
    - Some functions have their contents filled in by C++
    - others have actual definitions filling the alpha channel.
  - GBufferD is dedicated to custom data and
  - GBufferE is for precomputed shadow factors.
- Inside BasePassPixelShader.usf the FPixelShaderInOut_MainPS function acts as the entry point for the pixel shader.
- This function looks quite complicated due to the numerous preprocessor defines
  - but is mostly filled with boilerplate code.
-  Unreal uses several different methods to calculate the required data for the GBuffer depending on
  - what lighting model and features you have enabled.
  - Unless you need to change some of this boilerplate code,
  - the first significant function is partway down where the shader gets the values
  - for BaseColor ,Metallic , Specular , MaterialAO , and Roughness .
- It does this by calling the functions declared in MaterialTemplate.ush
  - and their implementations are defined by your material graph.
- Now that we have sampled some of the data channels,
  - Unreal is going to modify some of them for certain shading models.
- For example, if you’re using a shading model which uses Subsurface Scattering
  - (Subsurface, Subsurface Profile, Preintegrated Skin, two sided foliageor cloth)
  - then Unreal will calculate a Subsurface color based on the call to GetMaterialSubsurfaceData .
  - If the lighting model is not one of these it uses the default value of zero.
  - The Subsurface Color values are now part of further calculations,
    - but unless you’re using a shading model that writes to the value it will simply be zero!
- After calculating Subsurface Color Unreal allows DBuffer Decals to
modify the results of the GBuffer if you have it enabled in your project.
- After doing some math Unreal applies the DBufferData
  - to the BaseColor, Metallic, Specular, Roughness, Normal and Subsurface Color channels.
- After allowing DBuffer Decals to modify the data Unreal
  - calculates the Opacity (using the result from your material graph)
  - and does some volumetric lightmap calculations.
- Finally it creates the FGBufferData The shader caches the result of the material graph calls to avoid executing their functions
multiple times.
- struct and it packs all of this data into it with each FGBufferData instance representing a single pixel.
# Setting the GBuffer Shading Model
The next thing on Unreal’s list is to let each shading model modify the
GBuffer as it sees fit. To accomplish this, Unreal has a function called
SetGBufferForShadingModel inside of ShadingModelMaterials.ush. This
function takes our Opacity, BaseColor, Metallic, Specular, Roughness
and Subsurface data and allows each shading model to assign the data
to the GBuffer struct however it would like.
Most shading models simply assign the incoming data without
modification but certain shading models (such as Subsurface related
ones) will encode additional data into the GBuffer using the custom
data channels. The other important thing this function does is it writes
the ShadingModelID to the GBuffer. This is an integer value stored perpixel
that lets the deferred pass look up what shading model each pixel
should use later.
It’s important to note here that if you want to use the CustomData
channels of the GBuffer you’ll need to modify BasePassCommon.ush
which has a preprocessor define for WRITES_CUSTOMDATA_TO_GBUFFER . If
you try to use the CustomData part of the GBuffer without making sure
your shading model is added here it will be discarded and you won’t
get any values later!
Using the Data
Now that we’ve let each lighting model choose how they’re going to
write their data into the FGBufferData struct, the BasePassPixelShader
is going to do a fair bit more boilerplate code and house keeping—
A single model that uses three different shading models—hair, eyes and skin.
calculating per pixel velocity, doing subsurface color changes,
overriding the roughness for ForceFullyRough, etc.
After this boilerplate code though Unreal will get precomputed
indirect lighting and skylight data
( GetPrecomputedIndirectLightingAndSkyLight ) and adds that to the
DiffuseColor for the GBuffer. There’s a fair bit of code related to
translucent forward shading, vertex fogging, and debugging, and we
eventually come down to the end of the FGBufferData struct. Unreal
calls EncodeGBuffer (DeferredShadingCommon.ush) which takes in
the FGBufferData struct and writes it out to the various GBuffer
textures, A-E.
That wraps up the end of the Base Pass Pixel Shader for the most part.
You’ll notice that there’s no mention of lighting or shadows in this
function. This is because in a deferred renderer these calculations are
deferred until later! We’ll look at that next.
Review
The BasePassPixelShader is responsible for sampling the various PBR
data channels by calling into functions generated by your Material
graph. This data is packed into a FGBufferData which is passed
around to various functions that modify the data based on various
shading models. The shading model determines the ShadingModelID
that is written to the texture for choosing which shading model to use
later. Finally the data in the FGBufferData is encoded into multiple
render targets for use later.
# Deferred Light Pixel Shader

# Base Pass Pixel Shader
-


# FMeshDrawingPolicy:
- 整合渲染模型过程，从绑定Shader到调用DrawCall,各个子类对应不同的独立着色器程序。
  - 1 初始化，根据需要生成或绑定各个对应的Shader.
　- 2 SetSharedState,设定和Mesh无关的Shader变量。
　- 3 SetMeshRenderState,设定和Mesh相关的Shader变量。
　- 4 DrawMesh 调用DrawCall.

# FUniformLightMapPolicy:



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
