# NPC架构
- Net-Server-Client-Monster
- Player和NPC分开
- 怪物和NPC耦合
- Client里包含AITree
- 具体AI逻辑在Controller
- Cotroller架构
  - NetNPCController
    - NPCControlelr
- 在ServerNPCCharacter执行具体初始化和参数




- 攻击是改变AnimState 为ESkill
- 如果有三个技能分别给Parm传123来选择技能

- GetDateManager
  - GetFunctionLibrary
    - GetNPCBase(NPCID)
    - GetNPCAIBase(AIScript)

- NPC 属性
  - 管理属性逻辑


- NPC 方法
  - 交互逻辑（选择）
    - 对话逻辑（选择）
    - 任务逻辑(选择)
  - 攻击逻辑
    - 普攻
    - 使用技能
    - 受到伤害逻辑与伤害逻辑
    - 目标管理逻辑


  - 移动
    - 巡逻Task
    - 逃跑Task

  - 动画相关逻辑
  - 碰撞相关逻辑


# Player架构
- 初始化
- 场景模式类别
- 属性
  - 属性管理

- 交互逻辑
- UI逻辑
- 任务逻辑
- 对话逻辑

- 移动
- 攻击逻辑
  - 普攻
  - 使用技能
  - 伤害系统

- 装备系统逻辑
- 背包管理逻辑
