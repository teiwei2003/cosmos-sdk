# Hooks

他のモジュールは、ステーキング内で特定のイベントが発生したときに実行する操作を登録する場合があります。
これらのイベントは、ステーキングイベントの「前」または「後」のいずれかを実行するように登録できます（フック名に従って）。 次のフックをステーキングに登録できます。

- `AfterValidatorCreated(Context, ValAddress) error`
    - バリデーターが作成されたときに呼び出されます
- `BeforeValidatorModified(Context, ValAddress) error`
    - バリデーターの状態が変更されたときに呼び出されます
- `AfterValidatorRemoved(Context, ConsAddress, ValAddress) error`
    - バリデーターが削除されたときに呼び出されます
- `AfterValidatorBonded(Context, ConsAddress, ValAddress) error`
    - バリデーターが結合されたときに呼び出されます
- `AfterValidatorBeginUnbonding(Context, ConsAddress, ValAddress) error`
    - バリデーターが結合解除を開始したときに呼び出されます
- `BeforeDelegationCreated(Context, AccAddress, ValAddress) error`
    - 委任が作成されたときに呼び出されます
- `BeforeDelegationSharesModified(Context, AccAddress, ValAddress) error`
    - 代表団の株式が変更されたときに呼び出されます
- `BeforeDelegationRemoved(Context, AccAddress, ValAddress) error`
    - 委任が削除されたときに呼び出されます
