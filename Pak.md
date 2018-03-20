# Pak
- Content Cooking
- Packaging Project

- 在PIE模式下使用Pak文件的意义不大
  - 我最开始的时候按照自己的理解，认为既然是虚拟文件系统，那么只要我正确挂载路径，那么就可以正常的访问文件数据了
    - 按照样的说法在PIE模式下，这也应该是可行的，那么这样子我在测试Pak相关功能时是不是可以不用那么麻烦的每次都需要Pakcaging Project了呢？
      - 虽然这个问题的前半段确实可行，但是后半段的问题仍然无法避免。
    - 虽然能够挂载pak文件，并能够加载其中的Package，但是只能是未被Cooked的才可以正常使用
    - 同理Packaging之后的Project也无法使用没有Cooked的Package

- 区分Mount和Load的区别
  - Mount翻译为挂载，而Load一般被翻译为加载。
    - Mount只是告诉了文件系统有哪些文件可以从当前挂载的Pak文件中读到
    - 即提供了虚拟的路径来访问Pak文件包含的文件，而Load一般就是指把文件的内容加载到内存中了
    - 挂载pak文件之后就可以通过常规的方式来访问其中的文件了，并不需要特意的去做一次加载操作，且和使用或不使用异步加载方式没有直接的关系

- pak文件中可以存放非 Package文件类型
  - 这里的意思是在使用UnrealPak.exe可以放入类似.txt的文件，挂载后可以通过PlatformFile来访问其中的文件内容

- 制作pak文件的一个小技巧
  - UE4提供了命令行工具来生成pak文件，我们只需要编写一些脚本，就可以方便按照自己的需求来生成相应的pak文件。
    - 这里说的小技巧可以让你把某个路径下的制作成pak文件时保留你想要的部分路径
  - 简单来说就是以你想要相对的路径下加一个不存在的文件，上面代码中是dummy.uasset。
    - 然后作为命令行输入的时候，工具提示找不到那个文件且不会加入那个文件。
    - 但是这个时候，你要保留的路径就留下来了。

# Package
  - 是指的在UE4中能够在直接使用的游戏资源，比如material，map，blueprint等，每一个独立的资源的对应一个Package概念
  - 一个Package可能包含多个File，比如cooked之后的usset实际上可能会分离出.uexp .ubulk等文件
  - 多个Package可以称作一个Pack，比如Add Feature Pack就是提供了一系列s Packages。
  - Package和UOject关系很紧密，直接Load的 Package就是一个UObject对象。
  - FPackageName就是我们通常在Load Asset时使用的路径形式。更为具体点就是FStringAssetReference
    - 但是注意FStringAssetReference不是一个字串，不要被名字迷惑

# PlatformFile
  - UE4中使用使用IPlatformFile来抽象出平台无关的文件操作的接口，对应游戏引擎架构的平台独立层中的文件系统
  - 相对Asset Reference方式来说这是一个低层文件访问的对象。
  - 它提供了基本的文件Read，Write，Delete等等操作。
  - 比较接近std::fstream的一些操作。
    - 针对支持的各个平台都会从IPlatformFile派生出相应的具体实现的类，比如WidowsPlatformFile。
  - 除了平台相关的的派生类以外，还有IPlatformFilePak，IPlatformFileModule等
  - 和上面FPackageName相对应的是FPaths用来处理一些PlatformFile下的文件路径的操作
  - 如果阅读IPlaformFile相关的代码就会发现IPlatformFile实际上是一个链状的结构。
      - 在其Intialize的时候，会要求传一个IPlatformFile的指针，这个将会作为其inner lower PlatformFile存在
        - 在通过一个PlatformFile尝试访问文件目录时，可以通过这个链自顶向下访问每个节点上所挂载的文件。
      - 可以这样理解，每个节点都有包含一部分文件信息，整个链上节点的信息综合起来，就是你能够访问的所有文件信息
      - 值得注意的是，这种方式是允许节点间信息冗余的
        - 如果冗余会访问第一个找到的数据
      - 既然是一个链状结构，UE4提供了FPlatformFileManager来方便set/get topmost PlatformFile

# PakPlatformFile
  - 用以管理Pak相关操作的PlatformFile类。
    - 提供了使用.pak文件的最为核心的两个操作Mount及Unmount
  - 注意一个PakPlatformFile上可以挂载多个Pak文件
    - 网上大多数挂载pak的示例中，都是直接创建新的PakPlatformFile
    - 然后把PlatformFileManager取到的当前的topmost PlatformFile作为其inner lower PlatformFile
    - 然后将这个PakPlatformFile指定为topmost。实际上如果没有特殊要求的话，没有必要去增长这个链，会造成内存的浪费
    - 一个PakPlatformFile可以挂载多个Pak文件，其内部有一个记录FPakFile的List
