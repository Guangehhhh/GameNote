
# std::any_cast
# std::optional<>
# std::any
- any 描述用于任何类型的单个值的类型安全容器
- any 的对象存储任何满足构造函数要求的类型的一个实例或为空
  - 而这被称为 any 类对象的状态,存储的实例被称作所含对象
  - 若两个状态均为空，或均为非空且其所含对象等价，则两个状态等价。
- 非成员 any_cast 函数提供对所含对象的类型安全访问

# std::forward
    - std::move和std::forward在运行期都没有做任何事情
    - td::forward只有在它的参数绑定到一个右值上的时候，它才转换它的参数到一个右值
# std::bind
    - bind的思想实际上是一种延迟计算的思想，将可调用对象保存起来，然后在需要的时候再调用
    - bind()接受一个函数（或者函数对象，或者任何你可以通过”(…)”符号调用的事物）
      - 生成一个其有某一个或多个函数参数被“绑定”或重新组织的函数对象
# std::copy
    - copy只负责复制，不负责申请空间，所以复制前必须有足够的空间


- std::remove_reference
    -
# std::get
- std::integral_constant
    - 包装特定类型的静态常量。它是 C++ 类型特性的基类
- std::thread
- std::this_thread
- std::chrono
- any
    - 任何类型的单个值的类型安全容器
    -
- type_info
    - 保有一个类型的实现指定信息，包括类型的名称和比较二个类型相等的方法或相对顺序
