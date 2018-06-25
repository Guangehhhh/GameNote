## FrameGraph
- Setup状态
  - 所有资源都是虚拟的渲染过程的输入和输出
  - 为渲染资源（例如缓冲区，纹理）创建描述 并将其声明为framegraph输入和输出资源
  - 使用虚拟资源句柄
  - 例如 using buffer_resource     = fg::resource<buffer_description , gl::buffer    >;
  - Passes接口实现对资源的 读写创建 操作
- Compile状态
- 为每个声明的资源专门设置fg :: realize <description_type，actual_type>
  - 接受资源描述并返回实际资源

- Execute状态
  - 创建一个框架图并将渲染任务/保留的资源添加到它
- 一旦添加了所有渲染任务和资源，调用framegraph.compile（）
  - 然后，在每次更新中使用framegraph.execute（）



  Create aliases of	resources	that will	be created by the	future render passes
