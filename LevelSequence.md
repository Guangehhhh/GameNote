## LevelSequence
- MovieScene

- MovieScenePlayer
  - ResolveBoundObjects
    - 解析绑定到指定绑定标识的对象
      - InBindingId与要解析的对象有关的ID
      - OutObjects容器填充绑定的对象
- IMovieSceneObjectSpawner
  - IMovieScenePlayer接口

- FMovieSceneSpawnable
  - 描述了可以为这个MovieScene派生的一个对象
- MovieScenePossessable
  - 是一个“类型化的插槽”，用于允许MovieScene控制一个已经存在的对象

- FMovieSceneBinding
  - 绑定到运行时对象的一组Track
- FMovieSceneObjectBindingID
  - 在序列层次结构中的特定对象绑定的持久标识符
- FMovieSceneBindingOverrideData

- Tracks
  - 管理Section
  - UMovieSceneTrack接口
  - CreateTemplateForSection
  - CreateNewSection
  - GenerateTemplate
  - 维护一个Section列表

- Section
  - IKeyframeSection接口  
  - 编辑Section的数值和帧变化
  - 归属于Tracks
  - 拥有Template

- Template
  - FMovieSceneEvalTemplate接口
    - Initialize
    - Evaluate
    - Interrogate
    - 归属于Section
