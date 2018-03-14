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

# Asset Registry
- 是一个编辑器子系统，它在编辑器加载资源过程中，异步地收集卸载的资源的信息
- 该信息存储在内存中，以便编辑器不必加载这些资源就可以创建资源列表
- 该信息是权威信息，且随着资源在内存中发生改变或者文件在磁盘中发生改变，该信息可以自动更新

- FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
- TArray<FAssetData> AssetData;
- const UClass* Class = UStaticMesh::StaticClass();
- AssetRegistryModule.Get().GetAssetsByClass(Class, AssetData);

- 资源注册表是一个存储资源的元数据的系统，允许您搜索及查询这些资源
- 编辑器使用此资源注册表来显示内容浏览器中的信息，但是您也可以从游戏性代码使用该注册表，来查找当前没有加载的游戏资源的相关元数据
- 要想使得一个资源的数据是可搜索的，您需要给该属性添加"AssetRegistrySearchable"标签
- 查询资源注册表会返回FAssetData类型的对象，它包含了关于该对象的信息和一个 键-》值 对映射表，该映射表包含了标记为可搜索的属性

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
- 处理成组的未加载的资源的最简单方法是使用 ObjectLibrary
- ObjectLibrary 是一个对象，包含了一系列继承共享基类的未加载对象或者未加载对象的FAssetData
- 您可以通过提供一个搜索路径来加载一个对象库，它将加载那个路径中的所有资源
- 您可以为不同类型指定您的部分内容文件夹
- ObjectLibrary->LoadAssetDataFromPath(TEXT("/Game/PathWithAllObjectsOfSameType");
- ObjectLibrary->LoadAssetsFromAssetData();

# UAssetManager
- UAssetManager :: GetIfValid
- 与AssetRegistry一起用
- 具体应用可以参考DataAsset UKismetSystemLibrary::GetObjectFromPrimaryAssetId


# FStreamableManager SynchronousLoad
- 引用了磁盘上一个资源的 FStringAssetReference ，那么怎样真正地异步加载它哪？
  - FStreamableManager是完成这个处理的最简单方法
    - 首先，您需要创建一个 FStreamableManager ，我建议你把它放到某个全局游戏单例对象中
    - 比如在 DefaultEngine.ini 中使用 GameSingletonClassName 指定的对象
    - 然后，您可以把 StringAssetReference 传给该对象，并启动进行加载
    - SynchronousLoad 将会进行简单的、阻碍加载，并返回该对象
    - 这种方法对于较小的对象可能很好，但是它可能会使您的主线程停顿时间过长
    - 出现那种情况，您将需要使用 RequestAsyncLoad ，它将会异步地加载一组资源，并在完成后调用一个代理
