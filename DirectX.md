## DirectX
---

# 动态缓存
- 要在同一副精灵纹理中创建一套精灵动画，我们需要经常更改缓存中的顶点位置信息
  - 动态缓存对于我们需要修改一块缓存中的内容的这种情况来说是很合适的
  - 不推荐多次创建和销毁静态缓存块，特别是逐帧这样做，你应该使用动态缓存来做这样的任务
- D3D11_BUFFER_DESC vertexDesc
  - vertexDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;


# 
