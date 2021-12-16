# 守门员

在app初始化阶段，可以使用`Keeper.Subspace`为其他模块的keeper分配[subspaces](02_subspace.md)，并存储在`Keeper.spaces`中。 然后，这些模块可以通过 `Keeper.GetSubspace` 引用它们的特定参数存储。 
Example:

```go
type ExampleKeeper struct {
	paramSpace paramtypes.Subspace
}

func (k ExampleKeeper) SetParams(ctx sdk.Context, params types.Params) {
	k.paramSpace.SetParamSet(ctx, &params)
}
```
