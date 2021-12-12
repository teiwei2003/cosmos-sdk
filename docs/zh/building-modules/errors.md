<!--
order: 13
-->

# 错误

本文档概述了 Cosmos SDK 模块中用于错误处理的推荐用法和 API。 {概要}

鼓励模块定义和注册自己的错误以提供更好的
失败消息或处理程序执行的上下文。通常，这些错误应该是
可以进一步包装以提供额外的特定错误的常见或一般错误
执行上下文。

## 登记

模块应该在 `x/{module}/errors.go` 中定义和注册它们的自定义错误。登记
错误是通过 `types/errors` 包处理的。

例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0/x/distribution/types/errors.go#L1-L21

每个自定义模块错误都必须提供代码空间，通常是模块名称
(例如“分发”)并且每个模块都是唯一的，以及一个 uint32 代码。一起，代码空间和代码
提供全球唯一的 Cosmos SDK 错误。通常，代码是单调递增的，但不会
必须是。错误代码的唯一限制如下:

* 必须大于 1，因为 1 的代码值是为内部错误保留的。
* 在模块内必须是唯一的。

请注意，Cosmos SDK 提供了一组核心的*常见*错误。这些错误在 `types/errors/errors.go` 中定义。

## 包装

自定义模块错误可以作为它们的具体类型返回，因为它们已经满足了 `error`
界面。但是，可以包装模块错误以提供进一步的上下文和含义以失败
执行。

例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/b2d48a9e815fe534a7faeec6ca2adb0874252b81/x/bank/keeper/keeper.go#L85-L122

不管错误是否被包装，Cosmos SDK 的 `errors` 包提供了一个 API 来确定是否
错误是通过`Is` 产生的一种特殊类型。

## ABCI

如果注册了模块错误，Cosmos SDK `errors` 包允许提取 ABCI 信息
通过`ABCIInfo` API。该包还提供了 `ResponseCheckTx` 和 `ResponseDeliverTx` 作为
辅助 API 可自动从错误中获取 `CheckTx` 和 `DeliverTx` 响应。 