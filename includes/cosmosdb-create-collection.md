现在可以使用数据资源管理器创建一个集合，并将数据添加到数据库。 

1. 在 Azure 门户的导航菜单中，单击“数据资源管理器”。 
2. 在“数据资源管理器”边栏选项卡中，单击“新建集合”，然后使用以下信息填写该页。

    ![Azure 门户中的数据资源管理器](./media/cosmosdb-create-collection/azure-cosmosdb-data-explorer.png)

    设置|建议的值|说明
    ---|---|---
    数据库 ID|Items|新数据库的 ID。 数据库名称的长度必须为 1 到 255 个字符，不能包含 `/ \ # ?` 或尾随空格。
    集合 ID|ToDoList|新集合的 ID。 集合名称与数据库 ID 的字符要求相同。
    存储容量| 固定 (10 GB)|保留默认值。 这是数据库的存储容量。
    吞吐量|400 RU|保留默认值。 如果想要减少延迟，以后可以增加吞吐量。
    分区键|/userid|一个分区键，用于将数据均匀分配到每个分区。 选择正确的分区键对于创建高性能集合而言很重要，请在[设计分区](../articles/cosmos-db/partition-data.md#designing-for-partitioning)中了解更多相关信息。    



3. 填写表单后，请单击“确定”。