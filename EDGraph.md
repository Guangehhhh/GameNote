## Editor
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
