## Level Streaming流关卡
- 每一个World里面至少有一个PersistentLevel以及0-N个StreamingLevels


- 控制流关卡的加载总体上来说有两种方式
	- 一种是通过关卡流体积控制（LevelStreamingVolume）
		- LevelStreamingVolume相当于一个定制的触发器，当玩家摄像机（注意是玩家摄像机）进入LevelStreamingVolume体积内的时候
		- 对应的流关卡就会加载（对应关系是通过下图操作设置的 点击levels旁边的按钮打开leveldetails窗口 inspectlevel找到需要加载的流关卡添加对应的LevelStreamingVolume）
	- 另一种是通过脚本（代码）逻辑控制，想怎么写就怎么写，简单的方式就是设一个触发器在玩家进入触发器的时候调用LoadStreamLevel

# World Composition
- 由于每个level关卡都可能非常大，我们在编辑器里面一点点调整流关卡里面的Actor位置实在是过于麻烦，所以UE提供了世界构成器功能
- 把N个关卡用拼图的形式拼接成一个大世界地图
- 需要在当前的WorldSetting里面勾选 Enable World Composition
- 世界构成器默认的加载逻辑就是当玩家距离要加载的关卡满足一定数值时就会加载对应的子关卡



streaming的话本身就是先加载普通资源（模型，shader），贴图默认在64x64以上分辨率的mip数据都是实时streaming进来的。


# WorldComposition 是什么？
- 运用LevelStreaming 实现开放世界的框架

- 相关成员都有哪些？
  - FDistanceVisibleLevel
  - TileList
  - TileStreamingList
      - FWorldCompositionTile 数组
        - 包含FWorldTileInfo
          - 位置，绝对位置 ，边界
          - ParentTilePackageName
          - LodList ,ZOrder


# WorldTileCollectionModel
- GenetateLODLevels

# MeshMergeUtilities
- CreateProxyMesh



# 一个完整的加载逻辑是怎样的？？
  -  最小操作单元从FWorldCompositionTile开始
  - 入口 函数
  - Level->Tick
    - WorldComposition->UpdateStreamingState
      - UWorld::ProcessLevelStreamingVolumes()
- 如果拓展 怎么拓展？？
# ULevelStreaming
- 加载逻辑是怎样的？？
- RequestLevel
  - Find Loaded Package
    - Find World
    - Set LevelPackage
  - DoesPackageExist
    - LoadPackageAsync
    - FlushAsyncLoading
			- TickAsyncLoading
				- ProcessLoadedPackages  
- 相关成员都有哪些？


默认情况下，异步加载将使用每帧5ms来勾选其加载循环。这由项目设置中的'异步加载时间限制'设置控制（您可以更改，也可以像Piotr建议的那样以编程方式更改）。你可以尝试减少它，看看是否有助于搭便车（尽管减少它意味着加载时间很可能会增加）。
您还可以尝试启用异步加载线程 - 这会将大部分异步加载工作卸载到单独的线程，并且可能会减少故障，尽管在'异步加载时间限制'指定的时间内仍会在游戏线程上调用PostLoad。请注意，您应该确保没有任何游戏类的构造函数/ PostInitProperties / Serialize函数是线程安全的，以便能够使用此功能而没有崩溃风险。
我认为Event Driven Loader或者还没有实现，或者在4.15中不够稳定，建议使用它。
我通常建议你升级引擎版本，因为自4.15以来，对EDL和ALT有很多修复


- SteamingState 代理
- LODPackageName
