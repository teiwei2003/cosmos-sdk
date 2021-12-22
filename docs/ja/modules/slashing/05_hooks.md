# 钩子

本节包含对模块`hooks` 的描述。 挂钩是在引发事件时自动执行的操作。

## 放样钩子

slashing 模块实现了 `x/staking` 中定义的 `StakingHooks`，并用作验证者信息的记录保存。 在应用程序初始化期间，这些钩子应该在 staking 模块结构中注册。

以下挂钩会影响 slashing 状态:

+ `AfterValidatorBonded` 创建一个 `ValidatorSigningInfo` 实例，如下一节所述。
+ `AfterValidatorCreated` 存储验证者的共识密钥。
+ `AfterValidatorRemoved` 删除验证者的共识密钥。

## 验证者绑定

在成功首次绑定新验证器后，我们为
现在绑定的验证器，当前块的“StartHeight”。 

```
onValidatorBonded(address sdk.ValAddress)

  signingInfo, found = GetValidatorSigningInfo(address)
  if !found {
    signingInfo = ValidatorSigningInfo {
      StartHeight         : CurrentHeight,
      IndexOffset         : 0,
      JailedUntil         : time.Unix(0, 0),
      Tombstone           : false,
      MissedBloskCounter  : 0
    }
    setValidatorSigningInfo(signingInfo)
  }

  return
```
