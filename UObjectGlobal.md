# UObjectGlobal
- FObjectDuplicationParameters
  - 该结构用于将参数值传递给StaticDuplicateObject（）方法
  - 只有构造函数参数是必需的, 所有其他成员都是可选的

- NewObject
- DuplicateObject
- UsesPerObjectConfig
- GetConfigFilename
- ContainsObjectOfClass

- StaticFindObject
- StaticLoadObject

- StaticConstructObject_Internal
  - 创建一个对象的新实例
    - 返回的对象将被完全初始化。
      - 如果InFlags包含RF_NeedsLoad（指示对象仍然需要从磁盘加载其对象数据）
      - 则组件不是实例化的（这将发生在PostLoad（））中
    - StaticConstructObject和StaticAllocateObject的区别在于
      - StaticConstructObject也会调用对象的类构造函数
  - 这里面主要做两件事：
    - 一是用StaticAllocateObject来分配空间
      - 首先是检查要创建新对象，还是替换一个原有对象，因为一路传进来的有个Name参数
        - 如果找到一个已有的同名对象，那么会强制那个对象析构，然后重用它的空间，否则的话，就会直接分配空间了
      - 创建对象的新实例或替换现有对象
        - 如果同时指定了“Outer”和“Name”并且存在具有相同“Class”，“Outer”和“Name”的对象
        - 则现有的对象将被破坏，新的对​​象将被创建
    - 二是在此空间上初始化对象
      -  (* InClass->ClassConstructor)( FObjectInitializer(Result, InTemplate, bCopyTransientsFromClassDefaults, true, InInstanceGraph) );
      - ClassConstructor是每个UClass都有的一个函数指针类成员变量，实际上所有的UClass里该指针都指向一个全局模板函数
      - 每个类里的__DefaultConstructor也是用宏统一生成的，内容也是简单的转发给new
        - static void __ DefaultConstructor(const FObjectInitializer& X) { new((EInternal* )X.GetObj())TClass(X); }
        - 这个new的形式很不平凡，既有传给operator new的【(EInternal*)X.GetObj()】，又有传给该类实际构造函数的【FObjectInitializer& X】
          - 后者正是InClass->ClassConstructor需要的实参，而前者也是通过宏（DECLARE_CLASS）定义在每个UObject里的一个operator new
            - StaticAllocateObject分配了内存空间Result，然后以此为参数（当然还有其它的参数）构造了一个FObjectInitializer X，其中X.GetObj返回的就是这个内存地址Result；
            - 接着用【 new(Result) TClass(X) 】来构造对象，指明地址在Result上，并将X做为参数传给其构造函数
---
- StaticAllocateObject

---
- StaticDuplicateObject
  - 使用指定的Outer和Name创建SourceObject的副本，以及SourceObject包含的所有对象的副本
  - 由SourceOuter或RootObject引用并由SourceOuter包含的任何对象也被复制，并保持其名称相对于SourceOuter
  - 任何对重复对象的引用会自动替换为对象的副本

---
- StaticExec,StaticTick
  - 遍历所有被认为是根的一部分的对象来设置GC优化
- LoadPackage
  - 加载一个包和所有与上下文标志匹配的包含对象
- LoadPackageAsync
  - 异步加载包和所有与上下文标志匹配的包含对象 非阻塞
- CancelAsyncLoading
- GetAsyncLoadPercentage
  - 返回名称中传入的包中的异步装载百分比，如果没有，则返回-1
  - 这是缓慢的。可以阻止异步加载

- CollectGarbage
  - 删除所有未被引用的对象，保留设置了KeepFlags的任何对象。将等待其他线程解锁GC
    - KeepFlags对象与这些标志将被保留，不管被引用与否
    - bPerformFullPurge如果为true，则在标记传递后执行完全清除
- TryCollectGarbage
  - 只有在没有其他线程在GC上持有锁的情况下才执行垃圾收集
- IncrementalPurgeGarbage
  - 增量删除
- MakeUniqueObjectName
  - 通过组合基本名称和任意数字字符串来创建一个唯一的名称，返回的对象名称保证不存在
- MakeObjectNameFromDisplayLabel
  - 给定一个显示标签字符串，生成一个FName slug，该标签是一个有效的FName
  - 如果对象的当前名称已经满意，则返回该名称。
  - 例如，“[MyObject]：Object Label”变为“MyObjectObjectLabel”FName slug
- IsReferenced
  - 会影响一个对象是否被引用，不计数
- FlushAsyncLoading
  - 阻塞直到所有读取的包/到完成
- GetNumAsyncPackages
  - 返回异步加载的数量
- ProcessAsyncLoading
  - 用软时间限制序列化每帧的一些数据。该功能被设计为能够在给定足够时间的情况下一次完全加载一个包装
- ProcessAsyncLoadingUntilComplete
  - 阻止并运行ProcessAsyncLoading，直到达到时间限制，完成谓词返回true，或者完成所有的异步加载
- FindPackage
  - 按名称查找现有的软件包
    - InOuter要在里面搜索的外部对象
    - PackageName要查找的包的名称
- CreatePackage
  - 按名称查找现有软件包，如果不存在，则创建它
    - InOuter要在里面搜索的外部对象
    - 现有的软件包或新创建的软件包
- SaveToTransactionBuffer
---

# FObjectInitializer
- 在构造函数返回后，其临时参数【FObjectInitializer& X】是要自动析构的
  - 大部份的事都是与该体系内部状态管理有关的，暂时难以透彻理解。
- 但有一个的操作与应用相关，即属性初始化，这是通过InitProperties来完成的
  - 通过遍历UClass上记录的属性元数据，可以对当前实例的每个属性进行赋值。
    - 有趣的是DefaultData这个参数，也就是最早的Template参数，一路辗转到此。
      - 当然如果Template为空的话，这里的DefaultData就是该类的CDO了。
    - 代码里明显体现出对此两种情况的不同处理策略：
      - 如果是指定了要复制的对象Template->DefaultData，那么要遍历类上的所有的属性
        - 因为对于一个实际的复制目标，你不知道它哪些属性已经改变了（不是默认值 ），因此必须全盘复制
      - 如果只是从CDO复制的话，那么只需要处理该类里明确指定过可能有初始化状态的字段
        - 如打上CPF_Config标记的字段，它们在启动时会去ini文件里提取相应的配置值

---

- 内部类在调用真正的C++构造函数后 最终实现UObject创建（初始化属性）
- GetArchetype
  - 返回此对象稍后将复制属性的原型
- GetClass
- CreateDefaultSubobject
- CreateOptionalDefaultSubobject
- CreateAbstractDefaultSubobject
- SetDefaultSubobjectClass
- DoNotCreateDefaultSubobject
- IslegalOverride
- AssertIfInConstructor
- Get()
  - 获取当前构造的对象的ObjectInitializer。只能在UObject派生类的构造函数中使用
- InitProperties
  - 二进制初始化对象属性为零或默认值
- InitSubobjectProperties
  - 为通过此ObjectInitializer创建的任何默认子对象调用InitProperties
- InstanceSubobjects
- InitNonNativeProperty
- PostConstructInit






# FReferenceCollector
# FReferenceFinder



















- ParseObject
  - 从文本表示中解析对象
