# 概念

## プラン

`x/upgrade`モジュールは、ライブアップグレードが実行されるようにスケジュールされている` Plan`タイプを定義します。 「計画」は、特定のブロックの高さでスケジュールできます。
「プラン」は、適切なアップグレード「ハンドラー」（以下を参照）とともに（凍結された）リリース候補が合意されると作成されます。「プラン」の「名前」は特定の「ハンドラー」に対応します。通常、「計画」はガバナンス提案を通じて作成されます
プロセスは、投票されて合格した場合、スケジュールされます。
`Plan`の` Info`には、アップグレードに関するさまざまなメタデータが含まれる場合があります。通常、バリデーターが自動的にアップグレードできるgit commitなど、チェーン上に含まれるアプリケーション固有のアップグレード情報です。

### サイドカープロセス

アプリケーションバイナリを実行しているオペレーターがサイドカープロセスも実行して、バイナリの自動ダウンロードとアップグレードを支援する場合、 `Info`によりこのプロセスをシームレスにすることができます。 つまり、`x/upgrade`モジュールは[cosmosdUpgradeable Binary Specification（https://github.com/regen-network/cosmosd#upgradeable-binary-specification）仕様を満たし、オプションで` cosmosd`を使用して完全に自動化できます。 アップグレードノード演算子のプロセス。
`Info`フィールドに必要な情報を入力することにより、バイナリは自動的にダウンロードできます。 [こちら]（https://github.com/regen-network/cosmosd#auto-download）を参照してください

詳細については。

```go
type Plan struct {
  Name   string
  Height int64
  Info   string
}
```

## Handler

`x/upgrade`モジュールは、メジャーバージョンXからメジャーバージョンYへのアップグレードを容易にします。
これを実現するには、ノードオペレーターはまず、現在のバイナリを、新しいバージョンYに対応する`Handler`を持つ新しいバイナリにアップグレードする必要があります。
このバージョンは、コミュニティ全体で完全にテストおよび承認されていることを前提としています。
この`Handler`は、新しいバイナリYがチェーンを正常に実行する前に発生する必要のある状態の移行を定義します。 当然、この `Handler`はアプリケーション固有であり、モジュールごとに定義されていません。`Handler`の登録は、アプリケーションの` Keeper＃SetUpgradeHandler`を介して行われます。

```go
type UpgradeHandler func(Context, Plan, VersionMap) (VersionMap, error)
```

各 `EndBlock`の実行中に、` x / upgrade`モジュールは実行する必要のある `Plan`が存在するかどうかをチェックします（その高さでスケジュールされます）。 その場合、対応する `Handler`が実行されます。`Plan`が実行されることが期待されているが、` Handler`が登録されていない場合`。
または、バイナリのアップグレードが早すぎると、ノードは正常にパニックになり、終了します。

## ストレージローダー

`x/upgrade`モジュールは、アップグレードの一部としてストアの移行も容易にします。
`StoreLoader`は、新しいバイナリがチェーンを正常に実行する前に発生する必要のある移行を設定します。
この`StoreLoader`もアプリケーション固有であり、モジュールごとに定義されていません。 この`StoreLoader`の登録は、アプリケーションの`app＃SetStoreLoader`を介して行われます。 

```go
func UpgradeStoreLoader (upgradeHeight int64, storeUpgrades *store.StoreUpgrades) baseapp.StoreLoader
```

計画されたアップグレードがあり、アップグレードの高さに達した場合、古いバイナリはパニックになる前にディスクに「計画」を書き込みます。

この情報は、 `StoreUpgrades`が正しい高さと予想されるアップグレードでスムーズに行われるようにするために重要です。
これにより、新しいバイナリが再起動時に毎回 `StoreUpgrades`を複数回実行する可能性がなくなります。
また、同じ高さで複数のアップグレードが計画されている場合、 `Name`は、これらの` StoreUpgrades`が計画されたアップグレードハンドラーでのみ実行されるようにします。

## 提案

通常、「計画」は、「SoftwareUpgradeProposal」を介したガバナンスを通じて提案および提出されます。
この提案は、標準的なガバナンスプロセスを規定しています。 提案が通過すると、特定の「ハンドラー」を対象とする「計画」が永続化され、スケジュールされます。
新しいプロポーザルの `Plan.Height`を更新することで、アップグレードを遅らせたり早めたりすることができます。

```go
type SoftwareUpgradeProposal struct {
  Title       string
  Description string
  Plan        Plan
}
```

### アップグレード提案のキャンセル

アップグレードの提案はキャンセルできます。
`CancelSoftwareUpgrade`プロポーザルが存在します
タイプ。投票して合格することができ、スケジュールされたアップグレード `Plan`を削除します。
もちろん、これには、投票のための時間を確保するために、アップグレード自体がアップグレード自体のかなり前に悪い考えであることがわかっている必要があります。

そのような可能性が望まれる場合、アップグレードの高さは
アップグレード提案の最初から `2 *（VotingPeriod + DepositPeriod）+（SafetyDelta）`。
`SafetyDelta`は、成功から得られる時間です。
アップグレードの提案とそれが悪い考えであるという認識（外部の社会的コンセンサスのため）。

`VotingPeriod`が` SoftwareUpgradeProposal`の後に終了する限り、元の `SoftwareUpgradeProposal`がまだ投票されている間に、` CancelSoftwareUpgrade`プロポーザルを作成することもできます。