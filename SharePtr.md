## SharePtr
- 继承std::enable_shared_from_this
  - 能让其一个对象被一个 std::shared_ptr 对象管理全地生成其他额外的 std::shared_ptr 实例
  - 可调用shared_from_this();
# 虚幻中的智能指针
- SharedPointerInternals::EnableSharedFromThis(this, InObject, InObject);
- SharedPointerInternals::FSharedReferencer< Mode > SharedReferenceCount;
- FReferenceControllerBase
- FReferenceControllerOps
