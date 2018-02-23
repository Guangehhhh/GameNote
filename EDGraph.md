## Editor
- FEditorUndoClient

- UEdGraph
  - 包含一个语法GraphSchema
  -
- UEdGraphNode

- FEdGraphSchemaActio

- UEdGraphSchema
  - 获取所有可以执行的操作，如右键单击图形或从图形上拖动释放图形
  -
- SGraphPin

- SGraphNode

- IDetailCustomization

- IDetailLayoutBuilder

  - FAssetTypeActions_MyQuest::OpenAssetEditor
    - FMyQuestEditorModule::CreateMyQuestEditor
      - FMyQuestEditor::InitMyQuestAssetEditor
        - FTabManager::NewLayout
          - FAssetEditorToolkit::InitAssetEditor
            - FToolkitManager::RegisterNewToolkit
              - ToolkitHost->OnToolkitHostingStarted( NewToolkit );
              - FMyQuestEditor::RegisterTabSpawners
            - SetupInitialContent
              - RestoreFromLayout(DefaultLayout);
                - RestoreArea(ThisArea, ParentWindow, bEmbedTitleAreaContent);
                  - RestoreSplitterContent
                    - FTabManager::SpawnTab
                      - SpawnTab_UpdateGraph
                        - MyGraph->Initialize();
                        - CreateGraphEditorWidget(QuestAsset->EdGraph)
	            - GenerateMenus(bCreateDefaultStandaloneMenu);
          - FModuleManager::LoadModuleChecked<FMyQuestEditorModule>("MyQuestEditor");
          - AddMenuExtender
          - BindCommands();
	        - ExtendToolbar();
          - RegenerateMenusAndToolbars();


# 放置
  - FAISchemaAction_NewNode::PerformAction  
    - UAIGraphNode::PostPlacedNewNode
      - NodeInstance = NewObject<UObject>(GraphOwner, NodeClass);
			- NodeInstance->SetFlags(RF_Transactional);
			- InitializeInstance();
  - FGraphNodeClassData 表示一个类

##链接节点
- Node->NodeConnectionListChanged();
