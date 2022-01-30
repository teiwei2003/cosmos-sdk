# ICS 030:コスモス署名メッセージ

> TODO:有効なICS番号に置き換えて、新しい場所に移動する可能性があります。

* [変更ログ](#changelog)
* [要約](#abstract)
* [暫定版](#preliminary)
* [仕様](#specification)
* [将来の適応](#future-adaptations)
* [API](#api)
* [参照](#references)

## 状態

提案。

## 変更ログ

## 概要

オフチェーンでメッセージに署名する機能を持つことは、基本的な側面であることが証明されています
ほぼすべてのブロックチェーンの。 オフチェーンでメッセージに署名するという概念には多くの
計算コストの節約やトランザクションの削減などの追加のメリット
スループットとオーバーヘッド。 コスモスの文脈の中で、主要なもののいくつか
このようなデータに署名するアプリケーションには、以下が含まれますが、これに限定されません。
検証者の身元を証明する暗号化された安全で検証可能な手段と
おそらくそれを他のフレームワークや組織と関連付けることです。 加えて、
元帳または同様のHSMデバイスを使用してCosmosメッセージに署名する機能を備えています。

メッセージをハッシュ、署名、および検証するための標準化されたプロトコル
CosmosSDKおよびその他のサードパーティ組織による実装が必要です。 そのような
標準化されたプロトコルは、以下をサブスクライブします。

* 人間が読み取れる、機械で検証できる型付き構造化データの仕様が含まれています
* 構造化データの決定論的かつ単射エンコーディングのためのフレームワークが含まれています
* 暗号化されたセキュアハッシュおよび署名アルゴリズムを利用します
* 拡張機能とドメイン分離をサポートするためのフレームワーク
* 選択暗号文攻撃に対して無防備です
* ユーザーが意図していなかった潜在的に署名するトランザクションに対する保護があります

この仕様は、理論的根拠と標準化されたものにのみ関係しています。
Cosmos署名付きメッセージの実装。 それは**関係ありません**
リプレイ攻撃の概念は、上位レベルのアプリケーションに委ねられます
実装。 一部を承認する手段で署名されたメッセージを表示する場合
アクションまたはデータの場合、そのようなアプリケーションはこれを次のように扱う必要があります
べき等であるか、既知の署名付きメッセージを拒否するメカニズムがあります。

## 予備

Cosmosメッセージ署名プロトコルは暗号化されてパラメータ化されます
セキュアハッシュアルゴリズム `SHA-256`と署名アルゴリズム` S`を含む
セットにデジタル署名を提供する操作 `sign`と` verify`
それぞれバイト数と署名の検証。

ここでの私たちの目標は、必ずしも理由についてのコンテキストと推論を提供することではないことに注意してください
これらのアルゴリズムは、事実上のアルゴリズムであるという事実とは別に選択されました
TendermintとCosmosSDKで使用されており、そのようなニーズを満たしていること
衝突耐性や秒耐性などの暗号化アルゴリズム
原像攻撃、および[決定論的](https://en.wikipedia.org/wiki/Hash_function#Determinism)および[均一](https://en.wikipedia.org/wiki/Hash_function#Uniformity) 

## 仕様

Tendermintには、正規のメッセージを使用してメッセージに署名するための確立されたプロトコルがあります
定義されているJSON表現[ここ](https://github.com/tendermint/tendermint/blob/master/types/canonical.go)。

このような正規のJSON構造の例は、Tendermintの投票構造です。

```go
type CanonicalJSONVote struct {
    ChainID   string               `json:"@chain_id"`
    Type      string               `json:"@type"`
    BlockID   CanonicalJSONBlockID `json:"block_id"`
    Height    int64                `json:"height"`
    Round     int                  `json:"round"`
    Timestamp string               `json:"timestamp"`
    VoteType  byte                 `json:"type"`
}
```

このような正規のJSON構造では、仕様に次のものが含まれている必要があります
メタフィールド: `@ chain_id`と` @ type`。これらのメタフィールドは予約されており、
含まれています。どちらも `string`型です。さらに、フィールドは順序付けする必要があります
辞書式順序で昇順。

Cosmosメッセージに署名するには、 `@ chain_id`フィールドが対応している必要があります
コスモスチェーン識別子に。ユーザーエージェントは、次の場合に署名を**拒否**する必要があります
`@ chain_id`フィールドが現在アクティブなチェーンと一致しません！ `@ type`フィールド
定数 `" message "`と等しくなければなりません。 `@ type`フィールドは次のタイプに対応します
ユーザーがアプリケーションにサインインする構造。今のところ、ユーザーは
有効なASCIIテキストのバイトに署名できます([ここを参照](https://github.com/tendermint/tendermint/blob/master/libs/common/string.go#L61-L74))。
ただし、これは変更され、追加のアプリケーション固有をサポートするように進化します
人間が読み取れ、機械が検証できる構造([Future Adaptations](#futureadaptations)を参照)。

したがって、を使用してCosmosメッセージに署名するための正規のJSON構造を持つことができます
[JSONスキーマ](http://json-schema.org/)仕様自体:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "$id": "cosmos/signing/typeData/schema",
  "title": "The Cosmos signed message typed data schema.",
  "type": "object",
  "properties": {
    "@chain_id": {
      "type": "string",
      "description": "The corresponding Cosmos chain identifier.",
      "minLength": 1
    },
    "@type": {
      "type": "string",
      "description": "The message type. It must be 'message'.",
      "enum": [
        "message"
      ]
    },
    "text": {
      "type": "string",
      "description": "The valid ASCII text to sign.",
      "pattern": "^[\\x20-\\x7E]+$",
      "minLength": 1
    }
  },
  "required": [
    "@chain_id",
    "@type",
    "text"
  ]
}
```

e.g.

```json
{
  "@chain_id": "1",
  "@type": "message",
  "text": "Hello, you can identify me as XYZ on keybase."
}
```

## 将来の適応

アプリケーションはドメインによって大きく異なる可能性があるため、両方をサポートすることが不可欠です。
ドメインの分離と、人間が読み取れる機械で検証可能な構造。

ドメインの分離により、アプリケーション開発者はの衝突を防ぐことができます
それ以外は同一の構造。 アプリケーションごとに一意になるように設計する必要があります
を使用し、署名エンコーディング自体で直接使用する必要があります。

人間が読み取れ、機械で検証できる構造により、エンドユーザーは署名できます
文字列メッセージだけでなく、より複雑な構造でありながら、
彼らが何に署名しているのかを正確に知っている(任意のバイトの束に署名するのではなく)。

したがって、将来的には、コスモス署名メッセージの仕様が期待されます
そのような機能を含めるために、正規のJSON構造を拡張します。

## API

アプリケーション開発者と設計者は、次のようなAPIの標準セットを形式化する必要があります。
次の仕様を順守してください。

-----

### ** cosmosSignBytes **

パラメータ:

* `data`:Cosmos署名付きメッセージの正規JSON構造
* `address`:データに署名するためのBech32Cosmosアカウントアドレス

戻り値:

* `signature`:署名アルゴリズム` S`を使用して導出されたCosmos署名

-----

### 例

`secp256k1`をDSAとして使用すると、` S`:

```javascript
data = {
  "@chain_id": "1",
  "@type": "message",
  "text": "I hereby claim I am ABC on Keybase!"
}

cosmosSignBytes(data, "cosmos1pvsch6cddahhrn5e8ekw0us50dpnugwnlfngt3")
> "0x7fc4a495473045022100dec81a9820df0102381cdbf7e8b0f1e2cb64c58e0ecda1324543742e0388e41a02200df37905a6505c1b56a404e23b7473d2c0bc5bcda96771d2dda59df6ed2b98f8"
```

## 参照
