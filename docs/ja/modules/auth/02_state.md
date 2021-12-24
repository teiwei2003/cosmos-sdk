# 状態

## アカウント

アカウントには、SDKブロックチェーンによって一意に識別される外部ユーザーのID検証情報が含まれています。
リプレイ保護のために、公開鍵、アドレス、およびアカウント/シリアル番号を含めます。 効率のために、
料金を支払うにはアカウントの残高も取得する必要があるため、アカウント構造にはユーザーの残高も保存されます
`sdk.Coins`として。

アカウントはインターフェースとして公開されており、次のように内部的に保存されます。
基本アカウントまたは既得アカウント。 モジュールを追加したいお客様
アカウントタイプがこれを行う場合があります。

- `0x01 | アドレス -> ProtocolBuffer(account)`

### アカウントインターフェイス

アカウントインターフェイスは、標準のアカウント情報を読み書きするためのメソッドを公開します。
これらの方法はすべて、確認のためにアカウント構造で動作することに注意してください
インターフェース-アカウントをストレージに書き込むために、アカウント管理者は
使用する必要があります。 

```go
// AccountI is an interface used to store coins at a given address within state.
// It presumes a notion of sequence numbers for replay protection,
// a notion of account numbers for replay protection for previously pruned accounts,
// and a pubkey for authentication purposes.
//
// Many complex conditions can be used in the concrete struct which implements AccountI.
type AccountI interface {
	proto.Message

	GetAddress() sdk.AccAddress
	SetAddress(sdk.AccAddress) error // errors if already set.

	GetPubKey() crypto.PubKey // can return nil.
	SetPubKey(crypto.PubKey) error

	GetAccountNumber() uint64
	SetAccountNumber(uint64) error

	GetSequence() uint64
	SetSequence(uint64) error

	// Ensure that account implements stringer
	String() string
}
```

#### 基本アカウント

基本アカウントは最も単純で最も一般的なタイプのアカウントであり、必要なものをすべて保存するだけです。
フィールドは構造内に直接あります。

```protobuf
// BaseAccount defines a base account type. It contains all the necessary fields
// for basic account functionality. Any custom account type should extend this
// type for additional functionality (e.g. vesting).
message BaseAccount {
  string address = 1;
  google.protobuf.Any pub_key = 2;
  uint64 account_number = 3;
  uint64 sequence       = 4;
}
```

### アトリビューションアカウント

[属性]（05_vesting.md）を参照してください。