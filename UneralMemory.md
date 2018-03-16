## UEMemory
- Ansi
- Stomp	默认的C分配器.
- TBB	Allocator to check for memory stomping.
- Jemalloc	Thread Building Blocks malloc.
- Binned	Linux/FreeBSD malloc.
- Binned2	Older binned malloc.
- Platform	Newer binned malloc.


-
# FUObjectItem
- UObject针对于内存相关的东西都储存在结构体FUObjectItem
- 有个全局变量GUObjectArray，可以通过以下代码来遍历所有的Objects：
for (FRawObjectIterator It(false); It; ++It)
｛
    FUObjectItem* ObjectItem = * It;
    UObject* obj = ObjectItem-&gt;Object;
｝
- 可以通过如下代码来获得一个UObject对应的ObjectItem
  - FUObjectItem* ObjectItem = GUObjectArray.ObjectToObjectItem(Object);
- GUObjectArray中有几个函数可以通过索引和ObjectItem、Object的相互获取
  - 对应的函数为IndexToObject、ObjectToIndex、IndexToObjectUnsafeForGC等
- GUObjectArray中所有的UObjectArray的分布是有顺序的——那些不会被GC的UObject
  - 例如各StaticClass、核心Object，GamePlayInstance等会放在前面；
  - 而那些有可能被GC的UObject，则放在后面。此外，GUObjectArray中有个变量ObjLastNonGCIndex，用于分隔这两类UObject

# FUObjectCluster
- Cluster指得是一组UObject为一簇，这群UObject同生共死，每个Cluster有一个根root的UObject
- 每一个FUObjectItem里面有一个变量ClusterRootIndex，这个储存的是当前Object所在的Cluster的root object所在GUObjectArray的索引
- 引擎中有一个全局变量FUObjectClusterContainer GUObjectClusters，用于管理内存中所有的Clusters
- 一个关卡ULevel不可以成为一个Cluster的root，原因是在这个时候（postload之后）仍然有很多被Level引用的assets并未构建它们自己的Cluster
- 虽然ULevel本身不可以作为Cluster Root，而相反的是它会创建一个特殊的actor container
  - 用来储存原本应该位于Cluster的actors
  - 这是由于只有某些特殊的actor种类才能用于Cluster，所以剩下的那些不能被cluster的actors需要通过actor container来进行引用
- ULevel中使用ClusterActors数组用于储存那些用于Cluster的Actors；
  - 用ActorsForGC储存那些剩下的Actors。主食中提到不希望Level直接去引用那些本就是Cluster的Actors，注释中提到这会导致变慢

- 如何获得一个UObject所引用的其他UObject？
- TArray<Obj*> CollectedReferences 
- FReferenceFinder ObjectReferenceCollector(CollectedReferences);
- ObjectReferenceCollector.FindReferences(Object)
