# モジュールリスト

CosmosSDKアプリケーションで使用できる本番レベルのモジュールとそれぞれのドキュメントは次のとおりです。

- [Auth](auth/)-CosmosSDKアプリケーションのアカウントおよびトランザクション認証。
- [Authz](authz/)-他のアカウントに代わって操作を実行するようにアカウントを承認します。
- [Bank](bank/)-トークン伝達関数。
- [Capability](capability/)-オブジェクト機能の実現。
- [Crisis](crisis/)-特定の状況(たとえば、不変条件が壊れている場合)でブロックチェーンを停止します。
- [Distribution](distribution/)-コスト配分と住宅ローントークン供給配分。
- [Epoching](epoching/)-モジュールが特定のブロックの高さで実行するためにメッセージをキューに入れることを許可します。
- [Evidence](evidence/)-二重署名や不正行為などの証拠処理。
- [Feegrant](feegrant/)-トランザクションを実行するための手数料を付与します。
- [Governance](gov/)-チェーン上の提案と投票。
- [Mint](mint/)-新しい住宅ローントークンユニットを作成します。
- [Params](params/)-グローバルに利用可能なパラメータストレージ。
- [Slashing](slashing/)-ベリファイアの罰メカニズム。
- [Staking](staking/)-パブリックブロックチェーンのプルーフオブステークレイヤー。
- [Upgrade](upgrade/)-ソフトウェアアップグレードの処理と調整。

構築モジュールプロセスの詳細については、[構築モジュールリファレンスドキュメント](/building-modules/intro.html)にアクセスしてください。

## 国際商業銀行

SDKのIBCモジュールは、[独自のリポジトリ](https://github.com/cosmos/ibc-go)に移動されました。