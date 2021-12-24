# パラメータ

bankモジュールには次のパラメータが含まれています。

| Key                | Type          | Example                            |
| ------------------ | ------------- | ---------------------------------- |
| SendEnabled        | []SendEnabled | [{denom: "stake", enabled: true }] |
| DefaultSendEnabled | bool          | true                               |

## 送信イネーブル

send enableパラメーターは、コインをマッピングするSendEnabledエントリのセットです。
send_enabled状態への金種。 このリストのエントリには
`DefaultSendEnabled`設定よりも優先されます。

## デフォルトの送信が有効になっています

デフォルトの送信イネーブル値は、すべての送信および送信機能を制御します
特に `SendEnabled`配列に含まれていない限り、コインの種類
パラメータ。