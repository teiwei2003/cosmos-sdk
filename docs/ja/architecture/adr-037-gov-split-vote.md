# ADR 037:ガバナンス分割投票

## 変更ログ

-2020/10/28:最初のドラフト

## 状態

受け入れられました

## 概要

ADRは、ガバナンスモジュールの変更を定義し、利害関係者が投票をいくつかの投票オプションに分割できるようにします。たとえば、議決権の70％を使用して賛成票を投じ、議決権の30％を反対票を投じることができます。

## 環境

現在、アドレスは1つのオプション(yes .no .abstention .veto)でのみ投票でき、すべての投票権はオプションの背後で使用されます。

ただし、アドレスを所有するエンティティは通常、個人ではない場合があります。たとえば、会社にはさまざまな方法で投票したいさまざまな利害関係者がいる可能性があるため、議決権の分配を許可することは理にかなっています。別のユースケースの例は交換です。多くの集中型取引所は通常、ユーザーのトークンの一部を保管しています。現在、[パススルー投票]を実施して、ユーザーにトークンに投票する権利を与えることはできません。ただし、このシステムにより、取引所はユーザーの投票設定に投票し、投票結果に比例してチェーンに投票することができます。

## 決定

投票構造を次のように変更します 

```
type WeightedVoteOption struct {
  Option string
  Weight sdk.Dec
}

type Vote struct {
  ProposalID int64
  Voter      sdk.Address
  Options    []WeightedVoteOption
}
```

下位互換性のために、 `MsgVote`を保持しながら、` MsgVoteWeighted`を導入しました。 

```
type MsgVote struct {
  ProposalID int64
  Voter      sdk.Address
  Option     Option
}

type MsgVoteWeighted struct {
  ProposalID int64
  Voter      sdk.Address
  Options    []WeightedVoteOption
}
```

`MsgVoteWeighted`構造の` ValidateBasic`には

1.すべての金利の合計は1.0に等しい
2.重複するオプションはありません

ガバナンスカウント機能は、投票のすべてのオプションを繰り返し、投票者の投票権の結果*このオプションのカウントに対する比率を追加します。

```
tally() {
    results := map[types.VoteOption]sdk.Dec

    for _, vote := range votes {
        for i, weightedOption := range vote.Options {
            results[weightedOption.Option] += getVotingPower(vote.voter) * weightedOption.Weight
        }
    }
}
```

複数選択投票を作成するために使用されるCLIコマンドは次のとおりです。 

```sh
simd tx gov vote 1 "yes=0.6,no=0.3,abstain=0.05,no_with_veto=0.05" --from mykey
```

To create a single-option vote a user can do either

```
simd tx gov vote 1 "yes=1" --from mykey
```

or

```sh
simd tx gov vote 1 yes --from mykey
```

to maintain backwards compatibility.

## 結果

### 下位互換性

-以前のVoteMsgタイプは変更されないため、WeightedVoteMsg関数をサポートする必要がない限り、顧客はプログラムを更新する必要はありません。
-州から投票構造を照会する場合、その構造は異なるため、すべての投票者とそれぞれの投票を表示したい顧客は、新しい形式と1人の投票者が分割投票を持つことができるという事実に対処する必要があります。
-tally関数をクエリした結果は、クライアントと同じAPIを持つ必要があります。

### ポジティブ

-複数の利害関係者を表す住所(通常は最大の住所の一部)の投票プロセスをより正確にすることができます。

### ネガティブ

-単純な投票よりも複雑なので、ユーザーに説明するのが難しい場合があります。 ただし、この機能はオプションであるため、これは大幅に軽減されます。

### ニュートラル

-ガバナンス集計機能の変化は比較的小さい。 