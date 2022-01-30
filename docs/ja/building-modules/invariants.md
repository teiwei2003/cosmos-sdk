# 常数

不変条件はアプリケーションの属性であり、常に真である必要があります。 Cosmos SDKのコンテキストでは、[不変条件]は特定の不変条件をチェックする関数です。これらの関数を使用して、エラーを早期に検出し、それらに対してアクションを実行して、潜在的な結果を制限することができます(たとえば、チェーンを停止することによって)。また、アプリケーションの開発にも非常に役立ち、シミュレーションを通じてエラーを検出するために使用できます。 {まとめ}

## 読むための前提条件

-[キーパー](。/keeper.md){前提条件}

## `Invariant`を実装する

`Invariant`は、モジュール内の特定の不変条件をチェックする関数です。モジュール `Invariant`は` Invariant`タイプに従う必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/types/invariant.go#L9

`string`の戻り値は不変のメッセージであり、ログを出力するときに使用できます。`bool`の戻り値は、不変のチェックの実際の結果です。

実際、各モジュールは、モジュールフォルダのファイル `。/keeper/invariants.go`に` Invariant`を実装しています。標準では、次のモデルを使用して、不変条件の論理グループごとに[不変条件]関数を実装します。 

```go
//Example for an Invariant that checks balance-related invariants

func BalanceInvariants(k Keeper) sdk.Invariant {
	return func(ctx sdk.Context) (string, bool) {
      //Implement checks for balance-related invariants
    }
}
```

さらに、モジュール開発者は通常、モジュールのすべての `Invariant`関数を実行するために` AllInvariants`関数を実装する必要があります。 

```go
//AllInvariants runs all invariants of the module.
//In this example, the module implements two Invariants: BalanceInvariants and DepositsInvariants

func AllInvariants(k Keeper) sdk.Invariant {

	return func(ctx sdk.Context) (string, bool) {
		res, stop := BalanceInvariants(k)(ctx)
		if stop {
			return res, stop
		}

		return DepositsInvariant(k)(ctx)
	}
}
```

最後に、モジュール開発者は、[`AppModule`インターフェース](./module-manager.md#appmodule)の一部として` RegisterInvariants`メソッドを実装する必要があります。 実際、 `module/module.go`ファイルに実装されたモジュールの` RegisterInvariants`メソッドは、通常、 `keeper/invariants.go`ファイルに実装された` RegisterInvariants`メソッドの呼び出しを遅らせるだけです。 `RegisterInvariants`メソッドは、[` InvariantRegistry`](#invariant-registry)の各 `Invariant`関数のルートを登録します。 

```go
//RegisterInvariants registers all staking invariants
func RegisterInvariants(ir sdk.InvariantRegistry, k Keeper) {
	ir.RegisterRoute(types.ModuleName, "module-accounts",
		BalanceInvariants(k))
	ir.RegisterRoute(types.ModuleName, "nonnegative-power",
		DepositsInvariant(k))
}
```

詳細については、[`stakeing`モジュールでの` Invariant`実装の例](https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/staking/keeper/invariants.go)を参照してください。

## 変更されていないレジストリ

`InvariantRegistry`は、アプリケーションのすべてのモジュールの` Invariant`が登録されているレジストリです。 **アプリケーション**ごとに `InvariantRegistry`は1つだけです。つまり、モジュール開発者は、モジュールを構築するときに独自の` InvariantRegistry`を実装する必要はありません。 **モジュール開発者が行う必要があるのは、前のセクションで説明したように、モジュールの不変条件を `InvariantRegistry`に登録することだけです**。このセクションの残りの部分では、 `InvariantRegistry`自体に関する詳細情報を提供し、モジュール開発者に直接関連するものは含まれていません。

基本的に、 `InvariantRegistry`は、CosmosSDKのインターフェイスとして定義されています。

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/types/invariant.go#L14-L17

通常、このインターフェースは特定のモジュールの[キーパー]に実装されます。 `InvariantRegistry`の最も一般的に使用される実装は、` crisis`モジュールにあります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/crisis/keeper/keeper.go#L50-L54

 したがって、 `InvariantRegistry`は通常、[アプリケーションコンストラクター](../basics/app-anatomy.md#constructor-function)の` crisis`モジュールの `keeper`をインスタンス化することによってインスタンス化されます。

`Invariant`は、[` message`s](./messages-and-queries.md)を介して手動でチェックできますが、ほとんどの場合、各ブロックの最後で自動的にチェックされます。 [crisis]モジュールの例を次に示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/crisis/abci.go#L7-L14

どちらの場合も、 `Invariant`の1つがfalseを返すと、` InvariantRegistry`は特別なロジックをトリガーできます(たとえば、アプリケーションをパニックにし、 `Invariant`メッセージをログに出力します)。

## 次へ{hide}

[作成関数](./genesis.md)を理解する{hide} 