# 状态

## 帐户

帐户包含 SDK 区块链唯一标识的外部用户的身份验证信息，
包括用于重放保护的公钥、地址和帐号/序列号。 为了效率，
由于还必须获取帐户余额以支付费用，因此帐户结构还存储用户的余额
作为`sdk.Coins`。

帐户作为接口对外公开，在内部存储为
基本账户或归属账户。 希望添加更多模块的客户
帐户类型可能会这样做。

- `0x01 | 地址 -> ProtocolBuffer(account)`

### 账户界面

帐户接口公开了读取和写入标准帐户信息的方法。
请注意，所有这些方法都在一个帐户结构上运行，以确认
接口 - 为了将帐户写入存储，帐户管理员将
需要使用。 
```go
//AccountI is an interface used to store coins at a given address within state.
//It presumes a notion of sequence numbers for replay protection,
//a notion of account numbers for replay protection for previously pruned accounts,
//and a pubkey for authentication purposes.
//
//Many complex conditions can be used in the concrete struct which implements AccountI.
type AccountI interface {
	proto.Message

	GetAddress() sdk.AccAddress
	SetAddress(sdk.AccAddress) error//errors if already set.

	GetPubKey() crypto.PubKey//can return nil.
	SetPubKey(crypto.PubKey) error

	GetAccountNumber() uint64
	SetAccountNumber(uint64) error

	GetSequence() uint64
	SetSequence(uint64) error

	//Ensure that account implements stringer
	String() string
}
```

#### 基本帐户

基本帐户是最简单和最常见的帐户类型，它只存储所有必需的
字段直接在结构中。

```protobuf
//BaseAccount defines a base account type. It contains all the necessary fields
//for basic account functionality. Any custom account type should extend this
//type for additional functionality (e.g. vesting).
message BaseAccount {
  string address = 1;
  google.protobuf.Any pub_key = 2;
  uint64 account_number = 3;
  uint64 sequence       = 4;
}
```

### 归属账户

请参阅[归属](05_vesting.md)。 