# `x/bank`

## 概要

このドキュメントでは、CosmosSDKのバンキングモジュールを指定します.

銀行モジュールは、マルチアセットコイン間の転送を処理する責任があります
アカウントと追跡が異なる方法で機能する必要がある特別な状況の偽の転送
特定の種類のアカウントを持っている（特に帰属の承認/承認解除）
アカウント）. 異なるセキュリティ機能を持ついくつかのインターフェースを公開します
ユーザーのバランスを変更する必要がある他のモジュールとの相互作用.

さらに、銀行モジュールは、完全なクエリサポートを追跡して提供します.
アプリケーションで使用されるすべての資産の供給.

このモジュールは、Cosmosハブで使用されます.

## supply

`supply`関数：

- チェーン内のコインの総供給量を受動的に追跡し、
- モジュールが「コイン」を保存/操作するためのモードを提供し、
- チェーンの総供給量を確認するための定期的なチェックを導入します.

### Total Supply

ネットワークの「supply」の合計は、
アカウント. 総供給量は、「コイン」が鋳造されるたびに更新されます（例：一部として）
インフレメカニズムの）または燃やされた（例：スラッシュまたはガバナンスの場合
提案は拒否されます）.

## Module Accounts

プロビジョニング機能により、新しいタイプの「auth.Account」が導入されます.
特別な状況下でトークンを割り当てたり、トークンを作成または破棄したりするためのモジュール. 基地で
これらのモジュールアカウントがトークンを送受信できるレベル
`auth.Account`sおよびその他のモジュールアカウント. このデザインは以前のものを置き換えます
トークンを保持するために、モジュールが着信トークンを書き込む代替設計
その後、送信者のアカウントからのトークンが内部で追跡されます. 後、
トークンを送信するには、モジュールがトークンを効果的に作成する必要があります
ターゲットアカウント内. 新しい設計により、間のロジックの重複が排除されます
このアカウンティングを実行するモジュール.

`ModuleAccount`インターフェースは次のように定義されています.

```go
type ModuleAccount interface {
  auth.Account               // same methods as the Account interface

  GetName() string           // name of the module; used to obtain the address
  GetPermissions() []string  // permissions of module account
  HasPermission(string) bool
}
```

> **警告！**
>資金の直接または間接の送金を許可するモジュールまたはメッセージ処理プログラムは、これらの資金がモジュールアカウントに送金されないことを明確に保証する必要があります（許可されていない場合）.

`Keeper`の提供により、認証` Keeper`の新しいラッパー関数も導入されます
そして、できるようにするために「ModuleAccount」に関連付けられた銀行「Keeper」
到着：

- `Name`を指定して `ModuleAccount`を取得および設定します.
- 他の `ModuleAccount`または標準の` Account`の間でコインを送信します
   （ `BaseAccount`または` VestingAccount`） `Name`のみを渡します.
- `ModuleAccount`の `Mint`または` Burn`コイン（その権限に従う）.

### 許可

各 `ModuleAccount`には異なる権限のセットがあり、異なる権限を提供します
特定の操作を実行する対象の能力. 許可が必要です
供給「キーパー」を作成するときに登録して、毎回
`ModuleAccount`は許可された関数を呼び出し、` Keeper`は見つけることができます
特定のアカウントの許可と操作を実行するかどうか.

使用可能な権限は次のとおりです.

-`Minter`：モジュールが特定の数のコインをミントできるようにします.
-`Burner`：モジュールが特定の数のコインを燃やすことができるようにします.
-`ステーキング `：モジュールが特定の数のコインを委任およびキャンセルできるようにします.

## コンテンツ

1. **[State]（01_state.md）**
2. **[キーパー]（02_keepers.md）**
   - [一般的なタイプ]（02_keepers.md＃common-types）
   - [BaseKeeper]（02_keepers.md＃basekeeper）
   - [SendKeeper]（02_keepers.md＃sendkeeper）
   - [ViewKeeper]（02_keepers.md＃viewkeeper）
3. **[メッセージ]（03_messages.md）**
   - [MsgSend]（03_messages.md＃msgsend）
4. **[イベント]（04_events.md）**
   - [ハンドラー]（04_events.md＃handlers）
5. **[パラメータ]（05_params.md）**
6. **[クライアント]（06_client.md）**
   - [CLI]（06_client.md＃cli）
   - [gRPC]（06_client.md＃grpc）