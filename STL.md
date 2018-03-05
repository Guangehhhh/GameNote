
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

- sizeof  
- 32bit 64bit
- size_t
- cast
- assert
- virtual functions
- constexpr
- dealtype
- share_ptr

- memset
    - void* dest, int ch, std::size_t count
    - 转换值 ch 为 unsigned char 并复制它到 dest 所指向对象的首 count 个字节。若该对象非可平凡复制（如标量、数组或 C 兼容的结构体），则行为未定义。若 count 大于 dest 所指向的对象大小，则行为未定义
- memcpy
    -  void* dest, const void* src, std::size_t count
    - 从 src 所指向的对象复制 count 个字符到 dest 所指向的对象。两个对象都被转译成 unsigned char 的数组。
    - 若对象重叠，则行为未定义若  dest 或 src 为空指针则行为未定义，纵使 count 为零。
- memmove
    -  void* dest, const void* src, std::size_t count
    - 从 src 所指向的对象复制 count 个字节到 dest 所指向的对象。两个对象都被转译成 unsigned char 的数组
    - 如同复制字符到临时数组，再从该数组到 dest 一般发生复制。

- strlen
- strstr
- strchr
- strcpy
- strncpy

- new operator  ，operator new ，placement new
  - operator new
    - void *  p;
    - while ((p = malloc(size)) == 0)
    - if (callnewh(size) == 0)
      - THROWNCEE(XSTD bad_alloc, );
    -  return (p);
- new -> operator new -> malloc
- malloc出错 -> 调用new_handler -> 若new_handler返回为0 -> 抛出异常
　　　　　　　　　　　　 |
　　　　　　　　　　　　 v
　　　　　　　　　若new_handler返回非0 -> 继续调用malloc
- operator new[]根据所需数目调用operator new

- void operator delete( void * p ){
    RTCCALLBACK(RTC_Free_hook, (p, 0));
    free( p );
}
- delete -> 析构函数(如果有) -> operator delete -> RTCCALLBACK空宏定义 -> free
- 垃圾回收
    - 引用计数
    - Mark&Sweep
    - 节点复制

- 当应用程序使用malloc试图从堆上获得内存块时，通常都是以常规方式来调用malloc，而当malloc找不到合适空闲块的时候，它就会去调用垃圾收集器，以回收垃圾到空闲链表。此时，垃圾收集器将识别出垃圾块，并通过free函数将它们返回给堆。这样看来，垃圾收集器代替我们调用了free函数，从而让我们显式分配，而无须显式释放

- 内存管理和资源缓存
    - C++中，内存区分为5个区，分别是堆、栈、全局/静态存储区、常量存储区
    -
- 熟悉当前的引擎架构；熟悉动画粒子物理等模块的核心原理和工作方式
 - strcpy strlen
 - 在标准c++中，int的定义长度要依靠你的机器的字长，也就是说，如果你的机器是32位的，int的长度为32位，如果你的机器是64位的，那么int的标准长度就是64位

- FILE* fopen(const char*filename,const*char* mode)
- FILE* fopen(const char*filename,const*char* mode)
- fprintf() fscanf()

 - static 和const
 - 指针和引用

 - 结构体struct：把不同类型的数据组合成一个整体，自定义类型。
 - 共同体union：使几个不同类型的变量共同占用一段内存。

 static_cast：任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast。
 const_cast：去const属性，只能改变运算对象的底层const。常用于有函数重载的上下文中。
 reninterpret_cast：通常为运算对象的位模式提供较低层次的重新解释，本质依赖与机器。
 dynamic_cast：主要用来执行“安全向下转型”，也就是用来决定某对象是否归属继承体系中的某个类型主要用于多态类之间的转换


- HoelloWorld在编译运行过程中可以分为4个步骤，预处理、编译、汇编和链接。
- 编译的过程可分为6步：词法分析、语法分析、语义分析、源代码优化、代码生成和目标代码优化。

- 如果类没有定义析构函数，那么只有在类中含有成员对象或者是基类中含有析构函数的情况下，编译器才会自动合成一个出来
- 静态成员函数 static member functions
- 不能访问非静态成员
- 不能声明为const、volatile或virtual
- 参数没有this
- 可以不用对象访问，直接 类名::静态成员函数 访问
- 每一个nonstatic data member的偏移量在编译时即可获知，不管其有多么复杂的派生，都是一样



- 代码段(.text)：
    - 用来存放可执行文件的机器指令。存放在只读区域，以防止被修改
- 只读数据段(.rodata)：
    - 用来存放常量存放在只读区域，如字符串常量、全局const变量等
- 可读写数据段(.data)：
    - 用来存放可执行文件中已初始化全局变量，即静态分配的变量和全局变量
- BSS段(.bss)：
    - 未初始化的全局变量和局部静态变量一般放在.bss的段里，以节省内存空间
- 堆：用来容纳应用程序动态分配的内存区域
    - 当程序使用malloc或new分配内存时，得到的内存来自堆。堆通常位于栈的下方。
- 栈：用于维护函数调用的上下文。栈通常分配在用户空间的最高地址处分配。
- 动态链接库映射区：如果程序调用了动态链接库，则会有这一部分
    - 该区域是用于映射装载的动态链接库。
- 保留区：内存中受到保护而禁止访问的内存区域。


- 运行期多态
    - 虚函数
- 编译期多态
    - 以不同的模板参数具现化导致调用不同的函数
- 不要在构造函数和析构函数中调用虚函数

- ESP：栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈顶
 - EBP：基址指针寄存器(extended base pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的底部
- 函数栈帧：ESP和EBP之间的内存空间为当前栈帧，EBP标识了当前栈帧的底部，ESP标识了当前栈帧的顶部。
- EIP：指令寄存器(extended instruction pointer)， 其内存放着一个指针，该指针永远指向下一条待执行的指令地址
- 参数入栈：将参数从右向左依次压入系统栈中返回地址入栈：
    - 将当前代码区调用指令的下一条指令地址压入栈中，供函数返回时继续执行代码区跳转：
    - 处理器从当前代码区跳转到被调用函数的入口处栈帧调整：
    - 具体包括保存当前栈帧状态值，已备后面恢复本栈帧时使用（EBP入栈）将当前栈帧切换到新栈帧。
    - （将ESP值装入EBP，更新栈帧底部）给新栈帧分配空间。（把ESP减去所需空间的大小，抬高栈顶）
