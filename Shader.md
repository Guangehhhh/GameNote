## 着色器

- Vertex Shader
  - 当程序通过顶点着色器传入顶点缓冲数据时
    - GPU就会迭代顶点缓冲数据，并为每一个顶点执行一次着色器函数
    - 最主要的是用了进行坐标变换

- Domain Shader(域着色器)用于曲面细分
  - Domain Shader负责的最重要的功能就是通过贴图控制的方式，实现模型的形变

-  Pixel Shader
  - 像素着色器通过输入的像素颜色值，然后计算颜色并且将其返回给绘图管线

- Hull Shader
  - 主要负责定义细分等级（LOD）和相关控制点在细分中的“形变”趋势




  引擎首先调用CompileGlobalShaderMap编译所有的Global Shader；对于Global Shader是判断整个Data Key（CompileGlobalShaderMap, ShaderCompiler.cpp）；如果GetDerivedDataCacheRef().GetSynchronous(* DataKey, CachedData)判断成功，会调用SerializeGlobalShaders序列化Global Shader Map；否则在接下来的VerifyGlobalShaders中会遍历所有的Shader Type编译所有的Global Shader；Shader编译是多线程进行的；编译完成后，会在FShaderCompilingManager::ProcessCompiledShaderMaps中调用SaveGlobalShaderMapToDerivedDataCache将Global Shader Map写回DDC。


  游戏运行时如果Global Shader没有编译，会直接报错。
