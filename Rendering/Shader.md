## 着色器

#Vertex Shader
  - 当程序通过顶点着色器传入顶点缓冲数据时
    - GPU就会迭代顶点缓冲数据，并为每一个顶点执行一次着色器函数
    - 最主要的是用了进行坐标变换

# Domain Shader(域着色器)用于曲面细分
  - Domain Shader负责的最重要的功能就是通过贴图控制的方式，实现模型的形变

# Pixel Shader
  - 像素着色器通过输入的像素颜色值，然后计算颜色并且将其返回给绘图管线

# Hull Shader
  - 主要负责定义细分等级（LOD）和相关控制点在细分中的“形变”趋势



# UnOrderedAccessView
- D3D11中的Resource主要可以分为Buffers和Textures两类
    - Resource可以被绑定到渲染管线的特定阶段，有些绑定是直接的
      - （比如ID3D11DeviceContext::IASetVertexBuffers就直接把一定数量的VertexBuffers绑定到管线的Input Assembler阶段）
    - 也有的绑定必须是间接的
      - 即以Resource View为中间层进行绑定
      - 比如想让Pixel Shader读一个Texture中的内容
    - D3D11中对Resource View的使用只有四种情况，其他的Resource都可以直接绑定
      - 四种使用Resource View的情况如下：
        - 1.Render Target，即把内容渲染到的地方
          - 在初始化D3D11的时候一般都会由DXGISwapChain准备Backbuffers
          - 为其创建一个（或多个）Render Target View
          - 再用ID3D11DeviceContext::OMSetRenderTargets把这些Buffers绑定到管线的末端
          - 此外，在渲染到纹理的时候往往会设置别的Render Target
            - 对应的资源视图是RenderTargetView（RTV）
        - 2.Depth Stencil，也就是深度/模板缓存
          - 同样是在初始化D3D11的时候会用到
          - 对应的资源视图是DepthStencilView（DSV）
        - 3.Shader Resource，指会被着色器访问的一些外部资源，比如纹理（Textures）
          - 对应的资源视图是ShaderResourceView（SRV）
        - 4.Unordered Access，和Shader Resource相似
          - 但是它不光能被着色器读，还能被着色器写入，所以更加灵活一些
          - 不过它只能绑定于Pixel Shader或Compute Shader
          - 对应的资源视图是UnorderedAccessView（UAV）
# StructuredBuffer
    - 结构化缓冲区是一个包含相同大小元素的缓冲区
      - 使用具有一个或多个成员类型的结构来定义一个元素

# UniformBuffer/ConstantBuffer
# ShaderResourceView
# Frame Buffer Object (FBO) /RenderTarget


  引擎首先调用CompileGlobalShaderMap编译所有的Global Shader；对于Global Shader是判断整个Data Key（CompileGlobalShaderMap, ShaderCompiler.cpp）；如果GetDerivedDataCacheRef().GetSynchronous(* DataKey, CachedData)判断成功，会调用SerializeGlobalShaders序列化Global Shader Map；否则在接下来的VerifyGlobalShaders中会遍历所有的Shader Type编译所有的Global Shader；Shader编译是多线程进行的；编译完成后，会在FShaderCompilingManager::ProcessCompiledShaderMaps中调用SaveGlobalShaderMapToDerivedDataCache将Global Shader Map写回DDC。


  游戏运行时如果Global Shader没有编译，会直接报错。



  材质编译代码是FMaterial::BeginCompileShaderMap, 主要有两步:

   FHLSLMaterialTranslator将材质转成Shader代码.

   FMaterialShaderMap::Compile编译Shader代码.
