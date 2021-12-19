# 状态

`x/bank` 模块保持三个主要对象的状态:

1. 账户余额
2. 面额元数据
3. 所有余额的总供应量

此外，`x/bank` 模块保存了以下索引来管理
上述状态:

- 供应指数:`0x0 | byte(denom) -> byte(amount)`
- Denom 元数据索引:`0x1 | byte(denom) -> ProtocolBuffer(Metadata)`
- 余额指数:`0x2 | 字节(地址长度)| []字节(地址)| []byte(balance.Denom) -> ProtocolBuffer(balance)`
- 反向命名地址索引:`0x03 | 字节(denom) | 0x00 | []字节(地址)-> 0`