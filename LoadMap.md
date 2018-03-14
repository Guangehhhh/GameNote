## Level Streaming流关卡
- 每一个World里面至少有一个PersistentLevel以及0-N个StreamingLevels
- 控制流关卡的加载总体上来说有两种方式
	- 一种是通过关卡流体积控制（LevelStreamingVolume）
		- LevelStreamingVolume相当于一个定制的触发器，当玩家摄像机（注意是玩家摄像机）进入LevelStreamingVolume体积内的时候
		- 对应的流关卡就会加载（对应关系是通过下图操作设置的 点击levels旁边的按钮打开leveldetails窗口 inspectlevel找到需要加载的流关卡添加对应的LevelStreamingVolume）
	- 另一种是通过脚本（代码）逻辑控制，想怎么写就怎么写，简单的方式就是设一个触发器在玩家进入触发器的时候调用LoadStreamLevel

# World Composition
- 由于每个level关卡都可能非常大，我们在编辑器里面一点点调整流关卡里面的Actor位置实在是过于麻烦，所以UE提供了世界构成器功能
- 把N个关卡用拼图的形式拼接成一个大世界地图
- 需要在当前的WorldSetting里面勾选 Enable World Composition
- 世界构成器默认的加载逻辑就是当玩家距离要加载的关卡满足一定数值时就会加载对应的子关卡
-
# LoadMap地图切换流程分析
- make sure level streaming isn't frozen
- FCoreUObjectDelegates::PreLoadMap.Broadcast(URL.Map);
- UTexture2D::CancelPendingTextureStreaming();
- clean up any per-map loaded packages for the map we are leaving
- cleanup the existing per-game pacakges
- FlushAsyncLoading
- CancelPendingMapChange
- WorldContext.SeamlessTravelHandler.CancelTravel();
- double	StartTime = FPlatformTime::Seconds();
- GInitRunaway();
- Unload the current world
- trim memory to clear up allocations from the previous level
- IStreamingManager::Get().CancelForcedResources();
- appDefragmentTexturePool();
- VerifyLoadMapWorldCleanup();
-If this world is a PIE instance, we need to check if we are traveling to another PIE instance's world.
	- If we are, we need to set the PIERemapPrefix so that we load a copy of that world, instead of loading the
	- PIE world directly.

---
#一般的地图切换流程
# SetClientTravel
- 运行的时候，执行端就会从本地文件夹里面搜索到这个地图并进行加载
- 注意，如果他是一个纯客户端，在执行ClientTravel的时候URL只输入地图而不输入IP，而且TravelType是Relative
	- 他就会加入本地默认的7777端口的服务器，并且服务器会在Welcome的消息里面返回正确的地图信息来纠正客户端
	- 这样客户端可能就是重新加入了一次服务器。在这个过程中，客户端会清空NetDriver，重新生成PendingNetGame
	- 通过TickWorldTravel 执行Browse来与服务器重新建立链接并重新打开地图
- 如果他在URL里面输入了IP以及端口信息
	- 那么他就会从当前服务器断开并Travel到目标地址的服务器上去，而这个就是ClientTravel负责完成的主要功能
	- 想实现这个功能其实还有两个办法，一是就是在控制台命令里面输入 open 127.0.0.1:7777（假如服务器开在本地），你的客户端也会Travel到目标地址的服务器上
	- 二是调用全局的static接口UGameplayStatics::OpenLevel。不论哪种方式，本质上都是调用引擎的UEngine：：SetClientTravel（UPendingNetGame*…）函数
- ClientTravel的主要目的就是将客户端从一个服务器迁移到另一个服务器（也可以重新加入当前的服务器）

# ServerTravel
- UEngine::ServerTravel的主要功能就是让服务器去加载新的地图并且通知所有他连接下的客户端都跟着他进入到新的地图去（只能在客户端运行）
	- 同样，ServerTravel也可以设置Relative还是Absolute，不过影响不大了，但是注意URL里面不要填写IP地址了，因为他的功能就是在本地切换地图，所以不需要添加IP地址相关信息（会崩溃）
	- 服务器是首先需要自己加载地图，然后通知客户端执行SetClientTravel跟随服务器切换level，随后读取服务器发送的WelcomMessage消息并正确的加载响应的地图
	- 执行完ServerTravel后，GameMode等所有Actor都应该是重新生成的，旧场景的对象会被在执行LoadMap时被垃圾回收掉。

- 在编辑器里面表现可能比较奇怪
	- 编辑器下执行ServerTravelURL里面只填写地图名称会发现执行后发现客户端会卡主
		- 其实是因为下面的代码，编辑器的GIsClient属性为true（正常一个DedicateServer一定为false）
			- 导致服务器在LoadMap的时候不能正常初始化监听的NetDriver，因此客户端无法与服务器建立连接而一直处于Pending状态

# Browse
- 每次调用ClientTravel以及ServerTravel的时候都一定会用到
- Browse就像是加载新地图的硬重置，一定会断开客户端与服务器的连接，导致非无缝的切换
- 因为这里面会重置客户端的NetDriver，创建UPendingNetGame并进行相关初始化。所以，只要我们没有勾选无缝切换地图的选项，就一定会执行该操作

---
# 无缝地图切换SeamlessTravel
- 在不断开连接的情况下切换地图
- 如果客户端想切换地图，那么肯定是服务器先切换的地图，否则客户端无法在一个与服务器不同地图且保持连接的情况下正常游戏
- 无缝切换的正常情况只有一种：服务器切换地图，客户端与服务器在保持连接的情况下也跟着切换地图
- 缝加载的使用情景类似于一个房间服务器，玩家们从A场景完成一项任务或者结束一次比赛后重新开始新的任务或比赛
	- 而前面的流关卡更偏向与RPG式的大地图探索
- 在无缝连接的进行时没有处理的Actor就会被删除
- 不要直接调用ClientTravel！如果你理解客户端与服务器之间的关系，你就会明白，二者必须要保持一致
	- 只在客户端去执行无缝Travel是不合法的操作
- 让服务器去调用ServerTravel同时勾选GameMode里面的UseSeamlessTravel
- 编辑器模式下不支持SeamlessTravel
- UGameMapsSettings::TransitionMap 属性配置一个过度地图

# 无缝切换流程
- 无缝切换不会走Browse函数，自然也就不会断开连接
- travel过程当中有一个过度地图TransitionMap
- 所以先要把相关的Actor保存到TransitionMap ，再从TransitionMap 保存到目标场景中去
# 无缝切换时的一些问题与解决方法
- 无缝切换通过一个FSeamlessTravelHandler类Tick操作覆盖了原本的Browse操作
	- 这个过程中不会直接释放地图资源，而是通过一定机制将Map数据通过拷贝进行转移
	- 可以看到在函数FSeamlessTravelHandler::CopyWorldData里面会将当前的World的NetDriver赋值给要加载的World，从而保持了连接不断
- 客户端在一开始运行时会先打开默认的场景，然后发送连接到服务器的请求，服务器确认后才能加载新的地图
	- 可以给其设置一个默认的空场景，这个场景只显示加载的过场动画
