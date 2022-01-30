# ADR 027:確定的Protobufシリアル化

## 変更ログ

-2020-08-07:最初のドラフト
-2020-09-01:ルールをさらに明確にする

## 状態

提案

## 概要

完全に決定論的な構造のシリアル化、複数の言語とクライアントに適しています、
メッセージに署名するときに必要です。シリアル化するときはいつでも確認する必要があります
サポートされている言語に関係なく、生のバイトのデータ構造
変更されません。
[Protobuf](https://developers.google.com/protocol-buffers/docs/proto3)
シリアル化は全単射ではありません(つまり、実際には無限の数があります
指定されたprotobufドキュメントの有効なバイナリ表現)<sup> 1 <.sup>。

このドキュメントでは、決定論的なシリアル化スキームについて説明します
このユースケースをカバーするprotobufドキュメントのサブセットですが、
他の状況でも同じことが言えます。

### 環境

Cosmos SDKでの署名検証では、署名者と検証者が合意に達する必要があります
`SignDoc`で定義されているのと同じシリアル化
[ADR-020](..adr-020-protobuf-transaction-encoding.md)は送信しません
シリアル化。

現在、ブロック署名には回避策を使用しています。新しい[TxRaw]を作成します(https://github.com/cosmos/cosmos-sdk/blob/9e85e81e0e8140067dd893421290c191529c148c/proto/cosmos/tx/v1proto#L30x)
例([adr-020-protobuf-transaction-encoding](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-020-protobuf-transaction-encoding.md#で定義)トレード))
すべての[Tx]を変換する(https://github.com/cosmos/cosmos-sdk/blob/9e85e81e0e8140067dd893421290c191529c148c/proto/cosmos/tx/v1beta1/tx.proto#L13)
フィールドからクライアントまでのバイト数。これにより、追加のマニュアルが追加されます
トランザクションを送信および署名する際の手順。

### 決定

次のコーディングスキームは、他のADRで使用されます。
特に `SignDoc`シリアル化の場合。

## 仕様

### スコープ

このADRは、protobuf3シリアライザーを定義します。出力は有効なprotobufです
すべてのprotobufパーサーが解析できるようにシリアル化します。

定義が複雑なため、バージョン1ではマッピングはサポートされていません。
決定論的シリアル化。これは将来変更される可能性があります。実装する必要があります
マップを含むドキュメントは、無効な入力として拒否されます。

### 背景-Protobuf3エンコーディング

protobuf3のほとんどの数値タイプは、次のようにエンコードされます。
[varints](https://developers.google.com/protocol-buffers/docs/encoding#varints)。
各varintバイトには7ビットのデータがあるため、varintは最大10バイトになる可能性があります。
varintsは、 `uint70`(70ビットの符号なし整数)の表現です。いつ
エンコードすると、値は基本タイプから `uint70`に変換され、
デコード時に、解析されたuint70は適切な数値型に変換されます。

protobuf3に準拠するvarintの最大有効値
`FF FF FF FF FF FF FF FF FF 7F`(つまり` 2 ** 70 -1`)。フィールドタイプが
`{、u、s} int64`、70ビットの上位6ビットは、デコードプロセス中に破棄されます。
6ビットの延性を導入します。フィールドタイプが `{、u、s} int32`の場合、
70の最上位38ビットはデコードプロセス中に破棄され、38ビットが導入されます
可鍛性。

他の不確実性の原因の中でも、このADRは次の可能性を排除します 
可鍛性のコーディング。

### シリアル化ルール

シリアル化はに基づいています
[protobuf3エンコーディング](https://developers.google.com/protocol-buffers/docs/encoding)
次のコンテンツが追加されました。

1.フィールドは昇順で1回だけシリアル化できます
2.追加のフィールドまたは追加のデータを追加することはできません
3. [デフォルト値](https://developers.google.com/protocol-buffers/docs/proto3#default)
   省略しなければならない
4.スカラー数値型の `repeated`フィールドを使用する必要があります
   [パッキングエンコーディング](https://developers.google.com/protocol-buffers/docs/encoding#packed)
5.Varintエンコーディングは必要性を超えることはできません。
    *末尾のゼロバイトはありません(つまり、小さい方の端には、大きい方の先頭のゼロはありません)
      エンディアン)。上記のルール3によると、デフォルト値の[0]は省略しなければならないため、
      この規則は、そのような状況には適用されません。
    ※varintの最大値は `FF FF FF FF FF FF FF FFFF01`である必要があります。
      言い換えると、デコードするとき、70ビットの符号なしの上位6ビット
      整数は[0]でなければなりません。 (10バイトのvarintは、7ビットの10グループです。
      70ビット。そのうち最下位の70-6 = 64のみが有用です。 )。
    * varintエンコーディングの32ビット値の最大値は、 `FF FF FF FF0F`である必要があります
      1つの例外を除いて(以下)。言い換えれば、デコードするとき、最高の38
      70ビットの符号なし整数のビットは[0]である必要があります。
        *上記の1つの例外は、_negative_`int32`です。
          完全な10バイトエンコーディングを使用して、拡張機能<sup> 2 <.sup>に署名します。
    * varintエンコーディングのブール値の最大値は[01]である必要があります(つまり、
      [0]または[1]である必要があります)。上記のルール3によると、デフォルト値の[0]は
      省略されているため、ブール値が含まれている場合、その値は[1]である必要があります。

ルール1と2は非常に単純で、説明する必要がありますが
作成者に知られているすべてのprotobufエンコーダーのデフォルトの動作、3番目のルール
もっと楽しく。 protobuf3の逆シリアル化後は、区別できません
未設定フィールドとデフォルト値として設定されたフィールドの間の<sup> 3 <.sup>。存在
ただし、シリアル化レベルでは、フィールドを空に設定できます
それらに注意を払うか、それらを完全に省略してください。これは、JSONなどの重要な違いです
属性は空( `" "`、 `0`)、` null`、または未定義にすることができ、結果として3になります。
別のファイル。

パーサーが割り当てる必要があるため、デフォルト値として設定されたフィールドを省略することは有効です。
シリアル化で欠落しているフィールドのデフォルト値は<sup> 4 <.sup>です。スカラーの場合
タイプ、仕様では、デフォルト値<sup> 5 <.sup>を省略する必要があります。 [リピート]の場合
空のリストを表現する唯一の方法は、フィールドをシリアル化する代わりに使用することです。列挙型は
値が0の最初の要素があります。これはデフォルトの<sup> 6 <.sup>です。と
メッセージフィールドのデフォルトはunset <sup> 7 <.sup>です。

デフォルト値を省略すると、ある程度の上位互換性が得られます。
新しいバージョンのprotobufモードでは、ユーザーと同じシリアル化が生成されます
新しく追加されたフィールドが使用されていない限り(つまり、
デフォルト)。

### 埋め込む

低から高にソートされた3つの主要な実装戦略があります
ほとんどのカスタム開発:

-**上記のルールに従うprotobufシリアライザーがデフォルトで使用されます。 ** 例えば
  [gogoproto](https://pkg.go.dev/github.com/gogo/protobuf/gogoproto)既知
  ほとんどの場合に会いますが、いくつかのメモ(例:
  `nullable = false`を使用します。それはまたオプションかもしれません
  これに対応して、既存のシリアル化プログラム。
-**エンコードする前にデフォルト値を標準化します。 **シリアル化プログラムが次の場合
  ルール1.および2.で、フィールドのシリアル化を明示的に解除できます。
  デフォルト値を正規化して設定しないようにすることができます。これは、使用中に行うことができます
  [protobuf.js](https://www.npmjs.com/package/protobufjs):  

  ```js
  const bytes = SignDoc.encode({
    bodyBytes: body.length > 0 ? body : null,..normalize empty bytes to unset
    authInfoBytes: authInfo.length > 0 ? authInfo : null,..normalize empty bytes to unset
    chainId: chainId || null,..normalize "" to unset
    accountNumber: accountNumber || null,..normalize 0 to unset
    accountSequence: accountSequence || null,..normalize 0 to unset
  }).finish();
  ```

-**必要なタイプの手書きのシリアル化プログラムを使用します。 **上記のいずれでもない場合
    この方法は便利です。独自のシリアル化プログラムを作成できます。 SignDocの場合これ
    Goでは、既存のprotobufユーティリティに基づいて次のように表示されます。 

  ```go
  if !signDoc.body_bytes.empty() {
      buf.WriteUVarInt64(0xA)..wire type and field number for body_bytes
      buf.WriteUVarInt64(signDoc.body_bytes.length())
      buf.WriteBytes(signDoc.body_bytes)
  }

  if !signDoc.auth_info.empty() {
      buf.WriteUVarInt64(0x12)..wire type and field number for auth_info
      buf.WriteUVarInt64(signDoc.auth_info.length())
      buf.WriteBytes(signDoc.auth_info)
  }

  if !signDoc.chain_id.empty() {
      buf.WriteUVarInt64(0x1a)..wire type and field number for chain_id
      buf.WriteUVarInt64(signDoc.chain_id.length())
      buf.WriteBytes(signDoc.chain_id)
  }

  if signDoc.account_number != 0 {
      buf.WriteUVarInt64(0x20)..wire type and field number for account_number
      buf.WriteUVarInt(signDoc.account_number)
  }

  if signDoc.account_sequence != 0 {
      buf.WriteUVarInt64(0x28)..wire type and field number for account_sequence
      buf.WriteUVarInt(signDoc.account_sequence)
  }
  ```

### Test vectors

protobufが `Article.proto`を定義しているとすると 

```protobuf
package blog;
syntax = "proto3";

enum Type {
  UNSPECIFIED = 0;
  IMAGES = 1;
  NEWS = 2;
};

enum Review {
  UNSPECIFIED = 0;
  ACCEPTED = 1;
  REJECTED = 2;
};

message Article {
  string title = 1;
  string description = 2;
  uint64 created = 3;
  uint64 updated = 4;
  bool public = 5;
  bool promoted = 6;
  Type type = 7;
  Review review = 8;
  repeated string comments = 9;
  repeated string backlinks = 10;
};
```

serializing the values

```yaml
title: "The world needs change 🌳"
description: ""
created: 1596806111080
updated: 0
public: true
promoted: false
type: Type.NEWS
review: Review.UNSPECIFIED
comments: ["Nice one", "Thank you"]
backlinks: []
```

must result in the serialization

```
0a1b54686520776f726c64206e65656473206368616e676520f09f8cb318e8bebec8bc2e280138024a084e696365206f6e654a095468616e6b20796f75
```

When inspecting the serialized document, you see that every second field is
omitted:

```
$ echo 0a1b54686520776f726c64206e65656473206368616e676520f09f8cb318e8bebec8bc2e280138024a084e696365206f6e654a095468616e6b20796f75 | xxd -r -p | protoc --decode_raw
1: "The world needs change \360\237\214\263"
3: 1596806111080
5: 1
7: 2
9: "Nice one"
9: "Thank you"
```

## 結果

このようなエンコーディングを使用すると、決定論的なシリアル化を取得できます
CosmosSDK署名コンテキストで必要なすべてのprotobufドキュメント。

### ポジティブ

-参照とは無関係に検証できる明確に定義されたルール
   埋め込む
-トランザクション署名を実装するための障壁を下げるのに十分シンプル
-SignDocで0およびその他のnull値を引き続き使用して回避することができます
   0シーケンスを解く必要があります。これはからという意味ではありません
   https://github.com/cosmos/cosmos-sdk/pull/6949はマージしないでくださいが、マージしないでください
   重要すぎる。

### ネガティブ

-トランザクション署名を実装する場合は、上記のコーディングルールに準拠する必要があります
   理解して実行します。
-ルール番号3が必要なため、実装が複雑になります。
-特定のデータ構造では、シリアル化のためにカスタムコードが必要になる場合があります。したがって
   コードはあまり移植性がありません-それぞれに追加の作業が必要です
   クライアントは、カスタムデータ構造を正しく処理するためにシリアル化を実装します。

### ニュートラル

### CosmosSDKでの使用法

上記の理由([ネガティブ]な部分)のため、回避策を維持することをお勧めします
データ構造を共有するために使用されます。例:前述の `TxRaw`は生のバイトを使用します
解決策として。これにより、有効なProtobufライブラリを使用せずに使用できます。
この標準(および関連するエラーリスク)を満たすカスタムシリアル化プログラムを実装する必要があります。

## 参照

-<sup> 1 <.sup> _メッセージがシリアル化される場合、順序は保証されません
  既知または未知のフィールドをどのように書き込む必要がありますか。シリアル化の順序は
  実装の詳細および特定の実装の詳細は、
  将来の変更。したがって、プロトコルバッファパーサーは解析できる必要があります
  任意の順序のフィールド。 _から
  https://developers.google.com/protocol-buffers/docs/encoding#order
-<sup> 2 <.sup> https://developers.google.com/protocol-buffers/docs/encoding#signed_integers
-<sup> 3 <.sup> _スカラーメッセージフィールドの場合、メッセージが解析されると
  フィールドが明示的にデフォルト値に設定されているかどうかを判断する方法はありません
  値(ブール値がfalseに設定されているかどうかなど)またはまったく設定されていない場合:
  メッセージタイプを定義するときは、このことに注意する必要があります。例えば、
  falseに設定すると、ブール値のいずれも特定の動作を有効にしません。
  この動作がデフォルトで発生することは望ましくありません。 _から
  https://developers.google.com/protocol-buffers/docs/proto3#default
-<sup> 4 <.sup> _メッセージが解析されるとき、エンコードされたメッセージが解析されない場合
  特定の特異要素、分析の対応するフィールドが含まれています
  オブジェクトはフィールドのデフォルト値に設定されます。 _から
  https://developers.google.com/protocol-buffers/docs/proto3#default
-<sup> 5 <.sup> _スカラーメッセージフィールドがデフォルト値に設定されている場合は、
  値はオンラインでシリアル化されません。 _から
  https://developers.google.com/protocol-buffers/docs/proto3#default
-<sup> 6 <.sup> _列挙の場合、デフォルト値は最初に定義された列挙値です。
  0である必要があります。
  https://developers.google.com/protocol-buffers/docs/proto3#default
-<sup> 7 <.sup> _メッセージフィールドの場合、このフィールドは設定されていません。その正確な値は
  言語関連。 _から
  https://developers.google.com/protocol-buffers/docs/proto3#default
-コーディングルールと推論の一部は、
  [canonical-proto3 Aaron Craelius](https://github.com/regen-network/canonical-proto3) 