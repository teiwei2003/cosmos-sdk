# 状态

## 费用津贴

费用津贴通过将“受赠人”(费用津贴受赠人的帐户地址)与“赠与人”(费用津贴授予人的帐户地址)组合在一起来识别。

费用津贴补助金在州中存储如下:

- 授予:`0x00 | grantee_addr_len (1 字节) | grantee_addr_bytes | granter_addr_len (1 字节) | granter_addr_bytes -> ProtocolBuffer(Grant)`

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/x/feegrant/feegrant.pb.go#L221-L229 