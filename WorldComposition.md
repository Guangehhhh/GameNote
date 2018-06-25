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

- 一个完整的加载逻辑是怎样的？？
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

- 相关成员都有哪些？

- SteamingState 代理
- LODPackageName



# 初步计划
- 读取时机

- 优化读取速度





# FScpeCycleCounterUObject 是什么？？
