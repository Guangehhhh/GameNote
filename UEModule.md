## Module
---
# ue4 模块的构建和加载
- 一个包含*.build.cs的目录就是一个模块
- 这个目录里的文件在编译后都会被链接在一起，比如一个静态库lib，或者一个动态库dll
- 不管是哪种形式，都需要提供一个给外部操作的接口，也就是一个IModuleInterface指针
- 注意这里并不是说调用模块内任何函数（或类）都要通过该指针来进行，实际上外部代码只要include了相应的头文件，就能直接调用对应的功能了（比如new一个类，调一个全局函数等），因为实现代码要么做为lib被链接进exe，或是做为dll被动态加载了
- 外部获取这个指针，只有一个办法，就是通过FModuleManager上的LoadModule/GetModule
- FModuleManager去哪里找需要的模块呢？

1、在动态链接情况下，根据名字直接找对应的dll就行了，做为合法ue4模块的dll，必定要导出一些约定的函数，来返回自身IModuleInterface指针。

2、在静态链接时，去一个叫StaticallyLinkedModuleInitializers的Map里找，这就要求所有模块已把自己注册到这个Map里。

以上2点基本就是FModuleManager::LoadModule的内容
- FStaticallyLinkedModuleRegistrant是一个注册辅助类，它利用全局变量的构造函数被crt自动调用的特性，实现自动注册逻辑。

它在构造函数里把一个“注册器”添到前面说的StaticallyLinkedModuleInitializers里，而这个注册器的内容就是创建并返回相应模块

- 如果模块本身实在没什么特别的，那么就：

IMPLEMENT_MODULE(FDefaultModuleImpl, ModuleName)

FDefaultModuleImpl是一个IModuleInterface的空实现，什么都没干，ModuleName则是模块名字，很重要，其它地方要加载模块，就靠这个名字了

- 如果模块有自己的初始化逻辑，那么应该实现自己的IModuleInterface子类，比如说MyUtilModule，然后：

IMPLEMENT_MODULE(MyUtilModule,MyUtil)

这里类名带Module后缀，而模块名不带，是ue4的惯例

- 如果模块是一个包含游戏逻辑的模块（相对于通用功能型模块），那么可以用一个特殊的宏IMPLEMENT_GAME_MODULE，不过目前看来它和前者没啥差别，可能未来有所扩展

4、更特别的是，如果模块是表示当前游戏项目的“主模块”，那么应该用一个更特殊的宏IMPLEMENT_PRIMARY_GAME_MODULE，而且在构建一个UEBuildGame类型的Target时，必须有一个这样的模块。

当IS_MONOLITHIC，构建成单个exe时



- 统一来看，其实也就比普通模块多做了三件事，一是设置游戏名字GInternalGameName，二是设置了GIsGameAgnosticExe=false，三是多了个UELinkerFixupCheat暂且不究。

GIsGameAgnosticExe是一个有趣的变量，它表示当前这个exe是特定于某游戏的？还是一个通用的加载器？

如果整体编成一个exe文件，即IS_MONOLITHIC，那肯定是特定于某游戏，这时游戏名GInternalGameName必定是已知固定的，所以以宏参数的形式直接硬编码在exe里了。

与之相反，当使用模块化构建时（通过给ubt传入-modular参数），将会生成一个exe和一堆dll，并且能加载任何其它以dll形式存的游戏模块，游戏名是未知的，是通过命令行参数来决定要加载哪一个游戏，所以也就用不着GInternalGameName变量了


- 当构建一个工具程序（TargetType.Program）时，可以使用专门定制的IMPLEMENT_APPLICATION：

最特别的是它竟然提供了一个FEngineLoop GEngineLoop，而这个变量存在于一般的Game/Editor构建时都会链接的【Launch模块】中。

不过这也正常，因为在工具程序中一般是要自己写main函数的，所以就用不着重量级的Launch模块了
