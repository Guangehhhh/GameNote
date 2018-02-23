
# PreInit
- ProcessNewlyLoadedUObjects
  - UObjectProcessRegistrants
  - UObjectLoadAllCompiledInStructs
  - UObjectLoadAllCompiledInDefaultProperties

# FSeamlessTravelHandler
- 封装无缝世界旅行的类
# FLevelStreamingGCHelper
- Helper结构封装功能用于将标记参与者及其组件标记为未决通过注册回调杀死垃圾收集之前

# FLevelViewportInfo
- 保存编辑器视口状态信息
# FStartPhysicsTickFunction,FEndPhysicsTickFunction
- 物理开始与结束Tick
# FStartAsyncSimulationFunction
- 布料模拟Tick
#
# Map_Load
- Map_Load
  - ExistingWorld = UWorld::FindWorldInPackage(ExistingPackage);
  - CurrentLevel->UpdateLevelComponents
    - IncrementalUpdateComponents
      - UpdateModelComponents
      - SortActorsHierarchy(Actors, this);

      - Actor->IncrementalRegisterComponents(NumComponentsToUpdate)
        - RegisterAllActorTickFunctions

        - RootComponent->RegisterComponentWithWorld
          - ExecuteRegisterEvents
            - OnRegister
            - CreateRenderState_Concurrent
            - CreatePhysicsState

          - InitializeComponent
          - BeginPlay
        - PostRegisterAllComponents


# World_Render
- UWorld::SendAllEndOfFrameUpdates
  - DoDeferredRenderUpdates_Concurrent
    - RecreateRenderState_Concurrent
    - SendRenderTransform_Concurrent
    - SendRenderDynamicData_Concurrent
  - GetMarkedForEndOfFrameUpdateState



# CreatePhysicsState
- GetPhysicsScene
- OnCreatePhysicsState
  -  UPrimitiveComponent::OnCreatePhysicsState
    - BodyInstance.IsValidBodyInstance()
    - GetBodySetup
    - BodyInstance.InitBody(BodySetup, BodyTransform, this, GetWorld()->GetPhysicsScene());
      - InitBodiesHelper.InitBodies();
        - InitBodies_PhysX
          - CreateShapesAndActors_PhysX
          - AddActorsToScene_PhysX_AssumesLocked
          - FlushDeferredActors
      - Bodies.Reset();
	    - Transforms.Reset();
    - SendRenderDebugPhysics
    - CalculateMass
- ShouldCreatePhysicsState
  - GetStaticMesh
  - IsCollisionEnabled



# SpawnActorPipLine
- PostSpawnInitialize
  - PostActorCreated
    - FinishSpawning
      - ExecuteConstruction
        - PostActorConstruction
          - PreInitializeComponents
            - InitializeComponents
              - PostInitializeComponents



# PostLoad
- Map_Load
  - LoadPackage
    - EndLoad
      - ConditionPostLoad
        - PostLoad

# PostInitProperties
- CreateDefaultObject
  - ~FObjectInitializer
    - PostConstructInit
      - PostInitProperties
- SpawnActor
  - NewObject
    - StaticConstructObject_Internal
      - ~FObjectInitializer
        - PostConstructInit
          - InitProperties
          - PostInitProperties
