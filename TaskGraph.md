# TaskGraph
- FBaseGraphTask
  - 每个事件基类
- TGraphTask: FBaseGraphTask
  - 事件模板
  - TGraphTask<FTaskClass>::CreateTask().ConstructAndDispatchWhenReady

- FTaskThreadBase : public FRunnable, FSingleThreadRunnable
  - 这个类里放的是事件流TArray<FBaseGraphTask*> NewTasks，以及相应处理事件流的一些方法
  - 执行Task的线程的基类
  - 但外部线程不使用它，因为这些线程是在别处创建的
- FNameTaskThread: FTaskThreadBase
  - UE4里内置的用这个，如游戏线程，渲染线程

- FRunnable
  - 线程执行体
- FRenderingThread: FRunnable
  - Run调用执行渲染线程的事件流

- FRunnableThread 没有继承
  - 可运行线程的接口, 此接口用于管理线程生命周期
  - 包含一个FRunnable与相应的TLS实现
    - TLS搜了一下，简单来说，相同的变量，每个线程可以有不同的值
  - 创建线程 Thread = FRunnableThread::Create
    - 如果只有一个简单的任务
    - 不用新开线程，直接创建分发一个task就行，会有一个线程去执行
    - TGraphTask<FTaskClass>::CreateTask().ConstructAndDispatchWhenReady

- FWorkerThread
  - FTaskThreadBase(事件流)与FRunnableThread(线程执行与TLS)的引用
  - 封装相应对象FTaskThreadBase与FRunnableThread

- FTaskGraphInterface
  - 可以理解成一个单例管理类
    - 管理所有FWorkerThread(线程与事件流)
    - 根据类型得到对应的FWorkerThread



# FTaskGraphInterface初始化相应的渲染线程所需的FNameTaskThread
- 以及调用StartRenderingThread创建渲染线程执行体FRunnable的子类FRenderingThread
  - 注意有个全局变量GIsThreadedRendering开始标为true

- FRenderingThread开始执行RenderingThreadMain
  - 找到渲染线程的FWorkerThread，初始化相应TLS的ID值
    - 如上所说，循环执行FTaskThreadBase::ProcessTasksUntilQuit里的事件

- 点程序退出等，ProcessTasksUntilQuit中断，相应渲染线程上的数据开始清理
