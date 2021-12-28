# 情報

## メッセージ送信

あるアドレスから別のアドレスにコインを送ってください。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/bank/v1beta1/tx.proto#L19-L28

次の状況では、メッセージは失敗します。

- コイン送信が有効になっていません
- `to`アドレスは制限されています

## MsgMultiSend

一連の異なるアドレスとの間でコインを送信します。 受信アドレスが既存のアカウントに対応していない場合は、新しいアカウントが作成されます。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/bank/v1beta1/tx.proto#L33-L39

次の状況では、メッセージは失敗します。

- どのコインでも送信が有効になっていません
- 「宛先」アドレスはすべて制限されています
- コインはすべてロックされています
- 入力と出力が正しく対応していません