
## WorldComposition
- GetDistanceVisibleLevels
  - 跳过NondisBase Level
  - 计算level Offset以及level Bound
  - 获取最小的Bound
  - 然后用level Bound 和 Location 计算SphereInterct
## 画面提升
- 多种植被
- 树木
- 小溪 河流 海 瀑布
- 雪
- 后处理

## 材质管理
- ShaderCatch
- ShaderPre
# SceneRenderTarget
# InitView
# InternalInitializeTextures
# DoOcclusionQueries
# FXSystem->PreRender
# InitViewsPossiblyAfterPrepass
# RenderPrePass
# ComputeLightGrid

# RenderOcclusion
# RenderShadowDepthMaps
# ClearLPVs
# ComputeVolumetricFog
# RenderIndirectCapsuleShadows
# GCompositionLighting.ProcessBeforeBasePass

# BeginRenderingGBuffer
# RenderBasePass

# BeginRenderingSceneColor

# FXSystem->PostRenderOpaque
# RenderVelocities
# RenderIndirectCapsuleShadows
# RenderLights
# RenderDeferredReflectionsAndSkyLighting
# RenderLightShaftOcclusion
# RenderAtmosphere
# RenderFog
# RenderTranslucency
# RenderDistortion
# RenderLightShaftBloom
# RenderOverlayExtensions
# RenderDistanceFieldLighting
# GPostProcessing.Process(RHICmdList, Views[ ViewIndex ], VelocityRT);
# RenderFinish
## 场景规范
- 先用Streaming Level分成两个
  - 主要建筑
  - 装饰建筑
- 再用HLOD烘培LOD等级
-
#Shader Permutations

# Use ShaderLib


# 参考其他项目的优化

#FParallelAnimationEvaluationTask
- 多线程调用ParallelAnimationEvaluation
  - EvaluateAnimation
    - InAnimInstance->ParallelEvaluateAnimation
  - EvaluatePostProcessMeshInstance
  - FinalizePoseEvaluationResult
  - FillComponentSpaceTransforms

## 特效添加
- 日夜变化
- 草地
- 体积雾 大气雾
- 体积云  

- 反射SSR
- 遮蔽

- 软阴影
- 植物的透射和次表面
- FFT海洋
大方开朗附件；阿列克斯的飞机；阿历克斯酱豆腐；绿卡就是的；910 5h1



##  RHI 优化

## InstanceMesh 的消耗 和渲染方式

## Streaming level 切分

## TextureGroup 分类 贴图设置

## 主管卡和副本地图切换

## 地形材质消耗


## UWorld了解一下
## AsyncLoading
## Package
## AssetManager
## Streaming Pool
## 写一个Scene Manager
# CullDistanceVolumes


# RHI resource memory (not tracked by our allocator)
  -   Render target memory 3D - STAT_RenderTargetMemory3D - STATGROUP_RHI - STATCAT_Advanced
  -   Render target memory Cube - STAT_RenderTargetMemoryCube - STATGROUP_RHI - STATCAT_Advanced
  -  Render target memory 2D - STAT_RenderTargetMemory2D - STATGROUP_RHI - STATCAT_Advanced
  -  Texture memory 3D - STAT_TextureMemory3D - STATGROUP_RHI - STATCAT_Advanced
  -  Texture memory 2D - STAT_TextureMemory2D - STATGROUP_RHI - STATCAT_Advanced
  -  Texture memory Cube - STAT_TextureMemoryCube - STATGROUP_RHI - STATCAT_Advanced
1738.276MB total

# FRHICommandListBase
# RHICommandList
- 内部实现了 各种AllocCommand函数
- 这些函数被 各个渲染阶段调用
- 例子：new (AllocCommand<FRHICommandSetRenderTargetsAndClear>()) FRHICommandSetRenderTargetsAndClear(RenderTargetsInfo);
- 这些Command 内部实现了  RHICommand接口的Execute 函数
  - 这个 RHICommand接口的ExecuteAndDestruct 里的Execute 函数
  - 这个Execute会通过宏的方式映射到平台的API 调用
- 最后在FRHICommandListExecutor 里调用
# FRHICommandListImmediate
# FRHIAsyncComputeCommandList

## FParallelCommandListSet
- 将每个单独的传递分解为块，而不是同时运行不同的传递作为长任务。我们在所有主要通行证中执行此操作
  - 例如BasePass，DepthPass，VelocityPass等
  - 例如FbasePassParallelCommandListSet
- 负责为每个线程创建RHICommandList，以正确的顺序提交结果，在部分命令列表的开头设置任何必要的状态等
- 负载平衡在这里很重要，以避免一些工作线程太少或太多与其他人相比，工作要做
  - UE4将自动尽力正确地进行负载平衡
  - 特殊提交命令被插入到RHICommandList中，以确保以正确的顺序进行GPU提交，并在提交之前完成翻译

# FRHICommandListExecutor
- 全局单粒
- 包含一个FRHICommandListImmediate 以及 FRHIAsyncComputeCommandListImmediate
- 在FRHICommandListImmediate::ImmediateFlush 调用ExecuteList
- FRHICommandListExecutor::GetImmediateCommandList().ImmediateFlush(EImmediateFlushType::FlushRHIThreadFlushResources);
  - 被手动调用
- 在FParallelCommandListSet 里通过Dispatch   可以调用到此处
- 然后Executor 具体调用 平台的Command


  - New一个Command


## FExecuteRHIThreadTask
## FDispatchRHIThreadTask

- FPrePassParallelCommandListSet

# FRHICommandSubmitSubList
# FParallelTranslateCommandList
# FParallelTranslateSetupCommandList
# FRHICommandRHIThreadFence

# FRHICommandWaitForAndSubmitSubList
# FRHICommandWaitForAndSubmitRTSubList
# FRHICommandWaitForAndSubmitSubListParallel

# IRHICommandContext
- 每个平台实现接口
# FGnmCommandListContext
- InitContextBuffers
- InitializeStateForFrameStart
- RHIDrawPrimitive
- SetSRVForStage
- SetTextureForStage
- SetCurrentRenderTarget
- SetCurrentDepthTarget
- PrepareForDispatch
- PrepareForDrawCall
- SetupVertexBuffers
- SetupVertexShaderMode




## TextureStreaming
UseFixedPoolSize=1		
FramesForFullUpdate=60		
UseNewMetrics=1			
MaxTempMemoryAllowed=100
HLODStrategy=2		
HiddenPrimitiveScale=0.0
MaxEffectiveScreenSize=0
Boost=0.0				
MipBias=30.0				
UsePerTextureBias=0		
FullyLoadUsedTextures=1		
DefragDynamicBounds=1		
LimitPoolSizeToVRAM=1

Texture Pool

纹理资源可用的总内存。这包含各种非流送资源，如渲染目标、GPU 粒子缓存、立方体贴图、UI 纹理和不可流送纹理。在部分平台上，此内存可用于保存静态网格体之类的非纹理资源。Texture Pool 约等于 Safety Pool + Temporary Pool + Streaming Pool + NonStreaming Mips（如有，仅限波动的量，上至安全池的大小）

Safety Pool

此值在 Engine 配置文件中设置（在 [TextureStreaming] 下，作为 MemoryMargin）。这是为意外（非流送）分配预留的内存。如可用的内存因低于此值的量形成周期波动，纹理流送器将在此波动下最大程度地稳定其流送池如正常（预计）的波动超过安全池大小，纹理流送器将不断应用其预算，可能会创建流入和流出纹理的无限循环。

Temporary Pool

此值由 r.Streaming.MaxTempMemoryAllowed 控制，并指定调整纹理大小时流送器可用的额外内存量。变更纹理的 mip 数量时，引擎需要新建一个纹理（无论大小），用于保存之后的 mip 数据。这能间接控制进行中请求的数量，因为流送器将向 IO 系统发送临时池允许的请求数量。

注意：临时池最小尺寸必须与需要流送的最大资源相同，但设为过大会浪费内存（因其正是为此目的而预留）。从另一方面而言，设为过小会减缓流送速度（无法为 IO 系统生成足够的工作，使其进入待机状态）。此外还需注意：流送器无法对进行中请求内的处理顺序进行较大程度的控制。这意味着使用相对较小的临时池可更大程度地控制加载顺序。

Streaming Pool

纹理流送器可用的内存量。流送器通常会将所有可用内存用于流送新 mip，或将之前流送的 mip 尽可能久地保存在内存中。流送池（Streaming Pool）包含可见 Mip（Visible Mip）、隐藏 Mip（Hidden Mip）、强制 Mip（Force Mip）和缓存 Mip（Cached Mip）。Streaming Pool 约等于 Visible Mips* + Hidden Mips + Forced Mips + Cached Mips（完全使用时为 * :，否则未使用的空间必须被占用）

NonStreaming Mips

非流送分配使用的内存量。如这些分配因超过安全池的值而出现定期波动，这将影响流送池的预算，应避免出现此状况（减少分配次数或增加安全池）。

Required Pool

纹理流送器需要根据其指标加载的 mip 数据量。这可超过纹理流送池的 100%，但同时也会进行一些妥协，部分纹理将不会以其所要求的分辨率加载。

Visible Mips

可见纹理 mip 当前占用的所需内存。这并不包含强制 mip。

Hidden Mips

非可见纹理 mip 当前占用的所需内存。这并不包含强制 mip。为防止首次显示纹理时出现低精度纹理，流送器会提前预流送纹理，但通常会比所需要的少一个 mip（详见 r.Streaming.HiddenPrimitiveScale ）。

Forced Mips

强制流入纹理当前占据的所需内存。纹理通常通过游戏性机制在一小段时间内强制流入。这并不包含被标记为不可流送的纹理。

Cached Mips

不再需要的纹理 mip 所占据的内存。它们将被缓存，除非其内存被其他所需的 mip 占据。

Wanted Pool

将逐渐被流入的所需池的部分。

Wanted Mips

实际流入的所需池大小。达到 100% 后，流送器将停止发送 IO 请求加载新 mip。Wanted Mips 等于 Visible Mips + Hidden Mips + Forced Mips

Inflight Requests

仍然被 IO 处理的内存量。为 0 时，所有之前的请求已在新建请求时处理。如流送器正在流送内容时出现此情况，则说明流送器未使用所有可用的带宽。增加 r.Streaming.MaxTempMemoryAllowed（代价是浪费的内存更多，加载顺序控制更少）或降低 r.Streaming.FramesForFullUpdate（代价是更新时间更长）可增加带宽。

IO Bandwidth

上次更新之后加载完成的 mip 大小，除以上次更新后的时间。用此法测算的 IO 带宽并不准确，但仍可用于确定系统加载请求的时间。

计数器

Setup Async Task

准备同步流送器任务数据的时间。它作为完整更新循环的第一步运行。

Update Streaming Data

这是用于增量更新流送数据的时间。这包括刷新可视性、更新纹理状态，更新已使用动态组件的边界。如 r.Streaming.FramesForFullUpdate 所定义，该步骤将连续运行数帧。

Streaming Texture

准备和发送加载与取消请求所需的时间。


- 术语“pool”代表概念（保留）内存，与实际使用的内存无关。术语“mips”代表纹理当前使用的内存
float MaxEffectiveScreenSize;
  - 计算需要的纹理分辨率时将限制流送器所考虑的屏幕尺寸
    - 这将防止高分辨率要求极大的流送池
	int32 MaxTempMemoryAllowed;
  - 临时池的大小
	int32 DropMips;
  - 此调试选项可避免将 mip 保存在内存中
	int32 HLODStrategy;
  - 控制层级 LOD 纹理的加载策略：
    - 0：允许所有 mip 的流送
    - 1：只允许最后一个（最大） mip 的流送。
    - 2：流送所有可流送的 mip。

	float HiddenPrimitiveScale;
  - 在组件引用不可见纹理（即其边界框被遮挡）时控制应用到所需分辨率的比例
    - 这只会在分辨率被最大可用分辨率锁定前对其产生影响，避免降低已受限纹理的质量
    - 换言之，它只会对拥有恰当视口分辨率的纹理产生影响

	int32 PoolSize;
  - 控制纹理池的大小。在非流送分配后，流送池使用纹理池中剩余的内容
  - 设为 0 时，内存约束将被流送器无视
  - 这还会移除临时的内存约束（限制进行中的请求数量），所有纹理将被无限保存在内存中
	bool bUseNewMetrics;
  - 为流入 mip 的逻辑使用最新修改，或继续使用之前的计算
    - 只有新测度较之于之前的测度有所改善时才可用于兼容性
	bool bFullyLoadUsedTextures;
  - 将所有已使用的纹理流送至其最大可用分辨率，并将其永久保存在内存中
    - 可用作完全禁用纹理流送的另一种方法，避免加载从不使用的纹理（占用更多内存）。
	bool bUseAllMips;
  - 是否移除纹理群组设置和动画设置中的所有分辨率限制
    - 展示美术效果或制作宣传材料时使用。
	bool bUsePerTextureBias;
  - 限制整体 mip 偏差对最大允许 mip 的效果，不限制其对视点所需 mip 的效果，并将其应用至纹理以便适应流送池
    - 每个纹理皆有其自身的 mip 偏差，从 0 到 MipBias（取决于预算计算）

	bool bUseMaterialData;
	int32 MinMipForSplitRequest;
  bool bLimitPoolSizeToVRAM;
  int32 GlobalMipBias;


#Texture
- 贴图最大的问题自然会表现在Streaming上。
  - 至于r.TextureStreaming，不可以随便关掉，因为会导致所有的mip全部被载入到内存中，即便根本就没有使用到。
  - 这个方面可以参考[官方文档]。
  - TextureStreaming上，Fortnite似乎将其载入过程从game thread转移到了async task中。
  - 贴图的大小在制作时也同样是尽可能的合并类似贴图以及尝试减小大的贴图的尺寸，针对不同的平台的特性，对法线贴图和Colormap进行优化。



## Level Streaming
使用关卡流可以很好的降低性能的消耗，为其他的特性预留空间。

不仅能减少内存使用还可以削减DrawCall。

关卡流在使用时，文件的读取由Worker Thread负责，从FArchive进行Deserialize的过程中新版本中也可以放到AsyncLoadingThread。只有最后的PostLoad过程被分散到了GameThread的每一帧中。

这种分散，在空中的时候由于Stream的更新较快为5ms，在地面上移动时，由于实际的Stream需求变低，为每一帧3ms。

这个是针对60fps设计的值，如果在30fps模式的话必须乘以二。

为了保证关卡流的流畅性，会一直尝试保持IO的进行。在本区域载入完成后，会对邻接的区域进行Streaming。

在4.20中还将会有一个用于在编辑器中进行InstancedMesh合并的工具，虽然Fortnite本身最终并没有使用这个方案。


- 从WorldTile 开始规划
- 一个Tile 周围模型数量 大场景数量 规则？
- 大场景 怎么细分成子场景？
- 场景归类 ，分为 Mesh ，Actor 需要逻辑，材质，


- 具体参考几个大作的实现方法 通过catch一帧

70397910284982


## 卡顿原因
- Level：：Serielize  //这是拷贝操作PIE 忽略
- AsyncLoad GT  游戏线程需要同步的太多
## 性能优化




目前性能表现：



## 关卡切换
- 使用场景过度

- UWorld::ServerTravel
- GameMode->ProcessServerTravel(FURL, bAbsolute);
- World->SeamlessTravel(World->NextURL, bAbsolute);
- SeamlessTravelHandler.StartTravel(this, NewURL, MapPackageGuid) && !SeamlessTravelHandler.IsInTransition()
  - StartLoadingDestination
- Tick 里  判断是否还在读取Package 如果是就忽略 如果读取完毕就开始卸载
- 直到 SeamlessTravelLoadCallback  读取结束   这个会影响Tick 流程
- SetHandlerLoadedData 保存当前读取的world和package  
- World->PersistentLevel->HandleLegacyMapBuildData();
- World->AsyncLoadAlwaysLoadedLevelsForSeamlessTravel();
  - Need to do this now so that data can be set correctly on the loaded world's collections.
	// This normally happens in InitWorld but that happens too late for seamless travel.
- 开始卸载Frommap
- CurrentWorld->FlushLevelStreaming(EFlushLevelStreamingType::Visibility);
- LoadedWorld->DuplicateRequestedLevels(LoadedWorld->GetOuter()->GetFName());
- CurrentWorld->GetGameState()->SeamlessTravelTransitionCheckpoint(!bSwitchedToDefaultMap);
- 	CurrentWorld->DestroyDemoNetDriver();
- CurrentWorld->CleanupWorld(bSwitchedToDefaultMap);
			CurrentWorld->RemoveFromRoot();
			CurrentWorld->ClearFlags(RF_Standalone);
-  	AudioDevice->Flush(CurrentWorld);
- LoadedWorld->AddToRoot();
- 	CurrentContext.SetCurrentWorld(LoadedWorld);
- CurrentWorld = nullptr;
- 	CollectGarbage( GARBAGE_COLLECTION_KEEPFLAGS, true );
- 	GEngine->VerifyLoadMapWorldCleanup();
- CurrentContext.LastURL = PendingTravelURL;

- FromMap  PendingURL 目的地
- 如果TransitionMap和 FromMap，和 目的地 相同
- bSwitchedToDefaultMap 设为true;
- FLoadTimeTracker::Get().DumpRawLoadTimes();
- GameMode->PostSeamlessTravel();		
- 清空引用LoadedPackage = nullptr;
		LoadedWorld = nullptr;


- 碰撞出发到Transitionmap的效果
- 刚开始到TransitionMap的效果在BeginPlay 触发
- Transitionmap 结束的效果
- LoadedWorld->BeginPlay();之后可以用来播放目的地地图效果


- GameMode （服务器） （实际上默认GameMode并不会传递到新场景）
拥有一个有效的 PlayerState 的所有控制器（服务器）（其实还包括PlayerState本身）
所有 PlayerControllers （服务器）需要保证你的GameMode继承自GameMode类而且两个地图的PlayerController类型相同才行
所有本地 PlayerControllers （服务器和客户端）
如果我们想额外的保存其他Actor有两个函数处理：

通过 AGameMode::GetSeamlessTravelActorList 额外添加的任何Actor（服务器）
通过 APlayerController::GetSeamlessTravelActorList （在本地PlayerControllers上调用）额外添加的任何Actor（非专有服务器与客户端）
（具体的保存了流程与细节参考函数FSeamlessTravelHandler::Tick）


- 1) Load destination map using LoadPackageAsync and store reference to destination world somewhere so GC will not remove it.

2) Initiate standard seamless travel

3) Seamless travel code should find destination package in the memory so switch to a destination world will be almost instant.





# Seamless travel的流程是怎样的？？？
-    seamlessly travels to the given URL by first loading the entry level in the background,
 * switching to it, and then loading the specified level. Does not disrupt network communication or disconnect clients.
 * You may need to implement GameMode::GetSeamlessTravelActorList(), PlayerController::GetSeamlessTravelActorList(),
 * GameMode::PostSeamlessTravel(), and/or GameMode::HandleSeamlessTravelPlayer() to handle preserving any information

## 问题
- 读取lagging
- 读取后 还需要加载levelStream


## 关卡package 预先读取

- Level_Crescent_CaveEntrance
Level_Crescent_FarAssets
Level_Crescent_FarCliff
Level_Crescent_Lighting_Overcast
Level_Crescent_MidCliff
Level_Crescent_Quarry_B


- Level_Crescent_FB_01_Area1
- Level_Crescent_FB_01_Area6
Level_Crescent_FB_01_Effect
Level_Crescent_FB_01_Collision
Level_Crescent_FB_01_Area7_ShuGi
Level_Crescent_FB_01_Area7
Level_Crescent_FB_01_Warehouse



# 问题列表
- World和Package的关系？？
- Seamless Travel 的时间是否是可控的？ 怎么控制 ？？
  - 加载时间碎片是否是可控的？？

- Loadmap 的优化方案 ？？




# 使用Hierarchical Instanced Static Mesh Component
- 消耗 严重的阶段
  - PrePass  
  - BeginOcclusionTest
  - ShadowDepth  
  - BasePass
  - Light
  - Transluceny
  - PostProcess
