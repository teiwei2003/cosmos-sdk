# 要进行的更改

## 验证者自解绑(超过最小自委派)可能需要立即启动

触发解绑过程的案例

- 验证人 undelegate 可以解除绑定比他的 minimum_self_delegation 多的代币，它会自动将验证人变为解除绑定状态
在这种情况下，应立即开始解绑。
- 验证器错过块并被削减
- 验证器因双重签名而被削减

**注意:** 当验证者开始解绑过程时，可能需要立即将验证器转为解绑状态。
   这与开始解除绑定的特定委托人不同。 开始解除绑定的验证器意味着它不再在集合中。
   与验证者解除绑定的委托人会将其委托从验证者中移除。

## 待开发 

```go
//Changes to make
//— Implement correct next epoch time calculation
//— For validator self undelegation, it could be required to do start on end blocker
//— Implement TODOs on the PR #46
//Implement CLI commands for querying
//— BufferedValidators
//— BufferedMsgCreateValidatorQueue, BufferedMsgEditValidatorQueue
//— BufferedMsgUnjailQueue, BufferedMsgDelegateQueue, BufferedMsgRedelegationQueue, BufferedMsgUndelegateQueue
//Write epoch related tests with new scenarios
//— Simulation test is important for finding bugs [Ask Dev for questions)
//— Can easily add a simulator check to make sure all delegation amounts in queue add up to the same amount that’s in the EpochUnbondedPool
//— I’d like it added as an invariant test for the simulator
//— the simulator should check that the sum of all the queued delegations always equals the amount kept track in the data
//— Staking/Slashing/Distribution module params are being modified by governance based on vote result instantly. We should test the effect.
//— — Should test to see what would happen if max_validators is changed though, in the middle of an epoch
//— we should define some new invariants that help check that everything is working smoothly with these new changes for 3 modules e.g. https://github.com/cosmos/cosmos-sdk/blob/master/x/staking/keeper/invariants.go
//— — Within Epoch, ValidationPower = ValidationPower - SlashAmount
//— — When epoch actions queue is empty, EpochDelegationPool balance should be zero
//— we should count all the delegation changes that happen during the epoch, and then make sure that the resulting change at the end of the epoch is actually correct
//— If the validator that I delegated to double signs at block 16, I should still get slashed instantly because even though I asked to unbond at 14, they still used my power at block 16, I should only be not liable for slashes once my power is stopped being used
//— On the converse of this, I should still be getting rewards while my power is being used.  I shouldn’t stop receiving rewards until block 20
```
