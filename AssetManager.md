# AssetLoad
# FStringAssetReference
  - 是一个简单的结构体，包含了一个具有资源完整名称的字符串
  - 如果您在类中创建一个那种类型的属性，那么它将会显示在编辑器中，就像是个 UObject * 属性一样
  - 它还可以正确地处理烘焙和重定向
  - TryLoad()
    - 调用LoadObject
    - 循环迭代UObjectRedirector的DestinationObject
  - ResolveObject()
    - 调用FindObject()
    - 循环迭代UObjectRedirector的DestinationObject
  - SetPath(FString)
# TAssetPtr
- TAssetPtr基本上就是一个封装了 FStringAssetReference 的 TWeakObjectPtr
  - 它使用一个特定的类作为模板，以便您能限制编辑器用户界面，使其仅允许选择特定的类
  - 如果所引用的资源存在于内存中，那么 TAssetPtr.Get() 将返回该资源
  - 如果该资源不在内存中，那么您那可以调用 ToStringReference() 来查找它引用的资源
    - 并使用下面介绍的方法加载该资源，然后再次调用 TAssetPtr.Get() 解除对该资源的引用



- AssetID对应在C++中其实就是TAssetPtr<Template>的数据类型
  - AssetID相当于保存了一份package的path，并没有真正的保存Asset的数据；
  - C++中有FStringAssetReference的类型
  - InAssetId.ToStringReference();
---
# UAssetManager
- 用于管理primary assets和asset bundles，这些东西在runtime的时候很有用
- AssetManager是一个单例的UObject，它提供了在runtime的时候进行查询以及读取Primary Assets的操作
  - 这个东西是原本用于取代ObjectLibraries当前提供的操作的，并且可以对FStreamableManager进行一层封装来处理Async Loading的操作。
  - 引擎内置的asset manager只能提供基本的管理操作，但是一些更加复杂的东西，例如caching需要自己实现。Asset Manager的基本操作如下：
    - Get()：单例的Get操作。
    - ScanPathsForPrimaryAssets(Type, Paths, BaseClass)：
      - 这个函数可以查询特定目录下的某类特定的primary asset
        - 如果在Editor下则直接读取磁盘上的信息
        - 如果在cooked工程中会从asset registry cache中读取asset信息。
    - GetPrimaryAssetPath(PrimaryAssetId) :
      - 获得PrimaryAssetId对应的asset的object path。
    - GetPrimaryAssetIdForPath(StringReference):
      - 获得object path对应的asset的primary asset id信息，以Type:Name的形式。
    - GetPrimaryAssetIdFromData(FAssetData)：
      - 通过FAssetData来获得对应的Type:Name形式的Primary Asset Id。
    - GetPrimaryAssetData(PrimaryAssetId)：同理
    - GetPrimaryAssetDataList(Type)：
      - 返回一个所有对应类型的asset列表。
    - GetPrimaryAssetObject(PrimaryAssetId)：
      - 查询这个对应的UObject是否在内存中。如果这个UObject不再内存中，则返回nullptr。
    - LoadPrimaryAssets(AssetList, BundleState, Callback, Priority)：
      - 异步载入这些primary assets和BundleState所引用的所有其他assets。
      - 返回一个FStreamableHandle的shared_pointer用于追踪，并且在Loading完成后调用回调函数。
    - UnloadPrimaryAssets(AssetList)：
      - 这个函数会调用这些primary assets的GC。
    - ChangeBundleStateForPrimaryAssets(AssetList, Add, Remove)：
      - 这个函数可以用一种更复杂的方式去处理bundle state

- 与AssetRegistry一起用
- UAssetManager :: GetIfValid()
- 具体应用可以参考DataAsset UKismetSystemLibrary::GetObjectFromPrimaryAssetId


---
# Primary Assets
- Primary Assets指的是在游戏中可以进行手动载入/释放的东西。
  - 包括地图文件以及一些游戏相关的物件，例如character classs或者inventory items
- Primary Asset指的是可以针对于UObject::GetPrimaryAssetId()返回一个有效的值的UObject
- 所有在/Game/Maps路径下的Maps会被设为Primary Assets，而所有其他的assets如果需要设定为Primary Assets，都需要手动指定
- 所有的Primary Assets是通过FPrimaryAssetId来进行引用的，FPrimaryAssetId拥有如下的属性：
  - PrimaryAssetType：这指的是一个用于描述物件的逻辑类型的名字，通常是基类的名字。
    - 例如，我们有两个继承自同一个本地类AWeapon的BPAWeaponRangedGrenade_C和AWeaponMeleeHammer_C
    - 那么它们会有同样的PrimaryAssetType——"Weapon"
  - PrimaryAssetName：这指的是用于描述这个asset的名字。通常来说，这就是这个object的名字（short name），但是对于例如说maps来说，这个值就是对应的asset path。
  - PrimaryAssetType:
    - PrimaryAssetName可以组成整个游戏实例中的asset的唯一的描述。
    - 当客户端在和服务端通信的时候，就可以通过这个字符串来确认某个物件。
    - 例如，"Weapon:BattleAxe_Tier2“本质上和”/Game/Items/Weapons/Axes/BattleAxe_Tier2.BattleAxe_Tier2_C"是一样的。
- 在FPrimaryAssetId中分别有两个FName的Tag，分别是PrimaryAssetTypeTag和PrimaryAssetNameTag
  - 因此，当一个Primary Asset被保存了之后，就可以直接在Asset Registry中通过这两个Tag来找到这个Asset
# Secondary Assets
- Secondary Assets指的是其他的那些Assets了，例如贴图和声音等
  - 这一类型的assets是根据Primary Assets来自动进行载入的
# Asset Bundle
- Asset Bundle在Unity中很常见了——总体来说就是显式的primary assets相关的assets列表。
- 从底层来看，一个Bundle本质上其实就是一个FName到TArray<FStringAssetReferences>的map。
  - 每一个bundle都与一个Primary Asset Id相关。但是……这个东西也可以是一个动态的asset。
- 如果需要对普通的Primary Asset生成对应的Asset Bundle
  - 我们需要在自己的Object中加入一个FAssetBundleData的UStruct
  - 并且在进行save操作的时候将这个UStruct进行填充
  - 然后，这些数据就会被写在asset registry tag中，并且在这些asset数据被读取的时候，这个UStruct会被识别并处理。
- AssetBundleMeta Tag可以被设定为确定的AssetPtr或者StringAssetReference。
- 可以通过调用AddDynamicAsset函数来在runtime的时候处理一些特定的Asest Bundle.
- 在Asset Bundle中的任何东西，都会被认为是该primary asset的一部分
  - 这在进行Chunking的时候会非常有用


---
# Asset Registry
- 是一个编辑器子系统，它在编辑器加载资源过程中，异步地收集卸载的资源的信息 FAssetData
  - 该信息存储在内存中，以便编辑器不必加载这些资源就可以创建资源列表
  - 该信息是权威信息，且随着资源在内存中发生改变或者文件在磁盘中发生改变，该信息可以自动更新

- FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
- TArray<FAssetData> AssetData;
- const UClass* Class = UStaticMesh::StaticClass();
- AssetRegistryModule.Get().GetAssetsByClass(Class, AssetData);

- 资源注册表是一个存储资源的元数据的系统，允许您搜索及查询这些资源
- 编辑器使用此资源注册表来显示内容浏览器中的信息，但是也可以从游戏性代码使用该注册表，来查找当前没有加载的游戏资源的相关元数据
- 要想使得一个资源的数据是可搜索的，您需要给该属性添加"AssetRegistrySearchable"标签
- 查询资源注册表会返回FAssetData类型的对象，它包含了关于该对象的信息和一个 键-值 对 映射表，该映射表包含了标记为可搜索的属性

# FAssetData
-  A struct to hold important information about an assets found by the Asset Registry
  This struct is transient and should never be serialized
-

# CreateAsset流程
- void SAssetView::AssetRenameCommit
  - SAssetView::CreateAssetFromTemporary
    - UAssetToolsImpl::CreateAsset
      - FAssetToolsModule& AssetToolsModule = FModuleManager::LoadModuleChecked<FAssetToolsModule>("AssetTools");
          - Asset = AssetToolsModule.Get().CreateAsset(InName, PackagePath, AssetClass, Factory, FName("ContentBrowserNewAsset"));
            - SanitizePackageName
            - CanCreateAsset
            - CreatePackage
            - FactoryCreateNew
            - FAssetRegistryModule::AssetCreated(NewObj);
            - UAssetToolsImpl::OnNewCreateRecord(AssetClass, false);
  - RefreshThumbnail
  - TArray<FAssetData> AssetDataList;new(AssetDataList) FAssetData(Asset);
# SaveAsset 流程
- FAssetEditorToolkit里包含了所有asset editing 的函数
  - FAssetEditorToolkit::SaveAsset_Execute
    - TArray<UObject*> ObjectsToSave;   
    - GetSaveableObjects(ObjectsToSave);
    - ObjToSave循环PackagesToSave.Add(Object->GetOutermost());


    - FEditorFileUtils::PromptForCheckoutAndSave
      - 有选择地提示用户应该保存提供的哪些包，然后另外提示用户检出任何一个提供的源代码管理的软件包
      - 如果用户从任一对话框中取消出路，则不保存包
        - 这是可能的用户将会再次提示，如果保存过程因任何原因失败。
        - 在这种情况下，用户将被逐包提示，允许他们重试保存，跳过尝试保存当前包，或者再次取消整个对话框。
        - 如果用户跳过保存未能保存的包，包将被添加到可选的OutFailedPackages数组中，并继续执行。
      - 所有软件包保存（或不保存）后，都提供给用户有关可在磁盘上写入但不在源代码控制中的任何软件包的警告
      - 以及有关哪些软件包未能保存的警告

      - PackagesToSave要保存的包列表。地图和内容包都受支持
      - bCheckDirty如果为true，则只会保存PackagesToSave中脏的包
      - bPromptToSave如果为true，系统将提示用户要保存的包列表，否则保存的所有包都会保存
      - OutFailedPackages [out]如果指定，将填写所有未能成功保存的软件包
      - bAlreadyCheckedOut如果为true，则不会提示用户使用源代码管理对话框
      - bCanBeDeclined如果为true，则除“取消”之外还提供“不保存”选项，该选项不会导致取消返回代码




# UObjectLibrary
- 处理成组的未加载的资源的
- ObjectLibrary 是一个对象，包含了一系列继承共享基类的未加载对象或者未加载对象的FAssetData
- 您可以通过提供一个搜索路径来加载一个UObjectLibrary，它将加载那个路径中的所有资源
- ObjectLibrary->LoadAssetDataFromPath(TEXT("/Game/PathWithAllObjectsOfSameType");
- ObjectLibrary->LoadAssetsFromAssetData();



# FStreamableManager SynchronousLoad
- Streamable Managers负责进行读取物件并将其放在内存中
  - FStreamableManager可以用来处理assets的读取，并且将其在被需要的时候保存在内存中
    - 针对不同的操作，可以有不同的Streamable Manager
  - FStreamableManager可以与FStreamableHandle一起工作，来更好的处理物件在内存中的生命周期
  - FStreamableHandle是一个用于处理assets读取的结构体
    - 通常来讲，在streaming operation会返回一个shared pointer，这个shared pointer就是用于追踪这个结构体的
      - 当一个handle是active的时候，它就可以确保它引用的那些assets是在内存中的了
  - FStreamableHandle从Loading开始就被激活了，当这个handle被显式cancel或者release的时候，将被disactive
  - FStreamableHandle::CancelHandle()是用来中断Loading过程的接口
    - 这个函数被调用后，Loading将会被中断，并且将取消Loading完成过后的所有回调函数
  - FStreamableHandle::ReleaseHandle()可以用来被显示调用
    - 但是当所有的指向这个handle的只能指针被销毁的时候被隐式调用。
  - FStreamableHandle::WaitUntilComplete()将阻断线程，知道所需的asset被成功载入为止
    - 这个函数被调用时，所需的asset的载入优先级将被设为最高（通过将其移到优先级队列的top来实现），并且不会影响其他的异步载入操作
    - 因此通常比LoadObject函数更快。
  - RequestAsyncLoad是基本的stream操作
    - 可以传入一个FStringAssetReference或者一个FStringAssetReference的列表
    - 调用了这个操作之后，引擎会试图去Load这些Assets，并且在Loading完毕之后调用回调函数
    - 同时，这个函数会返回一个Streamable Handle的shared pointer来供后期调用。
  - RequestSyncLoad是RequestAsyncLoad的同步版本
    - 这个函数要么会进行异步载入并且调用WaitUntilComplete函数，要么直接调用LoadObject函数 —— 哪个更快就调哪个
  - LoadSynchronous是另一种Loading的方式，这个函数会返回一个asset，并且这个函数可以有模版安全的版本

- 引用了磁盘上一个资源的 FStringAssetReference ，那么怎样真正地异步加载它哪？
  - FStreamableManager是完成这个处理的最简单方法
    - 首先，您需要创建一个 FStreamableManager ，我建议你把它放到某个全局游戏单例对象中
    - 比如在 DefaultEngine.ini 中使用 GameSingletonClassName 指定的对象
    - 然后，您可以把 StringAssetReference 传给该对象，并启动进行加载
    - SynchronousLoad 将会进行简单的、阻碍加载，并返回该对象
      - 这种方法对于较小的对象可能很好，但是它可能会使您的主线程停顿时间过长
      - 出现那种情况，您将需要使用 RequestAsyncLoad ，它将会异步地加载一组资源，并在完成后调用一个代理
