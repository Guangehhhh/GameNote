## SceneRendering
- FSceneViewFamily::BeginRender 初始化后传给渲染线程

# FDeferredShadingSceneRenderer 是主要的渲染类


# 渲染概述
- FBaseGraphTask
  - 每个事件基类
- TGraphTask: FBaseGraphTask
  - 事件模板

- FTaskThreadBase : public FRunnable, FSingleThreadRunnable
  - 执行Task的线程的基类
  - 但外部线程不使用它，因为这些线程是在别处创建的
  - TArray<FBaseGraphTask*> NewTasks;
    - 这个类里放的是事件流，以及相应处理事件流的一些方法
- FNameTaskThread: FTaskThreadBase
  - UE4里内置的用这个，如游戏线程，渲染线程

- FRunnable
  - 线程执行体
- FRenderingThread: FRunnable
  - 主要有方法Run调用执行渲染线程的事件流
- FRunnableThread
  - 可运行线程的接口, 此接口指定用于管理线程生命周期的方法
  - 包含一个FRunnable与相应的TLS实现
    - TLS搜了一下，简单来说，相同的变量，每个线程可以有不同的值

- FWorkerThread
  - FTaskThreadBase(事件流)与FRunnableThread(线程执行与TLS)的引用。封装相应对象FTaskThreadBase与FRunnableThread公开

- FTaskGraphInterface
  - 可以理解成一个单例管理类，管理所有FWorkerThread(线程与事件流),一般管理类的方法，根据类型得到对应的FWorkerThread等



# FTaskGraphInterface初始化相应的渲染线程所需的FNameTaskThread
- 以及调用StartRenderingThread创建渲染线程执行体FRunnable的子类FRenderingThread
  - 注意有个全局变量GIsThreadedRendering开始标为true

- FRenderingThread开始执行RenderingThreadMain
  - 找到渲染线程的FWorkerThread，初始化相应TLS的ID值
    - 如上所说，循环执行FTaskThreadBase::ProcessTasksUntilQuit里的事件

- 点程序退出等，ProcessTasksUntilQuit中断，相应渲染线程上的数据开始清理
































# FScene
- 渲染场景，对渲染模块是私有的。
- 这是UWorld的渲染器版本，但是可以创建一个FScene用于在没有UWorld的编辑器中进行预览
- 存储任何视图或框架无关的渲染器状态，主要操作是添加和移除Primitive和灯光

# FSceneViewFamily

# FPrimitiveSceneInfo

# FMeshElementCollector

# FViewInfo
- 具有场景渲染使用的附加状态信息
- View.PrimitiveVisibilityMap.Init(false,Scene->Primitives.Num());
-	View.DynamicMeshEndIndices.SetNumUninitialized(Scene->Primitives.Num());
-	View.PrimitiveDefinitelyUnoccludedMap.Init(false,Scene->Primitives.Num());
-	View.PotentiallyFadingPrimitiveMap.Init(false,Scene->Primitives.Num());
-	View.PrimitiveFadeUniformBuffers.AddZeroed(Scene->Primitives.Num());
-	View.StaticMeshVisibilityMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshOccluderMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshFadeOutDitheredLODMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshFadeInDitheredLODMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshVelocityMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshShadowDepthMap.Init(false,Scene->StaticMeshes.GetMaxIndex());
-	View.StaticMeshBatchVisibility.AddZeroed(Scene->StaticMeshBatchVisibility.GetMaxIndex());
- View.VisibleLightInfos.Empty(Scene->Lights.GetMaxIndex());
- View.StaticMeshEditorSelectionMap.Init(false, Scene->StaticMeshes.GetMaxIndex());
- View.DirtyPrecomputedLightingBufferPrimitives.Reserve(Scene->Primitives.Num());
# SceneRenderingAllocator

# FPrimitiveViewMasks

# FSceneViewState
