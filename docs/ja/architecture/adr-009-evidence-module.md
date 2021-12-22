# ADR 009:証拠モジュール

## 変更ログ

-2019年7月31日:最初のドラフト
-2019年10月24日:初期実装

## 状態

受け入れられました

## 環境

安全性が高く、堅牢で相互運用可能なブロックチェーンの構築をサポートするため
アプリケーション、CosmosSDKがメカニズムを公開することは非常に重要です。
証拠を提出、評価、検証して、コンセンサスに達することができます
あいまいさ(二重投票)など、検証者による不正行為に対する罰則、
バインドされていない場合の署名、署名が正しくない場合の状態遷移(将来)など。
さらに、このメカニズムはすべての人に効果的です
[IBC](https://github.com/cosmos/ics/blob/master/ibc/2_IBC_ARCHITECTURE.md)または
機能をサポートするためのクロスチェーン検証プロトコルの実装
住宅ローンチェーンからメインチェーンに不適切な動作を渡します
あいまいなバリデーターをカットできるようにチェーンします。

## 決定

次の機能をサポートする証拠モジュールをCosmosSDKに実装します
特徴:

-定義に必要な抽象化とインターフェースを開発者に提供します
  証拠メッセージ、メッセージ処理手順、および削減と罰の方法をカスタマイズします
  それに対応して、それは不適切な振る舞いです。
-証​​拠メッセージを任意のモジュールのハンドラーにルーティングする機能をサポートします
  提出された違法行為の正当性を判断します。
-ガバナンスサポートを通じて罰を変更する機能
  証拠の種類。
-クエリパラメータ、エビデンスタイプ、パラメータ、および
  提出されたすべての有効な違法行為。

### タイプ

まず、 `Evidence`インターフェースタイプを定義します。 `x/evidence`モジュールを実装できます
独自のタイプは、多くのチェーンで使用できます(例: `CounterFactualEvidence`)。
さらに、他のモジュールも同様の方法で独自の「証拠」タイプを実装できます。
スケーラブルな方法でのガバナンス。特定のことに注意を払うことが重要です
`Evidence`インターフェースを実装するタイプには、たとえば任意のフィールドが含まれる場合があります
違反時間。 `Evidence`タイプは可能な限り柔軟なままにしておく必要があります。

`x/evidence`モジュールに証拠を提出するときは、特定のタイプを提供する必要があります
検証者のコンセンサスアドレス。これは `x/slashing`で認識されている必要があります
モジュール(違反が有効であると仮定)、違反の高さ
発生し、検証者のパワーは違反が発生したのと同じ高さになります。 

```go
type Evidence interface {
  Route() string
  Type() string
  String() string
  Hash() HexBytes
  ValidateBasic() error

./The consensus address of the malicious validator at time of infraction
  GetConsensusAddress() ConsAddress

./Height at which the infraction occurred
  GetHeight() int64

./The total power of the malicious validator at time of infraction
  GetValidatorPower() int64

./The total validator set power at time of infraction
  GetTotalPower() int64
}
```

### ルーティングと処理

各「証拠」タイプは、特定の一意のルートにマッピングされ、登録されている必要があります
`x/evidence`モジュール。 これは、 `Router`の実装を通じて実現されます。  

```go
type Router interface {
  AddRoute(r string, h Handler) Router
  HasRoute(r string) bool
  GetRoute(path string) Handler
  Seal()
}
```

`x/evidence`モジュールを正常にルーティングした後、` Evidence`タイプ
`Handler`によって渡されます。 この `ハンドラー`はすべてを実行する責任があります
証拠が有効であることを確認するために必要な対応するビジネスロジック。 存在
さらに、 `ハンドラー`は必要なカットと潜在的な投獄を実行することができます。
スラッシュスコアは通常、何らかの形の静的関数によって生成されるため、
`Handler`にそうすることを許可することは、最大の柔軟性を提供します。 例はできます
`k * Estate.GetValidatorPower()`です。ここで、 `k`は制御されたチェーンパラメータです。
ガバナンスを通じて。 `Evidence`タイプは、すべての外部情報を提供する必要があります
これは、「処理プログラム」が必要な状態遷移を実行するために必要です。
エラーが返されない場合、「証拠」は有効であると見なされます。 

```go
type Handler func(Context, Evidence) error
```

### Submission

`Evidence`は、内部の` MsgSubmitEvidence`メッセージタイプを介して送信されます
これは、 `x/evidence`モジュールの` SubmitEvidence`によって処理されます。 

```go
type MsgSubmitEvidence struct {
  Evidence
}

func handleMsgSubmitEvidence(ctx Context, keeper Keeper, msg MsgSubmitEvidence) Result {
  if err := keeper.SubmitEvidence(ctx, msg.Evidence); err != nil {
    return err.Result()
  }

./emit events...

  return Result{
  ./...
  }
}
```

`x/evidence`モジュールのキーパーは、` Evidence`をと接続する責任があります
モジュールのルーターであり、対応する `Handler`を呼び出します。
バリデーターをカットして投獄します。 成功後、提出された証拠は保持されます。 

```go
func (k Keeper) SubmitEvidence(ctx Context, evidence Evidence) error {
  handler := keeper.router.GetRoute(evidence.Route())
  if err := handler(ctx, evidence); err != nil {
    return ErrInvalidEvidence(keeper.codespace, err)
  }

  keeper.setEvidence(ctx, evidence)
  return nil
}
```

### 創世記

最後に、 `x/evidence`モジュールの作成状態を表す必要があります。 この
モジュールには、送信されたすべての有効な違反と必要なパラメーターのリストのみが必要です。
モジュールは提出された証拠を処理する必要があります。 `x/証拠`
モジュールは、最も必要なローカル証拠のタイプを自然に定義してルーティングします
ペナルティ定数を減らす必要があるかもしれません。 

```go
type GenesisState struct {
  Params       Params
  Infractions  []Evidence
}
```

## 結果

### ポジティブ

-ステートマシンがチェーンに送信された不適切な動作を処理および罰することを許可します
    合意された削減パラメータに基づくバリデーター。
-任意のモジュールが証拠タイプを定義および処理できるようにします。 これにより、さらに
    削減と投獄は、より複雑なメカニズムによって定義されます。
-証​​拠の提出をテンダーミントだけに頼らないでください。
### ネガティブ

-リアルタイムのオンチェーンガバナンスを通じて新しいタイプの証拠を導入することは容易ではありません
    新しいタイプの証拠に対応する手順を導入できないため

### ニュートラル

-私たちは無期限に違反を続けるべきですか？ それとも、イベントにもっと依存する必要がありますか？

## 参照

-[ICS](https://github.com/cosmos/ics)
-[IBCアーキテクチャ](https://github.com/cosmos/ics/blob/master/ibc/1_IBC_ARCHITECTURE.md)
-[テンダーミントフォークの説明責任](https://github.com/tendermint/spec/blob/7b3138e69490f410768d9b1ffc7a17abc23ea397/spec/consensus/fork-accountability.md) 