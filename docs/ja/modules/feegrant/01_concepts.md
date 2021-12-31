# 概念

## 付与

`Grant`はKVStoreに保存され、完全なコンテキストでGrantを記録します。 すべての助成金には、「助成金」、「助成金」、およびどのような「助成金」が付与されるかが含まれます。 「グランター」とは、「グランティー」の取引手数料の一部または全部を支払うことを「グランティー」（受取人のアカウントアドレス）に許可しているアカウントアドレスです。`allowance`は、`grantee`に付与される料金手当（`BasicAllowance`または`PeriodicAllowance`、以下を参照）の種類を定義します。`allowance`は、`Any`タイプとしてエンコードされた`FeeAllowanceI`を実装するインターフェースを受け入れます。 「グランティー」と「グランター」に許可される既存の料金付与は1つだけであり、自己付与は許可されません。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L75-L81

`FeeAllowanceI`は次のようになります。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/x/feegrant/fees.go#L9-L32

## 手数料手当の種類

現在、2種類の手数料手当があります。

- `BasicAllowance`
- `PeriodicAllowance`

## BasicAllowance

`BasicAllowance`とは、`grantee`が`granter`の口座からの手数料を使用するための許可です。 「支出制限」または「有効期限」のいずれかが制限に達すると、助成金は州から削除されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L13-L26

- `spend_limit`は、`granter`アカウントから使用できるコインの制限です。 空の場合、支出制限がないと想定され、`grantee`は有効期限が切れる前に`granter`アカウントアドレスから利用可能なトークンをいくつでも使用できます。

- `expiration`は、この手当が期限切れになるオプションの時間を指定します。 値が空のままの場合、付与の有効期限はありません。

- `spend_limit`と`expiration`の値が空のグラントが作成された場合でも、それは有効なグラントです。`grantee`が`granter`からのトークンをいくつでも使用するように制限することはなく、有効期限もありません。 「グランティー」を制限する唯一の方法は、助成金を取り消すことです。

## PeriodicAllowance

`PeriodicAllowance`は、上記の期間の繰り返し料金の手当であり、助成金の有効期限が切れる時期と期間がリセットされる時期について言及することができます。 また、指定された期間に使用できるコインの最大数を定義することもできます。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L28-L73

- `basic`は`BasicAllowance`のインスタンスであり、定期的な料金の許容範囲ではオプションです。 空の場合、付与には`expiration`と`spend_limit`はありません。

- `period`は特定の期間であり、各期間が経過すると、`period_spend_limit`がリセットされます。

- `period_spend_limit`は、その期間に使用できるコインの最大数を指定します。

- `period_can_spend`は、period_reset時間の前に使用されるコインの数です。

- `period_reset`は、次の期間のリセットがいつ発生するかを追跡します。

## FeeGranter 标志

`feegrant`モジュールは、料金付与者とのトランザクションを実行するために、CLIに`FeeGranter`フラグを導入します。 このフラグが設定されている場合、`clientCtx`はCLIを介して生成されたトランザクションの付与者アカウントアドレスを追加します。

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/client/cmd.go#L224-L235

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/client/tx/tx.go#L120

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/x/auth/tx/builder.go#L268-L277

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/proto/cosmos/tx/v1beta1/tx.proto#L160-L181

cmdの例：

```Go
./simd tx gov submit-proposal --title="Test Proposal" --description="My Awesome Proposal" --type="Text" --from validator-key --fee-granter=cosmos1xh44hxt7spr67hqaa7nyx5gnutrz5fraw6grxn --chain-id =testnet --fees="10stake"
```

## 付与された料金控除

手数料は、`x/auth`アンティハンドラーの助成金から差し引かれます。 アンティハンドラーの動作の詳細については、[Auth Module AnteHandlers Guide]（../../ auth / spec / 03_antehandlers.md）を参照してください。

## Gas

DoS攻撃を防ぐために、フィルタリングされた`x/feegrant`を使用するとGasが発生します。 SDKは、`grantee`のトランザクションがすべて`granter`によって設定されたフィルターに準拠していることを保証する必要があります。 SDKは、フィルターで許可されたメッセージを繰り返し処理し、フィルター処理されたメッセージごとに10Gasを課金することでこれを行います。 次に、SDKは、「グランティー」によって送信されたメッセージを繰り返し処理して、メッセージがフィルターに準拠していることを確認し、メッセージごとに10Gasを課金します。 SDKは、フィルターに準拠していないメッセージを検出すると、反復を停止し、トランザクションを失敗させます。

**警告**：Gasは付与された許容量に対して請求されます。 アローワンスを使用してトランザクションを送信する前に、メッセージがフィルターに準拠していることを確認してください。