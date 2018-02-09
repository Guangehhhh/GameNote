# PawnAction
- 流程控制
- Task的 PushAction
  - 获取AIController
  - 设置Action观察者
    - Action.SetActionObserver
  - PerformAction
    - 内部调用PawnActionComponent的PushAction
      - 设定OwnerComponent 和 Instigator
      - 调用OnEvent
        - OnEvent属于PawnActionComponent
        - 添加到内部的ActionEvents数组里
        - 如果数组为不为空，SetComponentTickEnable

  - 设置Task节点为Progress


# PawnActionComponent
- 维护一个自己的ActionArray和ActionStack

- 每个Action拥有自己的权重
- UpdateCurrentAction
- 在TickComponent里的操作
  - Sort
  - for循环遍历
    - 通过EventType Switch
    - 更新每个状态
      - 来决定Pop 和Push
    - Reset
    - UpdateCurrentAction
    - CurrentAction调用TickAction
    - 判断Action数组是否为空
      - 为空 SetComponentTickEnabled 为false

- OnActionEvent
  - 内部调用ActionEventHandler
  - 返回EPawnActionTaskResult
    - 如果为ActionLost
    - 调用OnActionLost函数
- ActionEventHandler
  - 获取AIController
  - 获取行为树组件
  - 行为树组建调用GetTaskStatus
  -
