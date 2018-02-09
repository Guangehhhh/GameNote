# FVertexFactory
- FRenderResource:
  - 定义接口
  - InitDynamicRHI / ReleaseDynamicRHI /InitRHI /ReleaseRHI/InitResource /ReleaseResource /UpdateRHI
  - 等渲染资源选择实现函数.
- 用来表示顶点数据格式，顶点分布结构，顶点元素Buffer，DeclarationElementList数组
- FVertexFactoryType:表示网格类型，如 Local/Particle(三种sprite/beamtrail,mesh)/Landscape/GPUSkin
- FShaderType: Global/Material/MeshMaterial (vertex/hull/demain/geomerty/pixel 一种)


- SetStreamSource

## FVertexStream


## FRHICommandBase
- 主要定义一个函数指针，一个执行方法调用函数指针指向的函数
- 函数指针：二个参数(FRHICommandListBase,FRHICommandBase)CallExecuteAndDestruct:传入自己FRHICommandBase到时函数指针指向的方向.
## FRHICommand
- FRHICommandBase的一个模板子类,模板需要定义Execute方法,其方法只需要FRHICommandListBase
  - 其会退化上面CallExecuteAndDestruct的FRHICommandBase参数,默认为自己.
