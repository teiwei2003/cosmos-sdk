# ADR 17:ヒストリカルタイトルモジュール

## 変更ログ

-2019年11月26日:最初のバージョンが始まります
-2019年12月2日:初版の最終ドラフト

## 環境

Cosmos SDKが[IBC仕様](https://github.com/cosmos/ics)を実装するには、Cosmos SDKのモジュールが最新のコンセンサス状態(バリデーターセットとコミットメントルート)を次のようにイントロスペクトできる必要があります。ハンドシェイク中にチェックする必要がある証明これらの値は他のチェーンにあります。

## 決定

アプリケーションは、最新の `n`ヘッダーを永続ストレージに保存する必要があります。 最初は、このストアは現在のMerklisedストアである可能性があります。 証明が必要ないため、Merklised以外のストレージを将来使用できます。

アプリケーションは、 `abci.RequestBeginBlock`を処理するときに新しいヘッダーを保存することにより、この情報をすぐに保存する必要があります。 

```golang
func BeginBlock(ctx sdk.Context, keeper HistoricalHeaderKeeper, req abci.RequestBeginBlock) abci.ResponseBeginBlock {
  info := HistoricalInfo{
    Header: ctx.BlockHeader(),
    ValSet: keeper.StakingKeeper.GetAllValidators(ctx),..note that this must be stored in a canonical order
  }
  keeper.SetHistoricalInfo(ctx, ctx.BlockHeight(), info)
  n := keeper.GetParamRecentHeadersToStore()
  keeper.PruneHistoricalInfo(ctx, ctx.BlockHeight() - n)
 ..continue handling request
}
```

または、アプリケーションはバリデーターセットのハッシュ値のみを保存できます。

アプリケーションは、これらの過去の `n`送信されたヘッダーを、` Keeper`の `GetHistoricalInfo`関数を介してCosmosSDKモジュールによるクエリに使用できるようにする必要があります。これは、新しいモジュールに実装することも、既存のモジュール(おそらく、 `x .stakeing`または` x .ibc`)に統合することもできます。

`n`はパラメータ保存パラメータとして設定できます。この場合、` ParameterChangeProposal`sで変更できますが、 `n`を追加すると、保存された情報に追いつくためにいくつかのブロックが必要になります。

## 状態

提案。

## 結果

このADRを実装するには、CosmosSDKを変更する必要があります。 Tendermintを変更する必要はありません。

### ポジティブ

-Cosmos SDKの任意の場所にあるモジュールを介して、最近の過去の高さのヘッダーと状態ルートを簡単に取得できます。
-TendermintにRPC呼び出しを行う必要はありません。
-ABCIの変更は必要ありません。

### ネガティブ

-Tendermintとアプリケーションの `n`ヘッダーデータをコピーします(追加のディスク使用量)-長期的には、[this](https://github.com/tendermint/tendermint/issues/4210)のような方法が望ましい場合があります。

### ニュートラル

(不明)

## 参照

-[ICS 2:「コンセンサス状態のイントロスペクション」](https://github.com/cosmos/ibc/tree/master/spec/core/ics-002-client-semantics#consensus-state-introspection) 