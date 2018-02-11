## 运动组件分析

# PerformMovement
  - 运动核心逻辑


  - FScopedMovementUpdate

  - 是否运动状态  处理None逻辑
  - 更新保存的LastPreAdditiveVelocity
    - 与自上次更新后发生的对Velocity的任何更改
  - 清理无效的RootMotionSource

  - ApplyAccumulatedForces 添加力的加速度 Force 和 Impluse
  - HandlePendingLaunch 处理发射速度替换速度
  - ClearAccumulatedForces 清空加速度
  - 使用加速度ApplyAccumulatedForces / HandlePendingLaunch后
    - 更新已保存的LastPreAdditiveVelocity

  - PrepareRootMotion逻辑
    - 准备根运动（从根运动源生成/积累以后使用）
  - 动画根运动覆盖Velocity，或者RootMotionSource覆盖它

  - 立即清除跳转输入，以允许移动事件触发下一次更新
  - StartNewPhysics 位移的核心逻辑

  - PhysicsRotation 没有RootMotion情况下更新Rotation
  - 运动完成后处理旋转

  - RootMotionParams清空

  - OnMovementUpdated触发
  - CallMovementUpdateDelegate触发

  - 如果存在有效的移动，则更新OldBaseLocation和OldBaseQuat
    - 并根据需要存储相对位置/旋转，忽略bDeferUpdateBasedMovement并强制更新
  - 更新组件的速度
  - 关于网络的巴拉巴拉
  - 保存LastUpdate相关的变量
