# ADR 004:分割金種キー

## 変更ログ

-2020-01-08:初期バージョン
-2020-01-09:アトリビューションアカウントの変更処理
-2020-01-14:コメントフィードバックの更新
-2020-01-30:実装の更新

### 用語集

*金種/金種キー-一意のトークン識別子。

## 環境

許可なくIBCを使用すると、誰でも他のアカウントに任意の金額を送信できます。現在、ゼロ以外のすべての残高はアカウントとともに「sdk.Coins」構造で保存されます。これにより、アカウントが変更されるたびに多くの金種を読み込んで保存するのに費用がかかるため、サービス拒否の問題が発生する可能性があります。その他のコンテキストの場合、問題[5467](https://github.com/cosmos/cosmos-sdk/issues/5467)および[4982](https://github.com/cosmos/cosmos-sdk/issues/4982)を参照してください。 。

金種の数の制限後に預金を拒否するだけでは、悲しみのベクトルが開かれるため、機能しません。誰かがIBCを介してユーザーに無意味なコインを大量に送信し、ユーザーが実際の金種を受け取らないようにすることができます(賭けの報酬として)。

## 決定

残高は、特定の金種の特定の口座残高へのO(1)読み取りおよび書き込みアクセスを実現するために、口座および金種に応じた口座の金種および一意のキーの下に保存する必要があります。

### アカウントインターフェース(x/auth)

コインの残高が増えるため、 `GetCoins()`と `SetCoins()`はアカウントインターフェースから削除されます
これで、bankモジュールに保存され、管理されます。

アトリビューションアカウントインターフェイスは、「LockedCoins」をサポートするために「SpendableCoins」に置き換わります
アカウントの残高は不要になりました。さらに、 `TrackDelegation()`はこれを受け入れるようになります
全体をロードするのではなく、既得残高で表示されるすべてのトークンのアカウント残高
勘定残高。

既得のアカウントは、元の帰属、無料のコミッション、およびコミッションを引き続き保存します
アトリビューションコイン(任意の金種を含めることができないため、これは安全です)。

### 銀行管理者(x/bank)

次のAPIが `x/bank`キーパーに追加されます。 

-`GetAllBalances(ctx Context, addr AccAddress) Coins`
-`GetBalance(ctx Context, addr AccAddress, denom string) Coin`
- `SetBalance(ctx Context, addr AccAddress, coin Coin)`
- `LockedCoins(ctx Context, addr AccAddress) Coins`
- `SpendableCoins(ctx Context, addr AccAddress) Coins`

反復とアクセシビリティを促進するためにAPIを追加する場合があります
コア機能または永続性。

残高は、最初に住所ごとに保存され、次に金種ごとに保存されます(その逆も同様です。
ただし、単一のアカウントのすべての残高を取得する頻度が高いと仮定します): 

```golang
var BalancesPrefix = []byte("balances")

func (k Keeper) SetBalance(ctx Context, addr AccAddress, balance Coin) error {
  if !balance.IsValid() {
    return err
  }

  store := ctx.KVStore(k.storeKey)
  balancesStore := prefix.NewStore(store, BalancesPrefix)
  accountStore := prefix.NewStore(balancesStore, addr.Bytes())

  bz := Marshal(balance)
  accountStore.Set([]byte(balance.Denom), bz)

  return nil
}
```

これにより、残高はバイトインデックスで表されます
`バランス/{アドレス}/{デノム}`。

`DelegateCoins()`と `UndelegateCoins()`は、全員をロードするように変更されます
(非)委託額に見られる金種別の口座残高。したがって、
口座残高の変更は、金種によって行われます。

`SubtractCoins()`と `AddCoins()`は、読み取りと書き込みのバランスに変更されます
`GetCoins()`/`SetCoins()`を呼び出す代わりに直接(もう存在しません)。

`trackDelegation()`と `trackUndelegation()`は更新されないように変更されます
勘定残高。

外部APIは、下位互換性を維持するために、アカウントの下のすべての残高をスキャンする必要があります。それ
次の状況では、これらのAPIで `GetAllBalances`の代わりに` GetBalance`と `SetBalance`を使用することをお勧めします
アカウントの残高全体が読み込まれない場合があります。

### 供給モジュール

供給モジュールは、総供給不変量を達成するために、現在、
すべてのアカウントをスキャンし、 `x/bank`キーパーを使用して` GetAllBalances`を呼び出し、合計します
バランスを取り、予想される総供給量と一致するかどうかを確認します。

## ステータス

受け入れられました。

## 結果

### 目的

-O(1)読み書き残高(金種番号について)
アカウントの1つにゼロ以外の残高があります)。これは実際の状況とは関係がないことに注意してください
必要な直接読み取りの総数ではなく、I/Oコスト。

### ネガティブ

-すべての天びんを読み書きするときの読み取り/書き込み効率がわずかに低下します
トランザクション内の単一のアカウント。

### ニュートラル

特にない。

## 参照する

-参照:https://github.com/cosmos/cosmos-sdk/issues/4982
-参照:https://github.com/cosmos/cosmos-sdk/issues/5467
-参照:https://github.com/cosmos/cosmos-sdk/issues/5492 