# フック

このセクションには、モジュールの「フック」の説明が含まれています。 フックは、イベントが発生したときに自動的に実行される操作です。

## Staking hooks

スラッシュモジュールは、`x/staking`で定義され`StakingHooks`を実装し、バリデーター情報の記録保持として使用されます。 アプリの初期化中に、これらのフックをステーキングモジュール構造体に登録する必要があります。

次のフックは、スラッシュ状態に影響を与えます。

+ `AfterValidatorBonded`は、次のセクションで説明するように、` ValidatorSigningInfo`インスタンスを作成します。
+ `AfterValidatorCreated`は、バリデーターのコンセンサスキーを格納します。
+ `AfterValidatorRemoved`は、バリデーターのコンセンサスキーを削除します。

## バリデーター結合

新しいバリデーターの最初の結合が成功すると、結合されたバリデーターの新しい`ValidatorSigningInfo`構造を作成します。これは、現在のブロックの`StartHeight`です。 

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
