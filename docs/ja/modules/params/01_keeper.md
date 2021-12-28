# キーパー

アプリの初期化段階では、[subspaces](02_subspace.md)を `Keeper.Subspace`を使用して他のモジュールのキーパーに割り当てることができ、`Keeper.spaces`に保存されます。 次に、これらのモジュールは、`Keeper.GetSubspace`を介して特定のパラメータストアへの参照を持つことができます。 

例:

```go
type ExampleKeeper struct {
	paramSpace paramtypes.Subspace
}

func (k ExampleKeeper) SetParams(ctx sdk.Context, params types.Params) {
	k.paramSpace.SetParamSet(ctx, &params)
}
```
