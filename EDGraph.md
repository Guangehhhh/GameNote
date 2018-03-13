## UEdGraph
* Editor系统是由三个部分组成：
    * 蓝图编辑系统
    * 蓝图本身
    * 蓝图编译后的字节码
* 蓝图字节码将不会包含蓝图本身的节点信息。这部分信息是在UEdGraph 中存储的，这也是一种优化。
* UEdGraph
|-Schema
|-Nodes
|-SubGraphes

* Schema
* 语法规定了当前蓝图能够产生什么样的节点等信息
  * 定义了自己的Schema 之后，通过重载对应的函数即可实现语法的规定
* GetContextMenuActions定义在当前蓝图编辑器中右键菜单的菜单项
  * 通过向FGraphContextMenuBuilder 引用中填充内容，实现对右键菜单的定制。
* GetGraphContextActions ，定义的是右键菜单和拖曳连线之后弹出的菜单共用的菜单项。
* CanCreateConnection 该函数传入两个UEdGraphPin，判断是否能够建立一条连线。
  * 通过构造一个FPinConnection-Response 作为返回


- FAIGraphEditor : public FEditorUndoClient
  - 包含Spawn各种SWidget函数
  - 创建Graph
- UEdGraph
  - 包含一个语法GraphSchema 和 Node数组
  - UEdGraphNode

  - UEdGraphSchema
    - 获取所有可以执行的操作，如右键单击图形或从图形上拖动释放图形


- SGraphPin
- SGraphNode

- IDetailCustomization
- IDetailLayoutBuilder


## 从Asset打开编辑器流程
- 语言描述
- FAssetTypeActions_BehaviorTree::OpenAssetEditor
  - FAssetEditorManager::Get().FindEditorForAsset
  - InitBehaviorTreeEditor
    - InitAssetEditor
    - BindCommonCommands();
		- ExtendMenu();
		- CreateInternalWidgets();

# Dubug
- 参照 FBehaviorTreeDebugger

# 放置
  - FAISchemaAction_NewNode::PerformAction  
    - UAIGraphNode::PostPlacedNewNode
      - NodeInstance = NewObject<UObject>(GraphOwner, NodeClass);
			- NodeInstance->SetFlags(RF_Transactional);
			- InitializeInstance();
  - FGraphNodeClassData 表示一个类

##链接节点
- Node->NodeConnectionListChanged();
