# 模块列表

以下是一些可用于 Cosmos SDK 应用程序的生产级模块及其各自的文档:

- [Auth](auth/) - Cosmos SDK 应用程序的账户和交易认证。
- [Authz](authz/) - 授权账户代表其他账户执行操作。
- [银行](bank/) - 代币转移功能。
- [Capability](capability/) - 对象能力实现。
- [Crisis](crisis/) - 在某些情况下停止区块链(例如，如果不变量被破坏)。
- [Distribution](distribution/) - 费用分配和抵押代币供应分配。
- [Epoching](epoching/) - 允许模块将消息排队以在特定块高度执行。
- [Evidence](evidence/) - 双重签名、不当行为等的证据处理。
- [Feegrant](feegrant/) - 授予执行交易的费用津贴。
- [治理](gov/) - 链上提案和投票。
- [Mint](mint/) - 创建新的抵押代币单位。
- [Params](params/) - 全局可用的参数存储。
- [Slashing](slashing/) - 验证者惩罚机制。
- [Staking](staking/) - 公共区块链的权益证明层。
- [升级](upgrade/) - 软件升级处理和协调。

要了解有关构建模块过程的更多信息，请访问 [构建模块参考文档](/building-modules/intro.html)。

## 国际商业银行

SDK 的 IBC 模块已移至其[自己的存储库](https://github.com/cosmos/ibc-go)。 