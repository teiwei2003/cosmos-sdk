# 前処理プログラム

現在、 `x / auth`モジュールには独自のトランザクションハンドラーがありませんが、特別な` AntiHandler`を公開して、トランザクションの基本的な有効性チェックを実行し、メモリプールからスローできるようにします。
[ADR 010](https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-alpha1/docs/architecture/adr-010-modular-antehandler.md)によると。

Tendermintの提案者は現在、提案されたブロックトランザクションに失敗した `CheckTx`を含めることができるため、` AnteHandler`は `CheckTx`と` DeliverTx`の両方で呼び出されることに注意してください。

## デコレータ

authモジュールは `AnteDecorator`を提供します。これは、次の順序で単一の` AnteHandler`に再帰的にリンクされます。

-`SetUpContextDecorator`： `Context`に` GasMeter`を設定し、次の `AntiHandler`をdefer句でラップして、` AnteHandler`チェーンのダウンストリーム `OutOfGas`パニックから回復し、供給されたガス情報エラーに関する情報を返します。使用ガス。

-`RejectExtensionOptionsDecorator`：protobufトランザクションにオプションで含めることができるすべての拡張オプションを拒否します。

-`MempoolFeeDecorator`： `CheckTx`中に` tx`料金がローカルメモリプールの `minFee`パラメータよりも高いかどうかを確認します。

-`ValidateBasicDecorator`： `tx.ValidateBasic`を呼び出し、ゼロ以外のエラーを返します。

-`TxTimeoutHeightDecorator`：タイムアウトの `tx`の高さを確認します。

-`ValidateMemoDecorator`：アプリケーションパラメータを使用して `tx`メモを検証し、ゼロ以外のエラーを返します。

-`ConsumeGasTxSizeDecorator`：アプリケーションパラメータに従って `tx`のサイズに比例したガスを消費します。

-`DeductFeeDecorator`： `tx`の最初の署名者から` FeeAmount`を差し引きます。 `x / feegrant`モジュールが有効になっていて、料金付与者が設定されている場合、料金付与者アカウントから料金が差し引かれます。

-`SetPubKeyDecorator`：対応する公開鍵をステートマシンと現在のコンテキストに保存していない `tx`の署名者からの公開鍵を設定します。

-`ValidateSigCountDecorator`：アプリケーションパラメータに基づいて `tx`の署名の数を検証します。

-`SigGasConsumeDecorator`：各シグニチャの消費パラメータによって定義されたガスの量。これには、コンテキスト内のすべての署名者の「SetPubKeyDecorator」の一部として公開鍵を設定する必要があります。

-`SigVerificationDecorator`：すべての署名が有効であることを確認します。これには、コンテキスト内のすべての署名者の「SetPubKeyDecorator」の一部として公開鍵を設定する必要があります。

-`IncrementSequenceDecorator`：リプレイ攻撃を防ぐために、各署名者のアカウントシーケンスを追加します。