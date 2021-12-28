# 状態

## プール

プールは、債券建ての債券および非債券のトークン供給を追跡するために使用されます。

## LastTotalPower

LastTotalPowerは、前のエンドブロック中に記録された結合トークンの合計量を追跡します。
「Last」で始まるストアエントリは、EndBlockまで変更しないでください。

- LastTotalPower:`0x12 -> ProtocolBuffer(sdk.Int)`

## パラメータ

Paramsは、システムパラメータを格納し、ステーキングモジュールの全体的な機能を定義するモジュール全体の構成構造です。

- パラメータ:`Paramsspace("staking") -> legacy_amino(params)`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.1/proto/cosmos/staking/v1beta1/staking.proto#L230-L241

## バリデーター

バリデーターは、3つのステータスのいずれかを持つことができます

-`Unbonded`:バリデーターがアクティブセットにありません。 ブロックに署名したり、報酬を獲得したりすることはできません。 彼らは代表団を受け入れることができます。
-`Bonded`":バリデーターが十分な結合トークンを受け取ると、[`EndBlock`](./ 05_end_block.md＃validator-set-changes)中にアクティブセットに自動的に参加し、ステータスが`Bonded`に更新されます。
彼らはブロックに署名し、報酬を受け取っています。 彼らはさらに委任を受けることができます。
それらは、不正行為のためにスラッシュされる可能性があります。 委任を解除するこのバリデーターの委任者は、チェーン固有のパラメーターであるUnbondingTimeの期間を待機する必要があります。その間、トークンが結合されている期間中にこれらの違反がコミットされた場合、ソースバリデーターの違反に対してスラッシュ可能です。
-`Unbonding`:バリデーターが選択によって、またはスラッシュ、ジェイル、またはトゥームストーンのためにアクティブセットを離れると、すべての委任の結合が解除されます。 その後、すべての委任は、トークンが`BondedPool`からアカウントに移動される前に、UnbondingTimeを待機する必要があります。

バリデーターオブジェクトは、主に保存され、
`OperatorAddr`、バリデーターのオペレーターのSDKバリデーターアドレス。 スラッシュとバリデーターセットの更新に必要なルックアップを満たすために、バリデーターオブジェクトごとに2つの追加のインデックスが維持されます。 3番目の特別なインデックス(`LastValidatorPower`)も維持されますが、ブロック内のバリデーターレコードをミラーリングする最初の2つのインデックスとは異なり、各ブロック全体で一定のままです。 

- Validators:`0x21 | OperatorAddrLen(1バイト)| OperatorAddr-> ProtocolBuffer(validator)`
- ValidatorsByConsAddr：`0x22 | ConsAddrLen(1バイト)| ConsAddr-> OperatorAddr`
- ValidatorsByPower：`0x23 | BigEndian(ConsensusPower)| OperatorAddrLen(1バイト)| OperatorAddr-> OperatorAddr`
- LastValidatorsPower：`0x11 | OperatorAddrLen(1バイト)| OperatorAddr-> ProtocolBuffer(ConsensusPower)`

`Validators`はプライマリインデックスです。これにより、各オペレーターが関連付けられたバリデーターを1つだけ持つことができ、そのバリデーターの公開鍵は将来変更される可能性があります。 委任者は、公開鍵の変更を気にすることなく、バリデーターの不変演算子を参照できます。

`ValidatorByConsAddr`は、スラッシュのルックアップを可能にする追加のインデックスです。
Tendermintが証拠を報告するとき、それはバリデーターアドレスを提供するので、このマップはオペレーターを見つけるために必要です。`ConsAddr`は、バリデーターの`ConsPubKey`から取得できるアドレスに対応していることに注意してください。

`ValidatorsByPower`は、現在アクティブなセットをすばやく判別するための潜在的なバリデーターのソートされたリストを提供する追加のインデックスです。 ここで、ConsensusPowerはデフォルトでvalidator.Tokens/10^6です。`Jailed`がtrueであるすべてのバリデーターは、このインデックス内に格納されないことに注意してください。

`LastValidatorsPower`は、最後のブロックの結合されたバリデーターの履歴リストを提供する特別なインデックスです。 このインデックスはブロック中は一定のままですが、[`EndBlock`](./ 05_end_block.md)で行われるバリデーターセットの更新プロセス中に更新されます。

各バリデーターの状態は、`Validator`構造体に格納されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L65-L99

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L24-L63

## 委任

委任は、`DelegatorAddr`(委任者のアドレス)と`ValidatorAddr`を組み合わせることによって識別されます。委任者は、ストアで次のようにインデックス付けされます。

- 委任：`0x31 | DelegatorAddrLen(1バイト)| DelegatorAddr | ValidatorAddrLen(1バイト)| ValidatorAddr-> ProtocolBuffer(delegation)`

ステーク保有者は、コインをバリデーターに委任することができます。この状況では、彼らの資金は「委任」データ構造で保持されます。これは1人の委任者によって所有され、1人の検証者の共有に関連付けられています。
トランザクションの送信者は、債券の所有者です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L159-L170

### 委任者の共有

1人のトークンをバリデーターに委任すると、動的為替レートに基づいて、バリデーターに委任されたトークンの総数とこれまでに発行されたシェアの数から次のように計算された委任株の数が発行されます。

`Shares per Token = validator.TotalShares() / validator.Tokens()`

受け取ったシェアの数だけがDelegationEntryに保存されます。 次に委任者が委任を解除すると、委任者が受け取るトークンの量は、現在保有している株式数と逆為替レートから計算されます。

`Tokens per Share = validator.Tokens() / validatorShares()`

これらの「共有」は、単なる会計メカニズムです。 それらは代替可能な資産ではありません。 このメカニズムの理由は、スラッシュに関するアカウンティングを簡素化するためです。
すべての委任エントリのトークンを繰り返しスラッシュするのではなく、代わりに、バリデーターの結合されたトークンの合計をスラッシュして、発行された各委任共有の値を効果的に減らすことができます。 

## 結合解除委任

「委任」の共有は結合解除できますが、ビザンチンの動作が検出された場合に共有を減らすことができる「結合解除委任」としてしばらくの間存在する必要があります。

`UnbondingDelegation`は、ストアで次のようにインデックス付けされます。

- UnbondingDelegation：`0x32 | DelegatorAddrLen(1バイト)| DelegatorAddr | ValidatorAddrLen(1バイト)| ValidatorAddr-> ProtocolBuffer(unbondingDelegation)`
- UnbondingDelegationsFromValidator：`0x33 | ValidatorAddrLen(1バイト)| ValidatorAddr | DelegatorAddrLen(1バイト)| DelegatorAddr-> nil`

ここでの最初のマップはクエリで使用され、特定の委任者のすべての非結合委任を検索します。2番目のマップはスラッシュで使用され、特定のバリデーターに関連付けられている、スラッシュが必要なすべての非結合委任を検索します。

UnbondingDelegationオブジェクトは、結合解除が開始されるたびに作成されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L172-L198

## 再委任

「委任」に相当する結合トークンは、ソースバリデーターから別のバリデーター(宛先バリデーター)に即座に再委任できます。
ただし、これが発生した場合は、`Redelegation`オブジェクトで追跡する必要があります。これにより、トークンがソースバリデーターによってコミットされたビザンチンフォールトに寄与した場合、共有を大幅に削減できます。

`Redelegation`は、ストアで次のようにインデックス付けされます。

- 再委任：`0x34 | DelegatorAddrLen(1バイト)| DelegatorAddr | ValidatorAddrLen(1バイト)| ValidatorSrcAddr | ValidatorDstAddr-> ProtocolBuffer(redelegation)`
- RedelegationsBySrc：`0x35 | ValidatorSrcAddrLen(1バイト)| ValidatorSrcAddr | ValidatorDstAddrLen(1バイト)| ValidatorDstAddr | DelegatorAddrLen(1バイト)| DelegatorAddr-> nil`
- RedelegationsByDst：`0x36 | ValidatorDstAddrLen(1バイト)| ValidatorDstAddr | ValidatorSrcAddrLen(1バイト)| ValidatorSrcAddr | DelegatorAddrLen(1バイト)| DelegatorAddr-> nil`

ここでの最初のマップは、特定の委任者のすべての再委任を検索するためのクエリに使用されます。 2番目のマップは`ValidatorSrcAddr`に基づくスラッシュに使用され、3番目のマップは`ValidatorDstAddr`に基づくスラッシュに使用されます。

再委任オブジェクトは、再委任が発生するたびに作成されます。 「再委任ホッピング」を防ぐために、次の状況では再委任が発生しない場合があります。

- (再)委任者はすでに別の未熟な再委任が進行中です
   バリデーターへの宛先を指定します(これを「バリデーターX」と呼びましょう)
- そして、(再)委任者は_new_再委任を作成しようとしています
   ここで、この新しい再委任のソースバリデーターは`ValidatorX`です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L200-L228

## Queues

すべてのキューオブジェクトはタイムスタンプで並べ替えられます。 キュー内で使用される時間は、最初に最も近いナノ秒に丸められ、次にソートされます。 使用されるソート可能な時間形式は、RFC3339Nanoのわずかな変更であり、形式文字列`" 2006-01-02T15：04：05.000000000 "`を使用します。 特にこのフォーマット:

- 右はすべてゼロを埋めます
- タイムゾーン情報を削除します(UTCを使用)

すべての場合において、保存されたタイムスタンプはキュー要素の成熟時間を表します。 

### UnbondingDelegationQueue

結合解除された委任の進行状況を追跡する目的で、結合解除された委任キューが保持されます。

- UnbondingDelegation：`0x41 | format(time)-> [] DVPair`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L123-L133

### RedelegationQueue

結合解除されたバリデーターの進行状況を追跡する目的で、バリデーターキューが保持されます。

- RedelegationQueue:`0x42 | format(time) -> []DVVTriplet`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L140-L152

### ValidatorQueue

結合解除されたバリデーターの進行状況を追跡する目的で、バリデーターキューが保持されます。。

- ValidatorQueueTime:`0x43 |格式(时间)-> []sdk.ValAddress`

各キーとして格納されているオブジェクトは、バリデーターオブジェクトにアクセスできるバリデーターオペレーターアドレスの配列です。 通常、特定のタイムスタンプに関連付けられるバリデーターレコードは1つだけであると予想されますが、同じ場所のキューに複数のバリデーターが存在する可能性があります。

## 履歴情報

履歴情報オブジェクトは、ステーキングキーパーがステーキングモジュールパラメータ`HistoricalEntries`によって定義された`n`最新の履歴情報を保持するように、各ブロックで保存およびプルーニングされます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L15-L22

各BeginBlockで、ステーキングキーパーは現在のヘッダーとコミットしたバリデーターを保持します
`HistoricalInfo`オブジェクトの現在のブロック。 バリデーターは、アドレスに基づいて並べ替えられ、次のことを確認します。
それらは決定論的な順序になっています。
最も古いHistoricalEntriesは、パラメーターで定義された数の
履歴エントリ。