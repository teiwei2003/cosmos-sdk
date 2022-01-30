# アーキテクチャ決定記録(ADR)

これは、すべての高レベルのアーキテクチャ上の決定がCosmos-SDKに記録される場所です。

アーキテクチャー決定(** AD **)は、アーキテクチャー的に重要な機能要件または非機能要件を解決するために使用されるソフトウェア設計の選択です。
アーキテクチャ上重要な要件(** ASR **)は、ソフトウェアシステムのアーキテクチャと品質に測定可能な影響を与える要件です。
アーキテクチャ上の決定レコード(** ADR **)は、個人的なメモや議事録を作成するときによく行われることなど、単一のADをキャプチャします。プロジェクトで作成および維持されるADRのセットは、その決定ログを構成します。これらはすべて、建設知識管理(AKM)の対象です。

ADRの概念について詳しくは、この[ブログ投稿](https://product.reverb.com/documenting-architecture-decisions-the-reverb-way-a3563bb24bd0#.78xhdix6t)を参照してください。

## 基本的

ADRは、新しい機能設計と新しいプロセスを提案し、特定の問題に関するコミュニティの意見を収集し、設計上の決定を記録するための主要なメカニズムになることを目的としています。
ADRは以下を提供する必要があります。

-関連する目標の背景と現在の状況
-目標を達成するために提案された変更
-長所と短所の概要
- 参照する
-変更ログ

ADRと仕様の違いに注意してください。 ADRは、コンテキスト、直感、推論、および
構造を変更する理由、または何かの構造
新着。仕様は、すべてのコンテンツのより圧縮された単純化された要約を提供します。
今日は立っています。

欠落している決定を見つけた場合は、ディスカッションに電話し、ここに新しい決定を記録してから、一致するようにコードを変更してください。

## 新しいADRを作成する

[PROCESS](..PROCESS.md)についてお読みください。

#### RFC2119キーワードを使用する

ADRを作成するときは、RFCを作成するときと同じベストプラクティスに従ってください。 RFCを作成するとき、キーワードは仕様の要件を表すために使用されます。これらの単語は通常大文字で表記されます:[MUST]、[MUST NOT]、[REQUIRED]、[SHALL]、[SHALL NOT]、[SHOULD]、[SHOULD NOT]、[RECOMMENDED]、[MAY]、[OPTIONAL]。それらは[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119)で説明されているように説明されます。

## ADRディレクトリ

### 受け入れる

-[ADR 002:SDKドキュメント構造](..adr-002-docs-structure.md)
-[ADR 004:分割金種キー](..adr-004-split-denomination-keys.md)
-[ADR 006:シークレットストアの交換](..adr-006-secret-store-replacement.md)
-[ADR 009:エビデンスモジュール](..adr-009-evidence-module.md)
-[ADR 010:モジュラーAnteHandler](..adr-010-modular-antehandler.md)
-[ADR 019:プロトコルバッファ状態エンコーディング](..adr-019-protobuf-state-encoding.md)
-[ADR 020:プロトコルバッファトランザクションエンコーディング](..adr-020-protobuf-transaction-encoding.md)
-[ADR 021:プロトコルバッファクエリエンコーディング](..adr-021-protobuf-query-encoding.md)
-[ADR 023:プロトコルバッファの命名とバージョン管理](..adr-023-protobuf-naming.md)
-[ADR 029:料金付与モジュール](..adr-029-fee-grant-module.md)
-[ADR 030:メッセージ認証モジュール](..adr-030-authz-module.md)
-[ADR 031:Protobufメッセージサービス](..adr-031-msg-service.md)

### 提案

-[ADR 003:動的機能ストア](..adr-003-dynamic-capability-store.md)
-[ADR 011:ジェネシスアカウントのプロモート](..adr-011-generalize-genesis-accounts.md)
-[ADR 012:州のアクセサー](..adr-012-state-accessors.md)
-[ADR 013:メトリクス](..adr-013-metrics.md)
-[ADR 016:検証者のコンセンサスキーローテーション](..adr-016-validator-consensus-key-rotation.md)
-[ADR 017:履歴ヘッダーモジュール](..adr-017-historical-header-module.md)
-[ADR 018:延長可能な投票期間](..adr-018-extendable-voting-period.md)
-[ADR 022:カスタムbaseappパニック処理](..adr-022-custom-panic-handling.md)
-[ADR 024:コインメタデータ](..adr-024-coin-metadata.md)
-[ADR 027:Deterministic Protobuf Serialization](..adr-027-deterministic-protobuf-serialization.md)
-[ADR 028:公開鍵アドレス](..adr-028-public-key-addresses.md)
-[ADR 032:型付きイベント](..adr-032-typed-events.md)
-[ADR 033:モジュール間RPC](..adr-033-protobuf-inter-module-comm.md)
-[ADR 035:Rosetta APIサポート](..adr-035-rosetta-api-support.md)
-[ADR 037:ガバナンス分割投票](..adr-037-gov-split-vote.md)
-[ADR 038:状態リスニング](..adr-038-state-listening.md)
-[ADR 039:Epoched Stakeing](..adr-039-epoched-staking.md)
-[ADR 040:ストレージとSMTの状態のコミットメント](..adr-040-storage-and-smt-state-commitments.md)
-[ADR 046:モジュールパラメータ](..adr-046-module-params.md)

### 下書き

-[ADR 044:Protobuf定義更新ガイド](..adr-044-protobuf-updates-guidelines.md) 