# DataTable
- 用户只是自定义了 Raw的种类 核心逻辑在DataTable
- FindRow
- FindRowUnchecked(RowName)
  - 内部从RowMap调用Find(RowName)
  - 返回uint8*类型
  - 如果uint8*不为空
    - 表示找到确定位置
  - 读取Table的RowStruct类型为USciptStruct*
  - RowStruct调用CopyScriptStruct
