# メッセージ

このセクションでは、`slashing`モジュールのメッセージの処理について説明します。

## Unjail

ダウンタイムのためにバリデーターが自動的に結合解除され、オンラインに戻って結合されたセットに再度参加したい場合は、 `MsgUnjail`を送信する必要があります。

```protobuf
//MsgUnjail is an sdk.Msg used for unjailing a jailed validator, thus returning
//them into the bonded validator set, so they can begin receiving provisions
//and rewards again.
message MsgUnjail {
  string validator_addr = 1;
}
```

以下は、`MsgSrv/Unjail`RPCの擬似コードです。

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

バリデーターが上位の `n = MaximumBondedValidators`に入るのに十分なステークを持っている場合、バリデーターは自動的に再結合されます。
そして、バリデーターにまだ委任されているすべての委任者は再結合され、再び引当金と報酬の収集を開始します。