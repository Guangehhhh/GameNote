## IMessageBus
- 消息总线的接口
  - 消息总线是促进应用程序（可能分布的）模块间通信的主要逻辑组件
  - 使用消息传递作为其基础架构模式。它随后允许注册发件人和收件人对象
  - 称为消息端点，以用户定义消息的形式交换结构化数据
  - 根据使用情况，消息分为命令，事件和文档。在虚幻引擎中，所有这些消息都是用常规的内置或用户定义的UStruct建模的，可以是空的或包含数据
  - UProperty领域的形式[2]。在被调度之前，消息被内部包装到所谓的消息上下文对象中（IMessageContext）
    - 其中包含有关邮件的其他信息，例如发送邮件的时间以及发件人和收件人

  - 消息的发送和接收不限于同一线程或进程内的消息端点，而可以扩展到运行在同一台计算机上的其他应用程序或连接到具有所谓消息传输的网络的其他计算机上
    - 插件，例如虚幻引擎附带的UdpMessaging插件。消息总线的主要目标是隐藏技术
  - 底层传输机制的细节，以便用户可以专注于实现他们的分布式应用程序而不是
  - 担心数据如何从一个端点转移到另一个端点。消息总线使其看起来好像所有发件人和收件人都位于此处
  - 在同一过程中，不管实际情况是否如此。可以限制消息的范围
  - 使用所谓的消息范围（EMessageScope）
  - 所有消息收件人（实现IMessageReceiver接口的对象）都必须通过消息总线注册
  - 请参阅IMessageBus.Register方法。在收件人销毁之前，它应该使用查看IMessageBus.Unregister来注销自己
  - 方法。消息发送者（实现IMessageSender接口的对象）不会向总线注册，而是通过一个
  - 每次发送消息时引用自己。 IMessageReceiver和IMessageSender都是非常低级的接口
  - 进入消息系统。大多数用户更喜欢使用FMessageEndpoint类的实例，而这提供了很多
  - 更方便的发送和接收消息。

  - 请注意远程过程调用（RPC）等更高层次的概念不是消息传递系统的一部分，但我们可能会提供它们 将来会有不同的功能

  - 虚幻引擎中的消息总线支持以下两种常见消息传递模式：请求-回复 和 发布-订阅
  - 在请求-回复（Request-Reply）模式中，使用IMessageBus.Send方法将消息发送给一个或多个特定消息收件人
    - 消息收件人实现了查看IMessageRecipient接口，并通过它们的地址（FMessageAddress）唯一标识收到消息后
      - 收件人可能会使用相同的IMessageBus.Send方法回复另一条消息
      - 或者，以前收到的消息可能会使用请参阅IMessageBus.Forward方法转发给另一个收件人
      - 这种模式很有用消息接收者已经彼此了解并希望直接通信，即交换命令或事件

  - 在发布-订阅（Publish-Subscribe）模式中，使用查看IMessageBus.Publish方法将消息发送到总线上的所有消息收件人
  - 只有先前使用IMessageBus.Subscribe方法订阅发送消息类型的收件人才会实际收到消息
  - 所有其他收件人将不会收到该消息。
  - 收到发布的消息后，收件人可能会回复另一条消息
    - 可以使用IMessageBus.Send方法直接发送给消息发送者
    - 也可以使用发布另一条消息IMessageBus.Publish方法
    - 这种模式对于发现公交车上的收件人和未知的邮件收件人非常有用

  - 大多数应用程序经常使用Request-Reply和Publish-Subscribe的组合
  - 分布式的典型实现 service providers 和 service consumers （服务可以是系统提供的任何有用的功能）
    - 该service consumers通常会发布特别消息以发现服务提供商
    - Service providers订阅了这些特殊的消息，并将与另一个消息，包含有关提供商的信息，并直接发回给回应service consumers
    - service consumers现在知道Service providers的存在和它的message address，然后它可以通过请求服务直接将所有后续消息直接发送给它

  - 将发送和发布的消息分派给正确的收件人由消息路由器（Message Router）完成，该消息路由器是内部的消息总线的组件
  - Message Router维护允许它确定的已知消息端点的地址簿消息的目的地
  - 如果消息不能被传递，则将其转发到所谓的死信通道（Dead Letter Channel）
    - 这是一个队列，可用于调试目的的消息[3]。
  - 通过注册所谓的消息拦截器（Message Interceptors），可以在消息被路由到收件人之前拦截消息与消息总线
    - 实现IMessageInterceptor接口
    - 这个功能可以实现需要检查的高级用例处理消息内容，如消息过滤 和丰富，分割和聚合，重新排序或认证。
  - 消息传递系统还提供了一个通过所谓的Message Tracer（一个内部对象）调试系统本身的工具
    - 实现可以使用IMessageBus.GetTracer（）方法访问的IMessageTracer接口。消息跟踪器是
    - 目前在Messaging Debugger中使用
    -  一个用于Unreal Frontend和虚幻编辑器中消息系统的可视化调试工具
