<!--
order: 8
-->

# 不变量

不变量是应用程序的一个属性，它应该始终为真。在 Cosmos SDK 的上下文中，“不变量”是检查特定不变量的函数。这些功能可用于及早检测错误并对其采取行动以限制其潜在后果(例如，通过停止链)。它们在应用程序的开发过程中也很有用，可以通过模拟检测错误。 {概要}

## 先决条件阅读

- [Keepers](./keeper.md) {prereq}

## 实现 `Invariant`s

`Invariant` 是一个函数，用于检查模块中的特定不变量。模块 `Invariant` 必须遵循 `Invariant` 类型:

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/types/invariant.go#L9

`string` 返回值是不变消息，可以在打印日志时使用，`bool` 返回值是不变检查的实际结果。

实际上，每个模块在模块文件夹内的`./keeper/invariants.go` 文件中实现`Invariant`。标准是使用以下模型为每个不变量的逻辑分组实现一个“不变量”函数: 

```go
// Example for an Invariant that checks balance-related invariants

func BalanceInvariants(k Keeper) sdk.Invariant {
	return func(ctx sdk.Context) (string, bool) {
        // Implement checks for balance-related invariants
    }
}
```

此外，模块开发人员通常应该实现一个 `AllInvariants` 函数来运行模块的所有 `Invariant` 函数:

```go
// AllInvariants runs all invariants of the module.
// In this example, the module implements two Invariants: BalanceInvariants and DepositsInvariants

func AllInvariants(k Keeper) sdk.Invariant {

	return func(ctx sdk.Context) (string, bool) {
		res, stop := BalanceInvariants(k)(ctx)
		if stop {
			return res, stop
		}

		return DepositsInvariant(k)(ctx)
	}
}
```

最后，模块开发者需要实现 `RegisterInvariants` 方法作为 [`AppModule` 接口](./module-manager.md#appmodule) 的一部分。 实际上，在`module/module.go` 文件中实现的模块的`RegisterInvariants` 方法通常只将调用延迟到在`keeper/invariants.go` 文件中实现的`RegisterInvariants` 方法。 `RegisterInvariants` 方法为 [`InvariantRegistry`](#invariant-registry) 中的每个 `Invariant` 函数注册一个路由: 

```go
// RegisterInvariants registers all staking invariants
func RegisterInvariants(ir sdk.InvariantRegistry, k Keeper) {
	ir.RegisterRoute(types.ModuleName, "module-accounts",
		BalanceInvariants(k))
	ir.RegisterRoute(types.ModuleName, "nonnegative-power",
		DepositsInvariant(k))
}
```

有关更多信息，请参阅 [`staking` 模块中的 `Invariant` 实现示例](https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/staking/keeper/invariants.go)。

## 不变注册表

`InvariantRegistry` 是一个注册表，其中注册了应用程序所有模块的 `Invariant`。每个 **application** 只有一个 `InvariantRegistry`，这意味着模块开发人员在构建模块时不需要实现他们自己的 `InvariantRegistry`。 **所有模块开发人员需要做的是在 `InvariantRegistry` 中注册他们模块的不变量，如上一节所述**。本节的其余部分提供有关 `InvariantRegistry` 本身的更多信息，不包含与模块开发人员直接相关的任何内容。

在其核心，`InvariantRegistry` 在 Cosmos SDK 中被定义为一个接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/types/invariant.go#L14-L17

通常，此接口在特定模块的“keeper”中实现。 `InvariantRegistry` 最常用的实现可以在 `crisis` 模块中找到:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/crisis/keeper/keeper.go#L50-L54

 因此，`InvariantRegistry` 通常是通过在 [应用程序的构造函数](../basics/app-anatomy.md#constructor-function) 中实例化 `crisis` 模块的 `keeper` 来实例化的。

`Invariant`s 可以通过 [`message`s](./messages-and-queries.md) 手动检查，但大多数情况下它们会在每个块的末尾自动检查。下面是来自“crisis”模块的一个例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/crisis/abci.go#L7-L14

在这两种情况下，如果 `Invariant` 之一返回 false，`InvariantRegistry` 可以触发特殊逻辑(例如，让应用程序恐慌并在日志中打印 `Invariant`s 消息)。

## 下一个{hide}

了解 [创世功能](./genesis.md) {hide} 