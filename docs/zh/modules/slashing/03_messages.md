# 消息

在本节中，我们描述了 `slashing` 模块的消息处理。

## 出狱

如果验证器因停机而自动解除绑定并希望重新上线&
可能重新加入绑定集，它必须发送`MsgUnjail`:

```protobuf
//MsgUnjail is an sdk.Msg used for unjailing a jailed validator, thus returning
//them into the bonded validator set, so they can begin receiving provisions
//and rewards again.
message MsgUnjail {
  string validator_addr = 1;
}
```

下面是 `MsgSrv/Unjail` RPC 的伪代码: 

```
unjail(tx MsgUnjail)
    validator = getValidator(tx.ValidatorAddr)
    if validator == nil
      fail with "No validator found"

    if getSelfDelegation(validator) == 0
      fail with "validator must self delegate before unjailing"

    if !validator.Jailed
      fail with "Validator not jailed, cannot unjail"

    info = GetValidatorSigningInfo(operator)
    if info.Tombstoned
      fail with "Tombstoned validator cannot be unjailed"
    if block time < info.JailedUntil
      fail with "Validator still jailed, cannot unjail until period has expired"

    validator.Jailed = false
    setValidator(validator)

    return
```

如果验证者有足够的股份进入顶部`n = MaximumBondedValidators`，它将自动重新绑定，
并且所有仍然委托给验证者的委托人将被重新绑定并开始再次收集
规定和奖励。 