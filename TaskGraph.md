# TaskGraph 研究

- FTaskGraphInterface
  -  task graph system的接口
  - 开始关闭逻辑
  - 附加到线程
  - 控制线程逻辑
  - Task回调逻辑

- FBaseGraphTask
  - 所有Task的基类
  - ENamedThreads::Type	 ThreadToExecuteOn;
  - ExecuteTask，QueueTask
  - FThreadSafeCounter

- FGraphEvent
  - FGraphEvent is a list of tasks waiting for something
  - 
