## FArchive
- 可用于加载，保存和垃圾回收的基类
- 操作符可以通过想要序列化FName实例的子类来实现
  - 例如FText::SerializeText(* this, Value);

- virtual void SerializeBits(void* V, int64 LengthBits)
- CustomVer
  - 从存档中查询定制版本。如果正在使用档案写入，那么自定义版本必须已经被注册

# UAsset
- UE中使用统一的格式存储资源(uasset, umap)
- 每个uasset对应一个包(package)，存储一个UPackage对象时，会将该包下的所有对象都存到uasset中。
- UE的uasset文件格式很像Windows下的DLL文件格式(PE格式)，并且使用起来神似

- BulkData
  - BulkData是指一大块数据, 它也被存在uasset文件中
  - 但是加载对象的时候可以不加载它，等到需要时在问LinkerLoad要数据，例如UTexture2D的纹理数据的加载
- Cook
- 压缩

# FLinkerLoad
- 负责将uasset文件中的对象加载到内存中，起桥梁作用

- 关于异步加载
  - AsyncLoading,AsyncLoadingThread

# FLinkerSave
- 负责将内存Package中的对象存储到uasset文件
