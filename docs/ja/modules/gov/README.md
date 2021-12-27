# `gov`

## 概要

このホワイトペーパーでは、最初に使用されたCosmos-SDKのガバナンスモジュールについて説明します。
[Cosmos Whitepaper]（https://cosmos.network/about/whitepaper）で説明されています
2016年6月。

このモジュールにより、Cosmos-SDKベースのブロックチェーンがオンチェーンガバナンスをサポートできるようになります
システム。 このシステムでは、チェーンのネイティブステーキングトークンの所有者が投票できます
1トークン1票ベースの提案。 次は、モジュールの機能のリストです
現在サポートしています：

- **プロポーザルの提出：**ユーザーはデポジットでプロポーザルを提出できます。 一度最低保証金に達し、提案が投票期間に入ります
- **投票：**参加者はMinDepositに到達した提案に投票できます
- **継承とペナルティ：**委任者は、次の場合にバリデーターの投票を継承します
彼らは自分自身に投票しません。
- **請求デポジット：**プロポーザルにデポジットしたユーザーは、提案が承認された場合、または提案が投票期間に入ったことがない場合は、保証金。

このモジュールは、Cosmosネットワークの最初のハブであるCosmosハブで使用されます。
将来追加される可能性のある機能については、[Future Improvements](05_future_improvements.md)で説明されています。

## 目次

次の仕様では、ネイティブステーキングトークンとして*ATOM*を使用しています。モジュール*ATOM*をネイティブに置き換えることで、任意のプルーフオブステークブロックチェーンに適合させることができますチェーンのステーキングトークン

1. **[Concepts](01_concepts.md)**
    - [Proposal submission](01_concepts.md#proposal-submission)
    - [Vote](01_concepts.md#vote)
    - [Software Upgrade](01_concepts.md#software-upgrade)
2. **[State](02_state.md)**
    - [Parameters and base types](02_state.md#parameters-and-base-types)
    - [Deposit](02_state.md#deposit)
    - [ValidatorGovInfo](02_state.md#validatorgovinfo)
    - [Proposals](02_state.md#proposals)
    - [Stores](02_state.md#stores)
    - [Proposal Processing Queue](02_state.md#proposal-processing-queue)
3. **[Messages](03_messages.md)**
    - [Proposal Submission](03_messages.md#proposal-submission)
    - [Deposit](03_messages.md#deposit)
    - [Vote](03_messages.md#vote)
4. **[Events](04_events.md)**
    - [EndBlocker](04_events.md#endblocker)
    - [Handlers](04_events.md#handlers)
5. **[Future Improvements](05_future_improvements.md)**
6. **[Parameters](06_params.md)**
7. **[Client](07_client.md)**
    - [CLI](07_client.md#cli)
    - [gRPC](07_client.md#grpc)
    - [REST](07_client.md#rest)