# 行われる変更

## バリデーターの自己バインド解除（最小の自己委任を超える）は、すぐにアクティブ化する必要がある場合があります

バインド解除プロセスをトリガーするケース

- バリデーターの委任解除は、minimum_self_delegationよりも多くのトークンをアンバインドでき、バリデーターをバインドされていない状態に自動的に変更します
この場合、バインド解除はすぐに開始する必要があります。
- バリデーターはブロックを見逃し、カットされました
- 二重署名のため、バリデーターがカットされました

**注：**ベリファイアがバインド解除プロセスを開始したら、すぐにベリファイアをバインド解除状態にする必要がある場合があります。
    これは、バインドを解除し始めた特定のプリンシパルとは異なります。 バインド解除を開始したバリデーターは、コレクションに含まれていないことを意味します。
    ベリファイアにバインドされていない委任者は、そのデリゲートをベリファイアから削除します。

## 開発される

```go
// Changes to make
// — Implement correct next epoch time calculation
// — For validator self undelegation, it could be required to do start on end blocker
// — Implement TODOs on the PR #46
// Implement CLI commands for querying
// — BufferedValidators
// — BufferedMsgCreateValidatorQueue, BufferedMsgEditValidatorQueue
// — BufferedMsgUnjailQueue, BufferedMsgDelegateQueue, BufferedMsgRedelegationQueue, BufferedMsgUndelegateQueue
// Write epoch related tests with new scenarios
// — Simulation test is important for finding bugs [Ask Dev for questions)
// — Can easily add a simulator check to make sure all delegation amounts in queue add up to the same amount that’s in the EpochUnbondedPool
// — I’d like it added as an invariant test for the simulator
// — the simulator should check that the sum of all the queued delegations always equals the amount kept track in the data
// — Staking/Slashing/Distribution module params are being modified by governance based on vote result instantly. We should test the effect.
// — — Should test to see what would happen if max_validators is changed though, in the middle of an epoch
// — we should define some new invariants that help check that everything is working smoothly with these new changes for 3 modules e.g. https://github.com/cosmos/cosmos-sdk/blob/master/x/staking/keeper/invariants.go
// — — Within Epoch, ValidationPower = ValidationPower - SlashAmount
// — — When epoch actions queue is empty, EpochDelegationPool balance should be zero
// — we should count all the delegation changes that happen during the epoch, and then make sure that the resulting change at the end of the epoch is actually correct
// — If the validator that I delegated to double signs at block 16, I should still get slashed instantly because even though I asked to unbond at 14, they still used my power at block 16, I should only be not liable for slashes once my power is stopped being used
// — On the converse of this, I should still be getting rewards while my power is being used.  I shouldn’t stop receiving rewards until block 20
```
