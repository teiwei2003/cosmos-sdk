# 開始-ブロック

各abciがブロック呼び出しを開始すると、履歴情報が保存され、プルーニングされます
`HistoricalEntries`パラメータによると。

## 履歴情報の追跡

`HistoricalEntries`パラメータが0の場合、` BeginBlock`はno-opを実行します。

それ以外の場合、最新の履歴情報はキー `historicalInfoKey | height`の下に保存され、` height --HistoricalEntries`より古いエントリは削除されます。
ほとんどの場合、これにより、ブロックごとに1つのエントリがプルーニングされます。
ただし、パラメータ `HistoricalEntries`が低い値に変更された場合、プルーニングする必要のある複数のエントリがストアに存在します。