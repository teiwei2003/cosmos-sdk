# BeginBlockerとEndBlocker

`BeginBlocker`と` EndBlocker`は、モジュール開発者がモジュールに実装できるオプションのメソッドです。[`BeginBlock`](../core/baseapp.md#beginblock)および[` EndBlock`](../core/baseapp.md #endblock)が基盤となるコンセンサスエンジンからABCIメッセージを受信した場合。 {まとめ}

## 読むための前提条件

-[モジュールマネージャー](。/module-manager.md){前提条件}

## BeginBlockerとEndBlocker

`BeginBlocker`と` EndBlocker`は、モジュール開発者が自動実行のためにモジュールにロジックを追加するための方法です。これは強力なツールであり、複雑な自動機能によってチェーンの速度が低下したり、停止したりする可能性があるため、注意して使用する必要があります。

必要に応じて、 `BeginBlocker`と` EndBlocker`は[`AppModule`インターフェース](./module-manager.md#appmodule)の一部として実装されます。 `module.go`に実装されているインターフェースの` BeginBlock`メソッドと `EndBlock`メソッドは、通常、それぞれ` BeginBlocker`メソッドと `EndBlocker`メソッドに従い、通常は` abci.go`に実装されています。

`abci.go`での` BeginBlocker`と `EndBlocker`の実際の実装は、[` Msg` service](./msg-services.md)の実装と非常によく似ています。

-通常、[`keeper`](./keeper.md)と[` ctx`](../core/context.md)を使用して、最新の状態に関する情報を取得します。
-必要に応じて、 `keeper`と` ctx`を使用して状態遷移をトリガーします。
-必要に応じて、 `ctx`の` EventManager`を介して[`events`](../core/events.md)を発行できます。

`EndBlocker`の特徴の1つは、[`[] abci.ValidatorUpdates`](https://tendermint.com/docs/app-dev/abci-spec.html#validatorupdate)として使用できることです。これは、カスタムバリデーターの変更を実装するための推奨される方法です。

開発者は、モジュールマネージャーの[SetOrderBeginBlocker]/[SetOrderEndBlocker]メソッドを使用して、各アプリケーションモジュールの[BeginBlocker]/[EndBlocker]関数間の実行順序を定義できます。モジュールマネージャの詳細については、[ここ](./module-manager.md#manager)をクリックしてください。

`disr`モジュールからの` BeginBlocker`の実装例をご覧ください。

+++ https://github.com/cosmos/cosmos-sdk/blob/f33749263f4ecc796115ad6e789cb0f7cddf9148/x/distribution/abci.go#L14-L38

そして、ステーキングモジュールからのEndBlockerの実装例:

+++ https://github.com/cosmos/cosmos-sdk/blob/f33749263f4ecc796115ad6e789cb0f7cddf9148/x/staking/abci.go#L22-L27

## 次へ{hide}

[`keeper`s](./keeper.md){hide}を理解する 