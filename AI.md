# AI
- UseBlackboard
  - 检查黑板资源是否为空
  - 寻找黑板组件
  - 初始化黑板组件
    - 调用InitializeBlackboard
      - 从BlackboardComponent调用InitializeBlackboard(BlackboardAsset)
      - SetKey
      - 调用OnUsingBlackBoard(&Compo, &BlackboardAsset)
      - 返回bool
  - 返回bool

- RunBehaviorTree
  - 判断BTAsset是否为空
  - 获取黑板组件
  - 检查并且初始化BTAsset的 黑板资源
  - Cast<BehaviorTreeComponent>(BrainComponent)
  - 如果Cast失败 就New并初始化
  - BTComponent->StartTree(* BTAsset， EBTExecutionMode::Looped)

- BehaviorTreeComponent是BrainComponent的子类


## AIPerception

## EQS

## AIGraphEditor
- Graph 包含 UEdGraphNode,UEdGraphSchema

## TAttribute

## FSequencer
- TRange
- FSequencerNodeTree
- UMovieSceneSequence
  - UMovieScene
- UMovieSceneSection
- UMovieScene3DAttachTrack
- UMovieSceneTrack
- FSequencerTrackNode
- FSequencerDisplayNode

- ISequenceRecorder
- ACineCameraActor

- FScopedTransaction

- FMovieSceneClipboard

- FSequencerSelectedKey

- FMovieScenePossessable
- UAutomatedLevelSequenceCapture
  - MovieSceneCapture->SetLevelSequenceAsset(GetCurrentAsset()->GetPathName());
- MovieSceneToolHelpers

- FMovieSceneSpawnable
- FMovieSceneSpawnRegister
- ISequencerObjectChangeListener
- ISequencerKeyCollection
- FSequencerTemplateStore
- FSequencerTimingManager
## Shader

## DebugDraw

## 矩阵
