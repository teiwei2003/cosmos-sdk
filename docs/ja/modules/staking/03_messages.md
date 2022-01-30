# メッセージ

このセクションでは、ステーキングメッセージの処理とそれに対応する状態の更新について説明します。 各メッセージで指定されたすべての作成/変更された状態オブジェクトは、[state](./02_state_transitions.md)セクション内で定義されます。

## MsgCreateValidator

バリデーターは、 `MsgCreateValidator`メッセージを使用して作成されます。
バリデーターは、オペレーターからの最初の委任で作成する必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L16-L17

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L35-L51

このメッセージは、次の場合に失敗すると予想されます。

- このオペレーターアドレスを持つ別のバリデーターはすでに登録されています
- このpubkeyを持つ別のバリデーターはすでに登録されています
- 最初の自己委任トークンは、結合デノムとして指定されていないデノムのものです
- コミッションパラメータに誤りがあります。つまり、次のとおりです。
     - `MaxRate`は> 1または<0のいずれかです
     - 最初の `Rate`は負または>` MaxRate`のいずれかです
     - 最初の `MaxChangeRate`は負であるか、>` MaxRate`です
- 説明フィールドが大きすぎます

このメッセージは、 `Validator`オブジェクトを作成して適切なインデックスに格納します。
さらに、自己委任は、初期トークン委任トークン `委任`を使用して行われます。 バリデーターは常に非結合として開始されますが、最初のエンドブロックで結合される場合があります。

## MsgEditValidator

バリデーターの `Description`、` CommissionRate`は、 `MsgEditValidator`メッセージを使用して更新できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L19-L20

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L56-L76

このメッセージは、次の場合に失敗すると予想されます。

- 最初の `CommissionRate`は負または>` MaxRate`のいずれかです
- `CommissionRate`は過去24時間以内にすでに更新されています
- `CommissionRate`は> `MaxChangeRate`です
- 説明フィールドが大きすぎます

このメッセージは、更新された `Validator`オブジェクトを格納します。

## MsgDelegate

このメッセージ内で、委任者はコインを提供し、その見返りに、 `Delegation.Shares`に割り当てられたバリデーター(新しく作成された)委任者共有の一部を受け取ります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L22-L24

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L81-L90

このメッセージは、次の場合に失敗すると予想されます。

- バリデーターが存在しません
- `Amount``Coin`の金種は `params.BondDenom`で定義されたものとは異なります
- 為替レートが無効です。つまり、バリデーターにはトークンがありませんが(スラッシュのため)、発行済みの株式があります
- 委任された量が、許可されている最小の委任より少ない

指定されたアドレスの既存の `Delegation`オブジェクトがまだ存在しない場合は、このメッセージの一部として作成されます。それ以外の場合は、既存の` Delegation`オブジェクトが作成されます。
[委任]が更新され、新しく受け取った株式が含まれるようになりました。

委任者は、現在の為替レートで新たに作成された株式を受け取ります。
為替レートは、バリデーター内の既存のシェアの数を現在委任されているトークンの数で割ったものです。

バリデーターは `ValidatorByPower`インデックスで更新され、委任は` Validators`インデックスのバリデーターオブジェクトで追跡されます。

投獄されたバリデーターに委任することは可能ですが、唯一の違いは、投獄が解除されるまで電力インデックスに追加されないことです。

！[委任シーケンス](../../../docs/uml/svg/delegation_sequence.svg)

## MsgUndelegate

`MsgUndelegate`メッセージを使用すると、委任者はトークンをバリデーターから削除できなくなります。。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L30-L32

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L112-L121

此消息返回一个响应，其中包含取消委派的完成时间:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L123-L126

このメッセージは、次の場合に失敗すると予想されます。

- 委任は存在しません
- バリデーターが存在しません
- 代表団は `金額`の価値よりも少ないシェアを持っています
- 既存の `UnbondingDelegation`には、` params.MaxEntries`で定義されている最大エントリがあります
- `Amount`の金額は `params.BondDenom`で定義されたものとは異なります

このメッセージが処理されると、次のアクションが発生します。

- バリデーターの `DelegatorShares`と委任の` Shares`は、どちらもメッセージ `SharesAmount`によって削減されます。
- 株式のトークン価値を計算し、バリデーター内に保持されているトークンの量を削除します
- バリデーターが次の場合、削除されたトークンを使用します。
    - `Bonded`-それらを `UnbondingDelegation`のエントリに追加します(存在しない場合は` UnbondingDelegation`を作成します)。完了時間は現在の時刻から完全な非結合期間です。プールの共有を更新して、BondedTokenを減らし、共有のトークンに相当するNotBondedTokenを増やします。
    - `Unbonding`-それらを `UnbondingDelegation`のエントリに追加します(存在しない場合は` UnbondingDelegation`を作成します)。バリデーターと同じ完了時間( `UnbondingMinTime`)を使用します。
    - `Unbonded`-次にコインにメッセージ `DelegatorAddr`を送信します
- 委任に[共有]がもうない場合、委任オブジェクトはストアから削除されます
    - この状況では、委任がバリデーターの自己委任である場合は、バリデーターも投獄します。

![結合解除シーケンス](../../../docs/uml/svg/unbond_sequence.svg)

## MsgBeginRedelegate

redelegationコマンドを使用すると、委任者はバリデーターを即座に切り替えることができます。 ボンディング解除期間が経過すると、EndBlockerで再委任が自動的に完了します。。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L26-L28

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L95-L105

このメッセージは、再委任の完了時刻を含む応答を返します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L107-L110

このメッセージは、次の場合に失敗すると予想されます。

- 委任は存在しません
- 送信元または宛先のバリデーターが存在しません
- 代表団は `金額`の価値よりも少ないシェアを持っています
- ソースバリデーターには、成熟していない受信側の再委任があります(別名、再委任は推移的である可能性があります)
- 既存の `Redelegation`には、` params.MaxEntries`で定義されている最大エントリがあります
- `Amount``Coin`の金種は `params.BondDenom`で定義されたものとは異なります

このメッセージが処理されると、次のアクションが発生します。

- ソースバリデーターの `DelegatorShares`と委任` Shares`は両方ともメッセージ `SharesAmount`によって削減されます
- 共有のトークン価値を計算し、ソースバリデーター内に保持されているトークンの量を削除します。
- ソースバリデーターが次の場合:
    - `Bonded`-`Redelegation`にエントリを追加します(存在しない場合は `Redelegation`を作成します)。完了時間は、現在の時刻から完全に非結合期間です。プール共有を更新して、BondedTokenを減らし、共有のトークンに相当するNotBondedTokenを増やします(ただし、これは次のステップで効果的に取り消される可能性があります)。
    - `Unbonding`-バリデーターと同じ完了時間( `UnbondingMinTime`)で` Redelegation`にエントリを追加します(存在しない場合は `Redelegation`を作成します)。
    - `Unbonded`-このステップではアクションは不要です
- トークンの価値を宛先バリデーターに委任し、場合によってはトークンを結合状態に戻します。
- ソース委任に[共有]がもうない場合、ソース委任オブジェクトはストアから削除されます
    - この状況では、委任がバリデーターの自己委任である場合は、バリデーターも投獄します。。

![再委任シーケンスを開始します](../../../docs/uml/svg/begin_redelegation_sequence.svg) 