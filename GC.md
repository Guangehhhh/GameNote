# GC 垃圾回收
---
# 分类
- 引用计数GC
  - 引用计数式GC通过额外的计数来实时计算对单个对象的引用次数，当引用次数为0时回收对象
  - 引用计数的GC是实时的
  - 像微软COM对象的加减引用值以及C++中的智能指针都是通过引用计数来实现GC的
  - 引用计数算法是即时的，渐近的，对于交互式和实时系统比较友好，因为它几乎不会对用户程序造成明显的停顿
- 追踪式GC
  - 追踪式GC算法在达到GC条件时（强制GC或者内存不够用、到达GC间隔时间）通过扫描系统中是否有对象的引用来判断对象是否存活，然后回收无用对象
  - 追踪式GC算法通过递归的检查对象的可达性来判断对象是否存活，进而回收无用内存。
  - 追踪式的GC算法的关键在于准确并快速的找到所有可达对象，不可达的对象对于用户程序来说是不可见的，因此清扫阶段通常可以和用户程序并行执行
- 保守式GC
  - 保守式GC并不能准备识别每一个无用的对象（比如在32位程序中的一个4字节的值，它是不能判断出它是一个对象指针或者是一个数字的）
    - 但是能保证在不会错误的回收存活的对象的情况下回收一部分无用对象
    - 保守式GC不需要额外的数据来支持查找对象的引用，它将所有的内存数据假定为指针，通过一些条件来判定这个指针是否是一个合法的对象
- 精确式GC
  - 精确式GC是指在回收过程中能准确得识别和回收每一个无用对象的GC方式，为了准确识别每一个对象的引用，通过需要一些额外的数据（比如虚幻中的属性UProperty）
- 搬迁式非搬迁式
  - 搬迁式GC在GC过程中需要移动对象在内存中的位置
    - 当然移动对象位置后需要将所有引用到这个对象的地方更新到新位置（有的通过句柄来实现、而有的可能需要修改所有引用内存的指针）
  - 非搬迁式GC跟搬迁式GC正好相关，在GC过程中不需要移动对象的内存位置
- 实时和非实现GC
  - 实时GC是指不需要停止用户执行的GC方式。而非实时GC则需要停止用户程序的执行（stop the world）
- 渐进式和非渐进式GC
  - 和实时GC一样不需要中断用户程序运行，不同的地方在于渐进式GC不会在对象抛弃时立即回收占用的内存资源，而在GC达成一定条件时进行回收操作
  - 渐进式GC要解决的问题是如何缩短自动内存管理引起的中断，以降低对实时系统的影响
  - 渐进式GC算法基于分代式GC算法，它的核心在于在用户程序运行过程中维护年轻分代的根结点集合
  - 分类：追踪式，非实时，精确式，搬迁式，渐进式
---
# 标记清扫（Mark-Sweep）
- 标记清扫式GC算法是后面介绍的追踪式GC算法的基础，它通过搜索整个系统中对对象的引用来检查对象的可达性，以确定对象是否需要回收
- 追踪式，非实时，保守(非搬迁式)或者精确式(搬迁式) ，非渐进
- 标记阶段
  - 从根结点集合开始递归的标记所有可达对象。
  - 根结点集合通常包括所有的全局变量，静态变量以及栈区(注2)。这些数据可以被用户程序直接或者间接引用到
- 清扫阶段
  - 遍历所有对象，将没有标记为可达的对象回收，并清理标记位
  - 保守式的标记清扫算法:
    - 保守式的标记清扫算法缺少对象引用的内存信息（事实上它本身就为了这些Uncooperative Environment设计的）,它假定所有根结点集合为指针，递归的将这些指针指向的内存堆区标记为可达,并将所有可达区域的内存数据假定为指针，重复上一步，最终识别出不可达的内存区域，并将这些区域回收。
    - 保守式的GC算法可能导致内存泄漏。由于保守式GC算法没有必需的GC信息，因此必须假设所有内存数据是一个指针，这很可能将一个非指针数据当作指针，比如将一个整型值当作一个指针，并且这个值碰巧在已经分配的堆区域地址范围内，这将会导致这部分内存被标记为可达，进而不能被回收。
    - 保守式的GC不能确定一个内存上数据是否是一个指针，因此不能移动对象的位置
- 标记缩并
  - 有些情况下内存管理的性能瓶颈在分配阶段，内存碎片增加了查找可用内存区域的开销，标记缩并算法就是为了处理内存碎片问题而产生的。
  - 分类：追踪式，非实时，精确式，搬迁式，非渐进
- 节点复制
  - 节点复制GC通过将所有存活对象从一个区移动到另一个区来过滤非存活对象。
  - 分类：追踪式，非实时，精确式，搬迁式，非渐进


# 虚幻4 中的GC
- 是追踪式、非实时、精确式，非渐近、增量回收（时间片）
- 虚幻4中GC的入口是CollectGarbage()
  - 增量式的（bPerformFullPurge）,非实时的（gc 锁），最后调用了CollectGarbageInternal来执行真正的垃圾回收操作
- 标记阶段
  - 通过FRealtimeGC类中的PerformReachabilityAnalysis()函数来完成标记的
  - 它首先会调用MarkObjectsAsUnreachable()来把所有不带KeepFlags标记和EinternalObjectFlags::GarbageCollectionKeepFlags标记的对象全部标记为不可达
  - 并把它们添加到ObjectsToSerialize中去
  - 这个函数会判断当前的FUObjectItem::GetOwnerIdnex()是否为0，如果为0那么它就是一个普通物体也就意味着不在簇（cluster）中
  - 把所有符合条件的对象标记为不可达后，然后会调用下面的代码来进行可达性标记
- 解标记的过程
  - 比较重要的对象TFastReferenceCollector、FGCReferenceProcessor、以及FGCCollector
  - TFastReferenceCollector
    - CollectReferences用于可达性分析
      - 如果是单线程的话就调用ProcessObjectArray()遍历UObject的记号流（token stream ）来查找存在的引用
      - 否则会创建几个FCollectorTask来处理，最终调用的还是ProcessObjectArray()函数来处理。
        - ProcessObjectArray()会遍历InObjectsToSerializeArray中的UObject对象，然后根据这个类的UClass拿到它的FGCReferenceTokenStream
        - 如果是单线程且bAutoGenerateTokenSteram为true，且没有产生token stream,那么会调用AssembleReferenceTokenStream()来生成
        - 然后它会根据当前的ReferenceTokenSteramIndex来获取FGCReferenceInfo，然后根据它的类型来做相应的操作
        - 在这个处理的过程中，如果新加入的对象数据大于一定数量（MinDesiredObjectsPerSubTask）且是多线程处理
        - 那么就会按需创建一些新的TGraphTask<FCollectorTask>来并行处理引用问题，那么这就是一个递归的过程了
- 清扫阶段
  - 些标记了不可达标记的物体可以进行删除了，为了减少卡顿，虚幻4加入了增量清除的概念（IncrementalPurgeGarbage()函数）
    - 就是一次删除只占用固定的时间片，当然，如果是编译器状态或者是强制完全清除（比如下一次GC了，但是上一次增量清除还没有完成，那么就会强制清除）
  - IncrementalPurgeGarbage()
  - UObject带有EInternalObjectFlags::ClusterRoot且不可达，那么它会直接把上面的UObject（Cluster->Objects）符合条件的进行销毁，并且把当前簇删除掉-
- 反射信息用于GC
  - UClass::AssembleReferenceTokenStream()函数就是用生成记号流（token steam，其实就是记录了什么地方有UObject引用）
    - 它有一个CLASS_TokenStreamAssembled来保存只需要初始化一次     
    - 这里省去了一些代码，其它的大致的逻辑就是如果没有创建token stream或者要强制创建（需要清空ReferenceTokenSteam）
      - 那么就会遍历自身的所有属性，然后对每个UProperty调用EmitReferenceInfo()函数，它是一个虚函数，不同的UProperty会实现它
      - 如果它有父类（GetSuperClass()），那么就会调用父类的AssembleReferenceTokenStream()并把父类的添加到数组的前面
      - 同时会处理GCRT_EndofStream的特殊情况，最后加上GCRT_EndOfStream到记号流里面去，并设置CLASS_TokenStreamAssembled标记   
  - UObjectProperty::EmitReferenceInfo的实现

# ReferenceToken
- 在UObject体系中,每个类都有一个UClass实例用于描述该类的反射信息，使用UProperty来描述每个类成员变量
  - 但是在GC中如果直接遍历UProperty来进行扫描对象引用的话，效率会比较低(因为有许多非Object引用型的Property)
  - 所以ReferenceToken就运用而生了，ReferenceToken是一组token流，描述了类中的对象引用情况

# FGCReferenceProcessor
# FGCCollector
- 继承自FReferenceCollector，HandleObjectReference()和HandleObjectReferences()都调用了FGCReferenceProcessor的HandleObjectReference()方法来进行UObject的可达性分析
