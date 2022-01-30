# 状態

## 料金手当

手数料手当は、[Grantee](手数料手当付与者の口座住所)と[Granter](手数料手当付与者の口座住所)を組み合わせることによって識別されます。

手数料手当の助成金は、次のように州に保管されます。

- 付与: `0x00 | grantee_addr_len(1バイト)| grantee_addr_bytes | granter_addr_len(1バイト)| granter_addr_bytes-> ProtocolBuffer(Grant) `

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/x/feegrant/feegrant.pb.go#L221-L229