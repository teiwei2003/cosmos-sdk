# 开始块

每个 abci 开始块调用，历史信息将被存储和修剪
根据`HistoricalEntries`参数。

## 历史信息跟踪

如果`HistoricalEntries` 参数为0，则`BeginBlock` 执行无操作。

否则，最新的历史信息存储在键“historicalInfoKey|height”下，而任何早于“height - HistoricalEntries”的条目都将被删除。
在大多数情况下，这会导致每个块修剪单个条目。
但是，如果参数“HistoricalEntries”已更改为较低的值，则存储中将有多个必须修剪的条目。 