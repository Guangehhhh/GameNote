## 一些UDN的问题

- 错误的引用一般肯定是你们自己的逻辑代码引起的
  - 譬如你地图里spawn了个actor，把actor的引用在了一个静态或者gameinstance之类，再或者Asset之类的对象身上了
  - 那么换地图要把整个world都gc掉的时候如果你不先手动清除掉这个引用，就肯定会crash
  - 因为这个actor反向引用了level，再反向引用到world，以致于之前那关地图完全都不能被释放掉



- UBlueprint对象比较特殊，事实上，在你的ContentBrowser里列出来的对象一般都是Assets
  - （例如UBlueprint，UStaticMesh，USkeletalMesh，UMaterial等）
  - 这些对象在拖到Viewport中的时候会有一个特殊的行为转成自己类型对应的Actor
    - 其中UBlueprint对象的实际Actor的UClass来自于这个UBlueprint的GeneratedClass属性
    - 所以你们其实应该SpawnActor用的Class是Load出来后的UBlueprint::GeneratedClass。


再然后，callback的delegate引擎定义成了不带参数的回调（DECLARE_DELEGATE(FStreamableDelegate);），你们需要带什么样的参数呢？事实上异步load完成，你除了会关心是哪个asset load完了，也没有其他数据或者参数是有意义的吧？那你也许可以把这个发出异步请求的对象单独封装起来（就是一个请求有一个独立的子对象），然后回调的Delegate做成这个子对象的方法，这样这个子对象可以在发出异步请求的时候保存请求的Asset为自己的成员变量，留待回调用查询使用。


顺便一提，为了提升异步加载的效率，可以尝试打开s.AsyncLoadingThreadEnabled的CVar，默认是关闭使用游戏线程分时间片的方式做异步加载的，对于加载时间和游戏线程都会有一些损耗


- 你可以在项目的Target.cs中的SetupGlobalEnvironment中加上BuildConfiguration.bUseMallocProfiler = true;
  - 这时候引擎的GMalloc会替换成FMallocProfilerEx（如果是Editor版本的话，是FMallocProfiler），记录下所有的分配释放操作
  - 可以用MemoryProfiler2工具查看对应的结果

- RVO是直接使用RVO算法在movement中调整速度来避免互相碰撞，没有什么设置，只要在movementcomponent中打开就可以了
- DetourCrwod更高级一点，有一些参数可以调整，并且是直接在navmesh层面完成的，所以会比RVO更准确但复杂，可以避免群体超出navmesh的行为



- 主线程调用GC的时候，会把逻辑线程中没有被rootset引用到的PrimitiveComponent都打上Destroy的标记但并不会实际的销毁和释放内存
  - 并且会标记一个FRenderCommandFence::BeginFence，知道实际对应的sceneproxy下的renderresource使用完后
  - 主线程query到这个fence pass掉以后才会去实际释放资源
  - 你们看到的这个错误比较像是你们自己写的类似PrimitiveComponent没有在Destroy的时候用Fence的机制来安全的等待渲染线程释放完资源才实际释放
  - 你们检查一下看看有没有自己写的PrimitiveComponent类型导致的这个问题？


- 引擎的帧平滑，其实就是对一段时间内的DeltaTime做取平均值
  - 具体的实现可以看UpdateTimeAndHandleMaxTickRate和UpdateRunningAverageDeltaTime函数


- Stat的中的很多数值，其实也是类似的做法，也是取一定数量的平均值，包括DC，所以你会看到DC数量是浮点，并且变化是一个过程
