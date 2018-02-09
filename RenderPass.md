## FSceneView
- 从场景空间投影到2D屏幕区域
- FViewUniformShaderParameters
  - 视图参数的统一缓冲区 这只会在渲染线程的FSceneView副本中初始化




## UPrimitiveComponent
- MoveComponentImpl  
  - 移动相关 ，比较重要
- Overlap相关函数比较多

## UModelComponent
- 

## SkeleltalMesh
- 查看SekeletalMeshType
- FSkeletalMeshSceneProxy
- FSkeletalMeshObject
  -


- FSoftSkinVertex

- FSkeletalMeshResource

- FSkeletalMeshVertexBuffer
  - FSkeletalMeshVertexDataInterface
  - InitRHI
    - VertexData->GetResourceArray
    - RHICreateVertexBuffer
    - RHICreateShaderResourceView

- USkeleton
  - 网格和动画之间的链接
  - FBoneNode
  - FVirtualBone
