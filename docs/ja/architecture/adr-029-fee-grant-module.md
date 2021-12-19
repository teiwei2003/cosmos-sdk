# ADR 029:経費補助モジュール

## 変更ログ

-2020/08/18:最初のドラフト
-2021 .05/05:高さベースの有効期限のサポートを削除し、名前を簡略化しました。

## 状態

受け入れられました

## 環境

ブロックチェーン取引を行うには、署名口座に十分な正しい金種残高が必要です
それを支払うために。十分な手数料でウォレットを維持する必要があるトランザクションにはいくつかの種類があります。
採用の障壁。

たとえば、適切な権限が設定されている場合、誰かが提案に投票する機能を一時的に委任することがあります。
電話に保存されている「バーナー」アカウントのセキュリティは最小限です。

他のユースケースには、サプライチェーン内のアイテムを追跡する労働者や分析のためにフィールドデータを提出する農家が含まれます
またはコンプライアンスの目的。

これらすべてのユースケースで、これらのアカウントの必要性を排除することにより、ユーザーエクスペリエンスが大幅に向上します
適切なコストバランスを維持します。これは、企業で何かを採用したい場合に特に当てはまります。
サプライチェーンの追跡など。

1つの解決策は、適切な料金でこれらのアカウントを自動的に埋めるサービスを提供することですが、より優れたユーザーエクスペリエンス
これは、これらのアカウントが適切な支出制限のある公費プールアカウントから引き出されることを許可することによって提供されます。
単一のプールは、多数の小さな「フィルイン」トランザクションの解約を減らし、レバレッジをより効果的に利用します
プールを設定した組織のリソース。

## 決定

解決策として、1つのアカウント、つまり「grant」が別のアカウント、つまり「grant」を付与できるようにするモジュール `x .feegrant`を提案しました。
明確に定義された特定の制限内の費用引当金については、付与者のアカウント残高を使用します。

料金制限は、拡張可能な `FeeAllowanceI`インターフェースによって定義されます。 

```go
type FeeAllowanceI {
 ..Accept can use fee payment requested as well as timestamp of the current block
 ..to determine whether or not to process this. This is checked in
 ..Keeper.UseGrantedFees and the return values should match how it is handled there.
 ./
 ..If it returns an error, the fee payment is rejected, otherwise it is accepted.
 ..The FeeAllowance implementation is expected to update it's internal state
 ..and will be saved again after an acceptance.
 ./
 ..If remove is true (regardless of the error), the FeeAllowance will be deleted from storage
 ..(eg. when it is used up). (See call to RevokeFeeAllowance in Keeper.UseGrantedFees)
  Accept(ctx sdk.Context, fee sdk.Coins, msgs []sdk.Msg) (remove bool, err error)

 ..ValidateBasic should evaluate this FeeAllowance for internal consistency.
 ..Don't allow negative amounts, or negative periods for example.
  ValidateBasic() error
}
```

Two basic fee allowance types, `BasicAllowance` and `PeriodicAllowance` are defined to support known use cases:

```proto
/.BasicAllowance implements FeeAllowanceI with a one-time grant of tokens
/.that optionally expires. The delegatee can use up to SpendLimit to cover fees.
message BasicAllowance {
 ..spend_limit specifies the maximum amount of tokens that can be spent
 ..by this allowance and will be updated as tokens are spent. If it is
 ..empty, there is no spend limit and any amount of coins can be spent.
  repeated cosmos_sdk.v1.Coin spend_limit = 1;

 ..expiration specifies an optional time when this allowance expires
  google.protobuf.Timestamp expiration = 2;
}

/.PeriodicAllowance extends FeeAllowanceI to allow for both a maximum cap,
/.as well as a limit per time period.
message PeriodicAllowance {
  BasicAllowance basic = 1;

 ..period specifies the time duration in which period_spend_limit coins can
 ..be spent before that allowance is reset
  google.protobuf.Duration period = 2;

 ..period_spend_limit specifies the maximum number of coins that can be spent
 ..in the period
  repeated cosmos_sdk.v1.Coin period_spend_limit = 3;

 ..period_can_spend is the number of coins left to be spent before the period_reset time
  repeated cosmos_sdk.v1.Coin period_can_spend = 4;

 ..period_reset is the time at which this period resets and a new one begins,
 ..it is calculated from the start time of the first transaction after the
 ..last period ended
  google.protobuf.Timestamp period_reset = 5;
}

```

Allowances can be granted and revoked using `MsgGrantAllowance` and `MsgRevokeAllowance`:

```proto
/.MsgGrantAllowance adds permission for Grantee to spend up to Allowance
/.of fees from the account of Granter.
message MsgGrantAllowance {
     string granter = 1;
     string grantee = 2;
     google.protobuf.Any allowance = 3;
 }

..MsgRevokeAllowance removes any existing FeeAllowance from Granter to Grantee.
 message MsgRevokeAllowance {
     string granter = 1;
     string grantee = 2;
 }
```

In order to use allowances in transactions, we add a new field `granter` to the transaction `Fee` type:

```proto
package cosmos.tx.v1beta1;

message Fee {
  repeated cosmos.base.v1beta1.Coin amount = 1;
  uint64 gas_limit = 2;
  string payer = 3;
  string granter = 4;
}
```

`granter`は節必要か、、だけ
経费経人の経费手当(最初は署名者なし→「経人」経のこと)。

「Fee_payer」との取引をする理する可能に、「DeductGrantedFeeDecorator」を表示名の「AnteDecorator」がありますます。
経费積金方法経费を設定し、ます控除します。

## 結果

###

-経営者。財のためだけに足入裝するのが面倒なユースケースに身事

### ネガティブ

### ニュートラル

-経営者。財のためだけに足入裝するのが面倒なユースケースに身事

## リファレンス

-元の職務記述書と提出:https://medium.com/regen-network/hacking-the-cosmos-cosmwasm-and-key-management-a08b9f561d1b
-新しい規制とスタイルがオープン:https://gist.github.com/aaronc/b60628017352df5983791cad30babe56
-B-harvestから元のテレビの設定は、この設定计に傒和えます:https://github.com/cosmos/cosmos-sdk/issues/4480 