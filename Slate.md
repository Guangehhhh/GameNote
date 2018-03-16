# Slate
- #define SNew( WidgetType, ... ) \
	MakeTDecl<WidgetType>( #WidgetType, __FILE__, __LINE__, RequiredArgs::MakeRequiredArgs(__VA_ARGS__) ) <<= TYPENAME_OUTSIDE_TEMPLATE WidgetType::FArguments()

- LeafWidget
  - 不可再分的单元；组件层次的最下层次节点；直接继承自SWidget；没有child Slot
  - 负责最基础的单元信息显示
  - 目前的LeafWidget有
		- STextBlock, SImage, SColorBlock, SCircularThrobber, SEditableText,
		- SColorWheel, SSlider,SSpacer, SProgressBar...

- CompoundWidget
  - widgets with a fixed number of explicitly named child slots定义稍微复杂的组件
  - 例如：SButton、SBorder、SCheckBox等
  - CompoundWidget内容可根据自己的需求添加 , 可以是很复杂的Page Widget也可是简单实用的控件，- 在Construct里面添加子节点
  - 如SButton中包含Sborder、STextBlock
  - 内部layout和width、height的控制

- SPanel
  - 一些SCompoundWidget可以负责布局，当然由Slate层暴露给上层使用；
    - 例如SOverLay，SBox， SBorder等
  - SOverlay：有Z-order属性可设置显示层级
  - SBox ：支持padding，widthoverride，heightoverride
    - 是目前发现唯一设置绝对长宽数据的组件（起效需要不跟其它的相对布局参数冲突）

  - SVerticalBox纵向布局的box ,设置width fill和auto方法；
  - SHorizontalBox横向布局的box, 设置height fill和auto方法、
  - 用Align的方式进行定位HAligh、VAlign

  - SGridePanel:可分格进行widget 布局
  - Z-order控制控件的分层显示（SOverlay组件）

  -  除此之外SPanel的派生类l是一个可以控制Slot的布局组件
    - 布局规则更加自由；以上的组件布局规则相对来说局限性较大
    - SPanel则使用更加自由的锚点+盒子模型
  - 相对布局的布局模型 与css的盒子模型类似
  - Padding对子控件的slot进行相对定位
  - 用Align的方式进行对齐定位HAligh、VAlign
  - 对child slot的Width和Height的控制 Fill方式和auto方式
  - Z-order控制控件的分层显示（SOverlay组件）

- RichTextBlock
- WeakWidget

- SNew
	- SNew创建一个widget对象，要加入DisplayObjectList才能绘制
- SASSIGNNEW操作符
	- SAssignNew创建一个widget对象并保存该对象的智能指针


# Widget 创建和绘制
- 关于DisplayObjectList，Swidget有Children列表，要加入到children列表中才能被绘制
- 布局类控件使用addSlot、removeSlot添加插槽的方式添加child
  - 非布局类控件使用ContentSlot的方式添加内容
  - 只能添加一个组件
  - 写成ChildSlot[]的形式绘制OnPaint
- LeafWidget在Onpaint方法中用DrawElement绘制基础的组件
- SCompoundWdiget 既可以在Onpaint中绘制需要的基础图形也可以在Construct中添加childContent
布局的控件找出arrangedchildren，每个子组件自己负责Paint

---
# 绘制 UI的过程分三个部分：
- 第一部分主要在SlateApplication中PrivateDrawWindow
	- 主线程Tick驱动控件不断递归调用Paint和Onpaint搜集DrawElement，存入windowelementlist
- 第二部分 SlateRHIrender中调用FSlateRHIRenderer::DrawWindows_Private
 	- 处理windowelementlist中的DrawElement；
	- 使用FSlateElementBatcher把DrawElement转化成FSlateBatchData批次数据
		- 具体调用在FSlateElementBatcher::AddElements中；
	- 针对layer优化的代码和合并drawcall的代码在这部分代码中
- 第三部分 全渲染线程 使用上一步生成的FSlateBatchData生成renderbatch数据
	- 然后送进渲染的pipeline	在FSlateRHIRenderer::DrawWindow_RenderThread接口中
- 合并的过程发生在mainthread ，FSlateApplication执行DrawWindows的接口调用SlateRHIRender做RHI的处理的时候
	- 算是第二个过程发生的时候。详细过程如下：





# Slate Event（事件）
- SLATE_EVENT( FOnClicked, OnClicked )
- DECLARE_DELEGATE_RetVal( FReply, FOnClicked )
- 基础事件监听 由SlateApplication调用
- 传入响应函数
  - .OnClicked(UIHandler.Pin().ToSharedRef(),&FLoginPage::OnClickLoginBtn)


# UI事件处理的过程
  - FWindowsApplication在AppWndProc处接受win32传进的message
  - AppWndProc根据message的类型对message分类处理然后执行deferMessage

  - deferMessage检查消息是否需要延迟处理
    - 延迟处理则放入处理listDeferredMessages
    - 否则立即执行ProcessDeferredMessage

  - ProcessDeferredMessage根据msg类型调用MessageHandler的不同方法开始处理事件
    - 关  于FWindowsApplication中的MessageHandler其实就是一个FSlateApplication
    - 如鼠标左键mouseDown的处理，则执行
    - MessageHandler->OnMouseDown( CurrentNativeEventWindowPtr, EMouseButtons::Left );

  - FSlateApplication作为FWindowsApplication的MessageHandler开始处理具体的事件；
    - 转化msg为 FPointEvent 然后进入ProcessMessage的处理
    - 如OnMouseDown 执行ProcessMouseButtonDownMessage

  - FSlateApplication 在ProcessMessage过程的处理根据不同的事件不同
    - 如鼠标单击事件 会使用LocateWindowUnderMouse方法找出当前FPointEvent点上的所有widgetPath
    - 然后对于每个widget path 里的widget生成FReply响应，
    - 并执行Widget里面对应的处理事件的方法
    - 如鼠标左键单击的事件会执行 最后开始进行ProcessReply的过程

  - ProcessReply的过程主要 Apply any requests form the Reply to the application

- 数据更新
  - SWidget在Construct的过程会对所有的dynamic的属性绑定delegate
  - delegate在Tick的时候执行计算返回数据
  - 示例：.ColorAndOpacity(this,&SShooterMenuItem::GetButtonTextColor)

- 组件生命周期
  - 构造（SNew或SAssingNew的时候执行）
  - added to parent
  - 进入正常的使用阶段（Visible的使用Visible、Hidden、MouseHidden等参数）
  - remove from parent
  - destroy
  - 检查remove from DisplayObjectList后Tick和Onpaint方式的执行？（不再执行Visibility同样）

# Slate的Animation机制
- Animation 机制 使用 construct调用的方法 tick调用 修改属性
  - 另有 FCurveSequence 进行计算差值
- Construct 中定义样式
  - .Padding(TAttribute<FMargin>(this,&SShooterMenuWidget::GetLeftMenuOffset))  
- 定义FCurveSequence 来计算差
  - LeftMenuScrollOutCurve = LeftMenuWidgetAnimation.AddCurve(0,MenuChangeDuration,ECurveEaseFunction::QuadInOut)
	- LeftMenuWidgetAnimation.Play();
- Tick时修改属性值
  - FMargin SShooterMenuWidget::GetLeftMenuOffset() const
	   {
		      const float LeftBoxSizeX = LeftBox-	>GetDesiredSize().X + OutlineWidth * 2;
		      return FMargin(0, 0,-LeftBoxSizeX + 	LeftMenuScrollOutCurve.GetLerp() * 	LeftBoxSizeX,0);
	   }
- 对Animation状态的检查：
  - 一般在Tick方法中访问animation的状态
  - IsAtEnd
  - ISAtStart
  - IsForward
  - IsInReverse
  - IsPlaying
