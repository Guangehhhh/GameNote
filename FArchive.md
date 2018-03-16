## FArchive
- 可用于加载，保存和垃圾回收的基类
- 操作符可以通过想要序列化FName实例的子类来实现
  - 例如FText::SerializeText(* this, Value);

- virtual void SerializeBits(void* V, int64 LengthBits)
- CustomVer
  - 从存档中查询定制版本。如果正在使用档案写入，那么自定义版本必须已经被注册

# UAsset文件格式
- UE中使用统一的格式存储资源(uasset, umap)，每个uasset对应一个包(package)，存储一个UPackage对象时，会将该包下的所有对象都存到uasset中。
- UE的uasset文件格式很像Windows下的DLL文件格式(PE格式)，并且使用起来神似(下一节分析Linker)
  - File Summary 文件头信息
  - Name Table 包中对象的名字表
  - Import Table 存放被该包中对象引用的其它包中的对象信息(路径名和类型)
  - Export Table 该包中的对象信息(路径名和类型)
  - Export Objects 所有Export Table中对象的实际数据
- FObjectExport
- FObjectImport

# BulkData
  - BulkData是指一大块数据, 它也被存在uasset文件中
  - 但是加载对象的时候可以不加载它，等到需要时在问LinkerLoad要数据，例如UTexture2D的纹理数据的加载
# Cook

# FPackageIndex
# FLinkerSave
- 负责将内存Package中的对象存储到uasset文件
# FLinkerLoad
- 负责将uasset文件中的对象加载到内存中，起桥梁作用
- FLinkerLoad::ELinkerStatus FLinkerLoad::Tick( float InTimeLimit, bool bInUseTimeLimit, bool bInUseFullTimeLimit );
  - 用于解析uasset文件，当bInUseTimeLimit为true时, Tick不会一次性做完解析工作，分时间片进行加载，在一帧里，Tick()不会占用太多时间。
  - UObject* CreateExport( int32 Index );
  - 创建Export Table中Index位置的对象。
  - void LoadAllObjects(bool bForcePreload);
  - 加载Export Table中的所有对象
- 读取包中对象时，可以一次性加载所有export table中的对象
  - 也可以按需加载某个对象(比如包P0中的对象A被包P1引用，在加载P1时，可能只会加载P0中的对象A，而不是P0中的所有对象)


- 关于异步加载
  - AsyncLoading,AsyncLoadingThread

# 存档读档
- ArchiveBase
- 二进制数组 TArray <uint8>
- 变量格式 - >二进制数组
- 二进制数组 - >硬盘
- 压缩二进制文件
- BufferArchive缓冲区存档是一个二进制数组
  - class FBufferArchive ： public FMemoryWriter，public TArray < uint8 >
  - TArray <uint8>和一个MemoryWriter
- FMemoryReader
  - class FMemoryReader ： public FMemoryArchive
  - 从二进制数组读取数据，这是由FileManager检索的
  - 用于从指定的内存位置读取任意数据
- 关于UE4存档系统最难的概念是<<运算符可能意味着
  - 将数据从存档中取出并放入变量中
  - 将变量中的数据转换为存档二进制格式
  - 取决于上下文！
- 如何将数据写入二进制文件的顺序必须是您读取的确切顺序！





# SaveGame
- TArray<uint8> ObjectBytes
- FMemoryWriter MemoryWriter(ObjectBytes, true);
- FObjectAndNameAsStringProxyArchive Ar(MemoryWriter, false);
- SaveGameObject->Serialize(Ar);
# LoadGame
- TArray<uint8> ObjectBytes;
- bool bSuccess = SaveSystem->LoadGame(false, * SlotName, UserIndex, ObjectBytes);
- FMemoryReader MemoryReader(ObjectBytes, true);
- UClass* SaveGameClass = FindObject<UClass>(ANY_PACKAGE, * SaveGameClassName);
- OutSaveGameObject = NewObject<USaveGame>(GetTransientPackage(), SaveGameClass);
- FObjectAndNameAsStringProxyArchive Ar(MemoryReader, true);
- OutSaveGameObject->Serialize(Ar);

# SaveJason
- 创建JasonObject 然后调用Set相关变量函数
- 创建FString OutPut
- 创建TSharedRef< TJsonWriter<> > Writer = TJsonWriterFactory<>::Create(&OutputString);
- 调用FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);
# LoadJason
- TSharedPtr<FJsonObject> JsonReaderObject;
- TSharedRef< TJsonReader<> > Reader = TJsonReaderFactory<>::Create(OutputString);
- FJsonSerializer::Deserialize(Reader, JsonReaderObject)
-  JsonReaderObject->GetStringField
