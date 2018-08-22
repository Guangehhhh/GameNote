## SceneRendering
- FSceneViewFamily::BeginRender 初始化后传给渲染线程

# FDeferredShadingSceneRenderer 是主要的渲染类



# FScene
- 渲染场景，对渲染模块是私有的。
- 这是UWorld的渲染器版本，但是可以创建一个FScene用于在没有UWorld的编辑器中进行预览
- 存储任何视图或框架无关的渲染器状态，主要操作是添加和移除Primitive和灯光

# FSceneViewFamily

# FPrimitiveSceneInfo

# FMeshElementCollector

# FViewInfo
- 具有场景渲染使用的附加状态信息

# SceneRenderingAllocator

# FPrimitiveViewMasks

# FSceneViewState
