# ADR 046:モジュールパラメータ

## 変更ログ

-2021年9月22日:最初のドラフト

## 状態

提案

## 概要

このADRは、使用方法、相互作用方法、
そして、それぞれのパラメータを保存します。

## 環境

現在、Cosmos SDKでは、パラメーターを使用する必要のあるモジュールが使用されています
`x .params`モジュール。 `x .params`は、モジュールにパラメータを定義させることで機能します。
通常、単純な「Params」構造を介して、構造をに登録します
`x .params`モジュールは固有の` Subspace`を通過します
モジュールを登録します。次に、登録モジュールはそれぞれに一意にアクセスできます
`部分空間`。この `Subspace`を介して、モジュールはその` Params`を取得および設定できます。
構造。

さらに、CosmosSDKの `x .gov`モジュールは変更を直接サポートします
「ParamChangeProposal」ガバナンス提案タイプを介してチェーンにパラメーターを設定します。ここで、
利害関係者は、提案されたパラメーターの変更に投票できます。

`x .params`モジュールを使用して単一を管理します
モジュールパラメータ。言い換えれば、管理パラメータは本質的に「無料」です
開発者は、 `Params`構造、` Subspace`、および
`ParamSetPairs`などのさまざまな補助関数は` Params`タイプにあります。でも、
明らかな欠点がいくつかあります。これらの欠点には、パラメータが含まれます
非常に遅いJSONを介して状態でシリアル化します。さらに、パラメータ
`ParamChangeProposal`ガバナンス提案を通じて行われた変更は読み取ることができません
または国に書いてください。言い換えれば、
パラメータを変更しようとしたときのアプリケーションの状態遷移。

## 決定

`x .gov`と` x .authz`の配置に基づいて構築します
[#9810](https://github.com/cosmos/cosmos-sdk/pull/9810)。モジュール開発者
シリアル化する必要がある1つ以上の一意のパラメータデータ構造を作成します
声明。 Paramデータ構造は、 `sdk.Msg`インターフェースとそれぞれを実装する必要があります
Protobuf Msgサービスメソッド、すべてのパラメータを検証および更新します
必要な変更。 `x .gov`モジュールが渡されます
[#9810](https://github.com/cosmos/cosmos-sdk/pull/9810)、Paramがスケジュールされます
メッセージはProtobufMsgサービスによって処理されます。

パラメータを構造化する方法を決定するのは開発者次第であることに注意してください。
対応する `sdk.Msg`メッセージ。現在定義されているパラメータを検討してください
`x .auth`はパラメータ管理に` x .params`モジュールを使用します。 

```protobuf
message Params {
  uint64 max_memo_characters       = 1;
  uint64 tx_sig_limit              = 2;
  uint64 tx_size_cost_per_byte     = 3;
  uint64 sig_verify_cost_ed25519   = 4;
  uint64 sig_verify_cost_secp256k1 = 5;
}
```

开发人员可以选择为每个字段创建唯一的数据结构
`Params` 或者他们可以创建一个单一的 `Params` 结构，如上所述
“x/auth”的情况。

在前者中，`x/params` 方法，需要为每个单独的创建一个 `sdk.Msg`
字段以及处理程序。 如果有很多，这可能会成为负担
参数字段。 在后一种情况下，只有一个数据结构和
因此只有一个消息处理程序，但是，消息处理程序可能必须是
更复杂，因为它可能需要了解正在使用的参数
更改与未更改的参数。

参数更改建议是使用 `x/gov` 模块提出的。 执行是通过
`x/authz` 授权给根 `x/gov` 模块的帐户。

继续使用`x/auth`，我们演示一个更完整的例子: 
```go
type Params struct {
	MaxMemoCharacters      uint64
	TxSigLimit             uint64
	TxSizeCostPerByte      uint64
	SigVerifyCostED25519   uint64
	SigVerifyCostSecp256k1 uint64
}

type MsgUpdateParams struct {
	MaxMemoCharacters      uint64
	TxSigLimit             uint64
	TxSizeCostPerByte      uint64
	SigVerifyCostED25519   uint64
	SigVerifyCostSecp256k1 uint64
}

type MsgUpdateParamsResponse struct {}

func (ms msgServer) UpdateParams(goCtx context.Context, msg *types.MsgUpdateParams) (*types.MsgUpdateParamsResponse, error) {
  ctx := sdk.UnwrapSDKContext(goCtx)

 ..verification logic...

 ..persist params
  params := ParamsFromMsg(msg)
  ms.SaveParams(ctx, params)

  return &types.MsgUpdateParamsResponse{}, nil
}

func ParamsFromMsg(msg *types.MsgUpdateParams) Params {
 .....
}
```

gRPCの `Service`クエリも提供する必要があります。次に例を示します。

```protobuf
service Query {
 .....
  
  rpc Params(QueryParamsRequest) returns (QueryParamsResponse) {
    option (google.api.http).get = "/cosmos/<module>/v1beta1/params";
  }
}

message QueryParamsResponse {
  Params params = 1 [(gogoproto.nullable) = false];
}
```

## 結果

モジュールパラメータメソッドを実装した結果、
モジュールパラメータの変更をステートフルで拡張可能にして、ほぼすべてに適応できるようにします
アプリケーションのユースケース。イベントを発行する(そして登録されたフックをトリガーする)ことができるようになります
[偶数フック](https://github.com/cosmos/cosmos-sdk/discussions/9656))で提案された作業のイベントを使用します。
他のメッセージサービスメソッドを呼び出すか、移行を実行します。
さらに、読書のパフォーマンスが大幅に向上します
そして、特に特定のパラメータのセットの場合、状態との間でパラメータを書き込みます
一貫して読んでください。

ただし、このアプローチでは、開発者はより多くのタイプを実装する必要があり、
パラメータが多いと、メッセージサービスの方法が煩雑になる場合があります。また、
開発者は、モジュールパラメータの永続ロジックを実装する必要があります。
ただし、これは些細なことです。

### 下位互換性

モジュールパラメータを処理する新しい方法は、当然、逆行しません。
既存の `x .params`モジュールと互換性があります。ただし、 `x .params`は
Cosmos SDKに保持し、非推奨としてマークされ、追加されなくなります
潜在的なバグ修正に加えて、機能も追加されました。 `x .params`に注意してください
モジュールは、将来のバージョンで完全に削除される可能性があります。
### ポジティブ

-モジュールパラメータをより効率的にシリアル化する
-モジュールはパラメータの変更に反応し、他の操作を実行できます。
-特別なイベントを発行して、フックをトリガーすることができます。
### ネガティブ

-モジュールパラメータは、モジュール開発者にとって少し負担になります。
     -モジュールは、パラメータ状態の永続化と取得を担当するようになりました
     -モジュールには、パラメーターを処理するための一意のメッセージハンドラーが必要です。
       各固有のパラメーターのデータ構造の変更。

### ニュートラル

-[#9810](https://github.com/cosmos/cosmos-sdk/pull/9810)を確認する必要があります
  とマージされました。

<！-## さらなる議論

ADRがドラフトまたは提案段階にある間、このセクションには、将来の反復で解決される問題の要約が含まれている必要があります(通常はプルリクエストディスカッションからのコメントを参照します)。
後で、このセクションでは、このADRの分析中に作成者またはレビュー担当者が見つけたアイデアや改善点をオプションで一覧表示できます。->

## 参照

-https://github.com/cosmos/cosmos-sdk/pull/9810
-https://github.com/cosmos/cosmos-sdk/issues/9438
-https://github.com/cosmos/cosmos-sdk/discussions/9913 