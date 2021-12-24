# 状態

## 許す

承認は、承認者のアドレス(承認者のアドレスバイト)、承認された人のアドレス(承認者のアドレスバイト)、および承認タイプ(そのタ​​イプURL)を組み合わせることによって識別されます。 したがって、(付与者、被付与者、承認)トリプレットに対して1つの承認のみを許可します。

- 許す:`0x01 | granter_address_len (1 字节) | granter_address_bytes | grantee_address_len (1 字节) | grantee_address_bytes | msgType_bytes-> ProtocolBuffer(AuthorizationGrant)`

付与オブジェクトは、承認タイプと有効期限のタイムスタンプをカプセル化します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/authz.proto#L21-L26 