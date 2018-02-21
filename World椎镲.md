
# PreInit
- ProcessNewlyLoadedUObjects
  - UObjectProcessRegistrants
  - UObjectLoadAllCompiledInStructs
  - UObjectLoadAllCompiledInDefaultProperties
# Map_Load
- Map_Load
  - FindWorldInPackage
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
            -
          - AddActorsToScene_PhysX_AssumesLocked
          - FlushDeferredActors
      - Bodies.Reset();
	    - Transforms.Reset();
    - SendRenderDebugPhysics
    - CalculateMass
- ShouldCreatePhysicsState
  - GetStaticMesh
  - IsCollisionEnabled
- Broadcast



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
