# 钩子

当某个事件发生时，其他模块可能会注册要执行的操作
发生在 Staking 内。 可以注册这些事件以执行
对赌注事件的“之前”或“之后”(根据挂钩名称)。 这
以下挂钩可以通过 staking 注册: 

- `AfterValidatorCreated(Context, ValAddress) error`
    - called when a validator is created
- `BeforeValidatorModified(Context, ValAddress) error`
    - called when a validator's state is changed
- `AfterValidatorRemoved(Context, ConsAddress, ValAddress) error`
    - called when a validator is deleted
- `AfterValidatorBonded(Context, ConsAddress, ValAddress) error`
    - called when a validator is bonded
- `AfterValidatorBeginUnbonding(Context, ConsAddress, ValAddress) error`
    - called when a validator begins unbonding
- `BeforeDelegationCreated(Context, AccAddress, ValAddress) error`
    - called when a delegation is created
- `BeforeDelegationSharesModified(Context, AccAddress, ValAddress) error`
    - called when a delegation's shares are modified
- `BeforeDelegationRemoved(Context, AccAddress, ValAddress) error`
    - called when a delegation is removed
