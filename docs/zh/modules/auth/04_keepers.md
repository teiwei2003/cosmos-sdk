# 守门员

auth 模块只暴露了一个keeper，accountkeeper，可以用来读写账户。

## 帐户管理员

目前只暴露了一个完全许可的帐户管理员，它具有读写能力
所有帐户的所有字段，并遍历所有存储的帐户。 

```go
//AccountKeeperI is the interface contract that x/auth's keeper implements.
type AccountKeeperI interface {
	//Return a new account with the next account number and the specified address. Does not save the new account to the store.
	NewAccountWithAddress(sdk.Context, sdk.AccAddress) types.AccountI

	//Return a new account with the next account number. Does not save the new account to the store.
	NewAccount(sdk.Context, types.AccountI) types.AccountI

	//Check if an account exists in the store.
	HasAccount(sdk.Context, sdk.AccAddress) bool

	//Retrieve an account from the store.
	GetAccount(sdk.Context, sdk.AccAddress) types.AccountI

	//Set an account in the store.
	SetAccount(sdk.Context, types.AccountI)

	//Remove an account from the store.
	RemoveAccount(sdk.Context, types.AccountI)

	//Iterate over all accounts, calling the provided function. Stop iteration when it returns true.
	IterateAccounts(sdk.Context, func(types.AccountI) bool)

	//Fetch the public key of an account at a specified address
	GetPubKey(sdk.Context, sdk.AccAddress) (crypto.PubKey, error)

	//Fetch the sequence of an account at a specified address.
	GetSequence(sdk.Context, sdk.AccAddress) (uint64, error)

	//Fetch the next account number, and increment the internal counter.
	GetNextAccountNumber(sdk.Context) uint64
}
```
