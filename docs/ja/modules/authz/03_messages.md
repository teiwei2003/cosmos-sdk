# 情報

このセクションでは、authzモジュールのメッセージ処理について説明します。

## メッセージ認証

「MsgGrant」メッセージを使用してライセンスを作成します。
`(granter、grantee、Authorization)`トリプレットの承認がすでにある場合、新しい承認は以前の承認を上書きします。 既存の承認を更新または拡張するには、同じ「(承認者、承認された人、承認)」トリプレットを持つ新しい承認を作成する必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L32-L37

次の条件が発生した場合、メッセージ処理は失敗するはずです。

- 付与者と受信者のアドレスは同じです。
- 提供された `Expiration`時間は、現在のUNIXタイムスタンプよりも短くなっています。
- `Grant.Authorization`が実装されていない場合。
- `Authorization.MsgTypeURL() `がルーターで定義されていません(このメッセージタイプを処理するには、ルーターで定義されていないハンドラーを適用してください)。

## メッセージキャンセル

承認は、 `MsgRevoke`メッセージを使用して削除できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L60-L64

次の条件が発生した場合、メッセージ処理は失敗するはずです。

- 付与者と受信者のアドレスは同じです。
- `MsgTypeUrl`が空であると想定します。

注:認証の有効期限が切れている場合、 `MsgExec`メッセージは認証を削除します。

## メッセージの実行

受信者が受信者に代わってトランザクションを実行したい場合は、「MsgExec」を送信する必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L47-L53

次の条件が発生した場合、メッセージ処理は失敗するはずです。

- 「承認」が達成されていないことが前提です。
- 譲受人には、トランザクションを実行する権利がありません。
- 付与された認証の有効期限が切れている場合。