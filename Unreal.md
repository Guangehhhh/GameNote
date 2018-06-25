# AnimNode_Base
- 初始化
- 获取骨骼
- 更新
- 求值
- 求值—组建空间
- 覆盖资源


错误的引用一般肯定是你们自己的逻辑代码引起的。譬如你地图里spawn了个actor，把actor的引用在了一个静态或者gameinstance之类，再或者Asset之类的对象身上了，那么换地图要把整个world都gc掉的时候如果你不先手动清除掉这个引用，就肯定会crash。因为这个actor反向引用了level，再反向引用到world，以致于之前那关地图完全都不能被释放掉

# FRawAnimSequenceTrack
- 关键帧数组
- 每个数组将包含n元素或1个元素
- 一个元素被用作一个简单的压缩方案
- 如果所有的键都是相同的，它们将会是减少到1个键在整个序列中是常数

# FAnimSequenceTrackContainer
- 关键帧数组对应的名称和操作

# FTranslationTrack/FRotationTrack/FScaleTrack
- 关键帧数组对应的时间，旋转，大小数组

# UAnimSequence/UAnimSequenceBase/UAnimationAsset
- 一个动画资源的层级关系
- 保存一个AnimNotify数组



# BodySetup
- 包含所有碰撞相关的资源信息
- 许多实例共享一个BodySetup
- 资源通过GetBodySetup，创建物理状态期间调用

# FBodyInstance
- 包含物理表现层所有信息
- 碰撞通道容器
- 碰撞响应
- BodySetup和UPrimitiveComponent 的弱指针
