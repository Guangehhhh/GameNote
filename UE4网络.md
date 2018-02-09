# 网络
- 广义的客户端-服务器模型
- 服务器还是控制着游戏状态的变化
- 客户端运行着和服务器一样的代码判断游戏的运行状态
- 服务器通过复制replication的方式将游戏状态信息发送到客户端
  - 客户端和服务器也可以replication交换信息
- 如果我是Server，我可以将自己的状态传给所有的客户端。
- 如果我是Client，我可以把我请求的运动发送给服务器
- 并从服务器收到游戏状态信息，发送到客户端上渲染到屏幕上
- 服务器管理游戏状态就是在管理世界中的所有Actors的状态
- 客户端上的游戏状态可以说是服务器端的近似状态，需要被督导和斧正
- 为了节省性能，我们得对Actor进行一些分类
  - 这些标志可以表明这个Actor的那些环节将会被replication
- 我们使用两种标志位将bNoDelete || bStatic 的Actor进行排除
  - 这两种标志位的Actor一般或是StaticMesh等或者GameInfo
- 在服务器和客户端都存在的Actor我们标注其为Authority


# 网络
- UNetDriver
  - 成员： UNetConnection* ServerConnection  代表了客户端向服务器建立的连接
    - TArray<UnetConnection* >ClientConnections 服务器持有的客户端的连接

- ue4是通过UActorChannel来实现Actor在服务器和客户端之间同步
- 个需要在服务器和客户端之间同步的actor都会关联一个UActorChannel
- TMap<TWeakObjectPtr<AActor>,class UActorChannel*> ActorChannels


# UE4中的几种同步方式
- Actor Replication
- Property Replication
- Function Call Replication
- Actor Component Replication
- Generic Subobject Replication


# RPC
- enum ENetRole
  - ROLE_None,
  - ROLE_SimulatedProxy,
    - 只能用来接收服务器给它同步的信息，而不能向服务器发送信息
  - ROLE_AutonomousProxy,
    - 不仅可以用来接收服务器给它同步的信息，还可以利用调用Server函数
    - 还可以利用调用Server函数，让服务器自行执行一段预设的代码，这个过程就是Server函数
  - ROLE_Authority,
  - ROLE_MAX,

- RPC是指远程过程调用
  - 在蓝图中也成为Event Replication，是一种利用网络手段，将函数调用和执行分开的方式
- 先得来认识一下主控（Authority）
  - 服务器对游戏状态拥有主控权力，机器之间出现数据的差异，都以服务器的为准
    - 可以说服务器对游戏具有Authority，任何与游戏规则，游戏状态有关的变量以及参与复制的对象
    - 都以服务器的为准，客户端的只是一个复制品
- 不在一个内存空间，不能直接调用
- 需要通过网络来表达调用的语义和传达调用的数据
- 通过在客户端和服务器之间建立TCP连接
  - 远程过程调用的所有交换的数据都在这个连接里传输

- RPC 的主要功能目标是让构建分布式计算（应用）更容易
  - 在提供强大的远程调用能力时不损失本地调用的语义简洁性。
- 是不是服务器就对所有的Actor拥有Authority呢？
  - 答案是：不是
  - 只出现在客户端的UI，或者说一些本地产生的特效效果
  - 此时的客户端就是这些Actor的Authority
- 要看谁是这个Actor的Authority，就看服务器是否有这个对象
  - 如果没有，客户端就对这个Actor拥有Authority
  - Switch Has Authority
- 一般情况下，我们可以把拥有Authority的对象看作是服务器
  - 相反如果没有的话就看作客户端

- RPC是一种利用网络手段，将函数调用和执行分开的方式，一共有三种RPCs
  - Server函数
    - 客户端调用，服务器执行
  - Client函数
    - ROLE_Authority的角色调用，ROLE_AutonomousProxy的角色执行
  - Multicast函数
    - 服务器调用，服务器和所有客户端执行

- 程调用函数主要包括 3 种类型：Multicast 广播、Run on Server 在服务端执行 和 Run on owning Client 在客户端执行。

广播函数在服务器上调用和执行，然后自动转发给客户端。 在服务端执行的函数由客户端调用，然后仅在服务器上执行。 在客户端执行的函数由服务器调用，然后仅在自有客户端上执行
