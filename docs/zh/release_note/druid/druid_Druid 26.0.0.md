# druid Druid 26.0.0

Apache Druid 26.0.0 包含超过 390 个新功能、错误修复、性能增强、文档改进和额外的测试覆盖。

[查看完整的更改集以获取更多详细信息](https://github.com/apache/druid/pulls?q=is%3Apr+milestone%3A26.0+is%3Aclosed)。

在升级到 Druid 26.0.0 之前，请查看[升级说明和不兼容更改](#26.0.0-upgrade-notes-and-incompatible-changes)。

# 亮点

## 自动类型列模式（实验性）

在本地摄取中添加了新的“auto”类型列模式和索引器，这是嵌套列功能的下一逻辑迭代。这个自动类型列索引器会根据给定的输入生成最合适的列，生成的列可以是 `STRING`、`ARRAY<STRING>`、`LONG`、`ARRAY<LONG>`、`DOUBLE`、`ARRAY<DOUBLE>` 或 `COMPLEX<json>` 列，所有这些列都共享一个通用的“嵌套”格式。

所有由“auto”生成的列都有索引以帮助快速过滤（与经典的 `LONG` 和 `DOUBLE` 列不同），并使用基于基数的阈值尝试仅在可能实际加速查询时使用这些索引（与经典的 `STRING` 列不同）。

由此“auto”索引器生成的 `COMPLEX<json>` 列以不同于其“json”（v4）对应物的方式存储简单标量类型的数组，将它们存储为 ARRAY 类型的列。这意味着 `JSON_VALUE` 函数现在可以提取整个数组，例如 `JSON_VALUE(nested, '$.array' RETURNING BIGINT ARRAY)`。目前，复杂对象数组的存储方式没有变化。

此改进还为 Druid 添加了全新的功能，即 `ARRAY` 类型列，与经典的多值 `STRING` 列不同，这些列具有 ARRAY 语义。这些列目前只能通过“auto”类型索引器创建，当所有值都是具有相同类型元素的数组时。

数组数据类型是一种允许在数据库表的单列中存储多个值的数据类型。数组通常用于存储相关数据集，可以作为一个组轻松访问和操作。

此版本添加了对存储原始值数组（如 `ARRAY<STRING>`、`ARRAY<LONG>` 和 `ARRAY<DOUBLE>`）的支持，将它们作为专门的嵌套列存储，而不是将它们分解为单独的元素列。

这些更改影响了 26.0 中的两个额外新功能：模式自动发现和解嵌。

### 模式自动发现（实验性）

我们正在向 Druid 添加带有类型推断的模式自动发现功能。使用此功能，当模式可用时，会检测到每个传入字段的数据类型。对于可能包含添加、删除或更改字段的传入数据，您可以选择拒绝不符合的数据（“数据库始终正确 - 拒绝错误数据！”），或者让模式自动发现更改数据源以匹配传入数据（“数据始终正确 - 更改数据库！”）。

模式自动发现推荐用于新的用例和摄取。对于现有用例，请谨慎切换到模式自动发现，因为 Druid 会将类似数组的值（例如 `["tag1", "tag2"]`）摄取为 `ARRAY<STRING>` 类型列，而不是多值（MV）字符串，这可能会导致依赖 MV 行为的下游应用程序出现问题。在官方迁移路径可用之前，请暂缓切换。

要使用此功能，请在任务或主管规范中将 `spec.dataSchema.dimensionsSpec.useSchemaDiscovery` 设置为 `true`，或者如果使用控制台中的数据加载器，请取消选中“显式定义模式”切换开关。Druid 可以推断整个模式或部分模式，如果您在维度列表中显式列出维度。

模式自动发现适用于本地批处理和流式摄取。

## 解嵌数组（实验性）

解嵌的一个有趣之处在于它允许对数组数据类型进行更广泛的操作。您可以使用 UNNEST 函数（SQL）或 `unnest` 数据源（本地）解嵌数组。

解嵌将嵌套数组或表转换为单个行。UNNEST 函数在处理包含嵌套数组的复杂数据类型（如 JSON）时特别有用。

例如，假设您有一个名为“orders”的表，其中包含一个名为“items”的列，该列包含每个订单的产品数组。您可以使用解嵌提取单个产品（“each_item”），如下 SQL 示例所示：

```sql
SELECT order_id, each_item FROM orders, UNNEST(items) AS unnested(each_item)
```

这会生成一个结果集，每个订单中的每个项目都有一行，包含订单 ID 和单个项目的列。

请注意左表/数据源（示例中的 `orders`）后的逗号。这是必需的。

## MSQ 的排序合并连接和哈希洗牌连接

我们现在可以通过将上下文参数 `sqlJoinAlgorithm` 设置为 `sortMerge` 来执行洗牌连接，或者省略它以执行广播连接（默认）。

多阶段查询可以使用排序合并连接算法。使用此算法，每个成对连接被规划到其自己的阶段，具有两个输入。这种方法通常性能较低但可扩展性更高。

将上下文参数 `sqlJoinAlgorithm` 设置为 `sortMerge` 以使用此方法。

广播哈希连接类似于本地连接查询的执行方式。

## 字典压缩的存储改进

切换到使用前编码字典压缩（实验性）可以节省高达 30% 的存储空间，对查询性能几乎没有影响。

此版本进一步改进了 `indexSpec` 上的 `frontCoded` 类型的 `stringEncodingStrategy`，具有新的段格式版本，通常具有更快的读取速度和更小的段大小。此改进与 Druid 25.0 不兼容。添加了一个新的 `formatVersion` 选项，默认值为当前版本 `0`。将 `formatVersion` 设置为 `1` 以开始使用新版本。

此外，整体存储大小，特别是使用更大桶时，得到了改进。

# 其他功能和改进

## MSQ 任务引擎

### SQL 查询的数组值参数

添加了对 SQL 查询的数组值参数的支持。您现在可以重用相同的 SQL 进行每次摄取，只需将不同的输入文件集作为查询参数传递。

### EXTERN 函数的 EXTEND 子句

您现在可以使用 EXTEND 子句以标准 SQL 格式提供源数据的列定义列表。

Web 控制台现在默认使用 EXTEND 子句语法生成的所有查询。这意味着 Druid 26 中由 Web 控制台生成的基于 SQL 的摄取语句（例如来自基于 SQL 的数据加载器）在早期版本的 Druid 中将无法工作。

### MSQ 容错

添加了 MSQ 控制器任务在失败情况下重试工作任务的能力。要启用，请在查询上下文中传递 `faultTolerance:true`。
```