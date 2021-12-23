# IBC クライアント開発者向けアップグレードガイド

カスタムIBCクライアントのアップグレード機能を実装する方法を学びます。 {まとめ}

[README]（./ README.md）で説明したように、価値の高いIBCのお客様は、IBCエコシステムへの損傷を回避するために、基盤となるチェーンをアップグレードできることが重要です。 したがって、IBCクライアント開発者は、チェーンのアップグレード中でもクライアントが接続とチャネルを維持できるように、アップグレード機能を実装することを望んでいます。

IBCプロトコルを使用すると、クライアント実装は、アップグレードされたクライアントステータス、アップグレードされたコンセンサスステータス、および各プルーフを前提として、クライアントをアップグレードするためのパスを提供できます。 

```go
// Upgrade functions
// NOTE: proof heights are not included as upgrade to a new revision is expected to pass only on the last
// height committed by the current revision. Clients are responsible for ensuring that the planned last
// height of the current revision is somehow encoded in the proof verification process.
// This is to ensure that no premature upgrades occur, since upgrade plans committed to by the counterparty
// may be cancelled or modified before the last planned height.
VerifyUpgradeAndUpdateState(
    ctx sdk.Context,
    cdc codec.BinaryCodec,
    store sdk.KVStore,
    newClient ClientState,
    newConsState ConsensusState,
    proofUpgradeClient,
    proofUpgradeConsState []byte,
) (upgradedClient ClientState, upgradedConsensus ConsensusState, err error)
```

クライアントは、アップグレードされたクライアントとアップグレードされたコンセンサス状態が使用するメルケルパスを事前に知っている必要があることに注意してください。 アップグレードが発生した高さも証明書にコード化する必要があります。 Tendermintクライアントの実装は、ClientState自体にUpgradePathを含めることでこれを実現します。これは、アップグレードの高さとともに使用され、クライアント状態とコンセンサス状態を送信するためのMerkelパスを構築します。

開発者は、古いチェーンの最後の高さ送信の前に `UpgradeClientMsg`が渡されないようにする必要があり、チェーンがアップグレードされた後、` UpgradeClientMsg`はすべてのカウンターパーティクライアントで1回だけ渡される必要があります。

開発者は、新しいクライアントがすべての新しいクライアントパラメータを採用することを確認する必要があります。これらのパラメータは、個々のクライアントがカスタマイズできるクライアントパラメータ（顧客クライアント選択パラメータ）を維持しながら、チェーンの各有効なライトクライアント（チェーン選択パラメータ）で統一する必要があります。 ）以前のバージョンのクライアントから。

アップグレードは、IBCセキュリティモデルに準拠する必要があります。 IBCは、正直な中継者の正確さの仮定に依存していません。 したがって、ユーザーはクライアントの正確性とセキュリティを維持するためにリピーターに依存するべきではありません（ただし、リピーターをアクティブに保つには正直なリピーターが必要です）。 リピーターは新しい「ClientState」を作成するときにクライアントパラメーターの任意のセットを選択できますが、ユーザーはセキュリティと正確さの要件に合ったリピーターによって作成されたクライアントをいつでも選択できるため、これはセキュリティモデルにも適用されます。以下の場合に必要なパラメータ、そのようなクライアントは存在しません。

ただし、既存のクライアントをアップグレードする場合、多くのユーザーがすでにクライアントの特定のパラメーターに依存していることを覚えておく必要があります。 これらのパラメーターが選択されると、アップグレードリピーターがこれらのパラメーターを自由に選択できるようにすることはできません。 クライアントに依存するユーザーは、同じレベルのセキュリティを維持するためにアップグレードされたリピーターに依存する必要があるため、これはセキュリティモデルに違反します。 したがって、開発者は、チェーンのアップグレードによってこれらのパラメーターが変更されたときに、クライアントがチェーンによって指定されたパラメーターをアップグレードできるようにする必要があります（Tendermintクライアントの例には、 `UnbondingPeriod`、` ChainID`、 `UpgradePath`などが含まれます）。また、必ず送信してください。 `UpgradeClientMsg`のリピーターは、ユーザーが依存するクライアントによって選択されたパラメーターを変更できません（Tendermintクライアントの例には、` TrustingPeriod`、 `TrustLevel`、` MaxClockDrift`などがあります）。

開発者は、チェーンの各有効なライトクライアントの統合クライアントパラメータ（チェーン選択パラメータ）と、個々のクライアントのカスタマイズ可能なクライアントパラメータ（クライアント選択パラメータ）を区別する必要があります。この違いは、 `ClientStateのクライアント状態にとって重要であるためです。 `インターフェースに` ZeroCustomFields`メソッドを実装するために必要です。

```go
// Utility function that zeroes out any client customizable fields in client state
// Ledger enforced fields are maintained while all custom fields are zero values
// Used to verify upgrades
ZeroCustomFields() ClientState
```

カウンターパーティクライアントは、チェーンによって送信された「UpgradedClient」のすべてのチェーン選択パラメータを使用し、すべての古いクライアント選択パラメータを保持することにより、安全にアップグレードできます。 これにより、正直なリピーターに依存せずにチェーンを安全にアップグレードできますが、場合によっては、新しいチェーン選択パラメーターが古いクライアント選択パラメーターと競合すると、最終的な「ClientState」が無効になる可能性があります。 アップグレードチェーンによって「UnbondingPeriod」（チェーン選択）がカウンターパーティ顧客「TrustingPeriod」（クライアント選択）の期間よりも短くなる場合、これはTendermintクライアントの場合に発生する可能性があります。 このような状況は、開発者が明確に文書化して、この問題を防ぐためにどのアップグレードを回避する必要があるかをチェーンが認識できるようにする必要があります。 最終的にアップグレードされたクライアントは、クライアントが無効な「ClientState」にアップグレードされないことを確認するために、戻る前に「VerifyUpgradeAndUpdateState」で検証する必要があります。