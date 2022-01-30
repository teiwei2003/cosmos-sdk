# ADR 020:プロトコルバッファトランザクションコーディング

## 変更ログ

-2020年3月6日:最初のドラフト
-2020年3月12日:APIアップデート
-2020年4月13日:[oneof]インターフェースの処理に関する詳細情報を追加
-2020年4月30日:[任意]に切り替えます
-2020年5月14日:公開鍵のエンコーディングについて説明する
-2020年6月8日:[TxBody]と[AuthInfo]をバイトとして[SignDoc]に保存します。ドキュメント[TxRaw]をブロードキャストおよびストレージタイプとして保存します。
-2020年8月7日:ADR027を使用して `SignDoc`をシリアル化します。
-2020年8月19日:[#6966](https://github.com/cosmos/cosmos-sdk/issues/6966)の説明に従って、シーケンスフィールドを `SignDoc`から` SignerInfo`に移動します。
-2020年9月25日:[PublicKey]タイプを削除し、[secp256k1.PubKey]、[ed25519.PubKey]、[multisig.LegacyAminoPubKey]に置き換えます。
-2020年10月15日:[AccountRetriever]インターフェースに[GetAccount]メソッドと[GetAccountWithHeight]メソッドを追加しました。
-2021年2月24日:Cosmos SDKはTendermintの `PubKey`インターフェースを使用しなくなりましたが、独自の` cryptotypes.PubKey`を使用しています。これを反映するように更新します。
-2021年5月3日: `clientCtx.JSONMarshaler`の名前を` clientCtx.JSONCodec`に変更しました。
-2021年6月10日: `clientCtx.Codec:codec.Codec`を追加します。

## 状態

受け入れられました

## 環境

ADRは
[ADR 019](..adr-019-protobuf-state-encoding.md)、つまり、私たちの目標は設計することです
CosmosSDKクライアントのプロトコルバッファ移行パス。

具体的には、クライアント移行パスには主にtx生成と
署名、メッセージの作成とルーティング、およびCLIとRESTハンドラーと
ビジネスロジック(つまり、クエリア)。

これを念頭に置いて、2つの主要な領域であるtxsと
お問い合わせください。ただし、このADRはトランザクションのみに焦点を当てています。クエリは
これは将来のADRで解決される予定ですが、これらの推奨事項に基づいている必要があります。

詳細なディスカッションに基づく([\#6030](https://github.com/cosmos/cosmos-sdk/issues/6030)
そして[\#6078](https://github.com/cosmos/cosmos-sdk/issues/6078))、オリジナル
トランザクションの設計は、 `oneof` .JSON署名から大幅に変更されました
以下に説明する方法の方法。

## 決定

### トレード

インターフェイス値は、状態で `google.protobuf.Any`を使用してエンコードされるため([ADR 019](adr-019-protobuf-state-encoding.md)を参照)、
`sdk.Msg`sは、トランザクションでのエンコードに` Any`を使用します。

`Any`を使用してインターフェース値をエンコードする主な目標の1つは、
アプリケーションが再利用するタイプのコアセット
クライアントは、可能な限り多くのチェーンと安全に互換性があります。

柔軟なクロスチェーントランザクションを提供することは、この仕様の目標の1つです。
クライアントを壊すことなく、さまざまなユースケースにサービスを提供できるフォーマット
互換性。

署名を容易にするために、トランザクションは `TxBody`に分割されます。
次の `SignDoc`と` signatures`はそれを再利用します: 


```proto
/.types/types.proto
package cosmos_sdk.v1;

message Tx {
    TxBody body = 1;
    AuthInfo auth_info = 2;
   ..A list of signatures that matches the length and order of AuthInfo's signer_infos to
   ..allow connecting signature meta information like public key and signing mode by position.
    repeated bytes signatures = 3;
}

/.A variant of Tx that pins the signer's exact binary represenation of body and
/.auth_info. This is used for signing, broadcasting and verification. The binary
/.`serialize(tx: TxRaw)` is stored in Tendermint and the hash `sha256(serialize(tx: TxRaw))`
/.becomes the "txhash", commonly used as the transaction ID.
message TxRaw {
   ..A protobuf serialization of a TxBody that matches the representation in SignDoc.
    bytes body = 1;
   ..A protobuf serialization of an AuthInfo that matches the representation in SignDoc.
    bytes auth_info = 2;
   ..A list of signatures that matches the length and order of AuthInfo's signer_infos to
   ..allow connecting signature meta information like public key and signing mode by position.
    repeated bytes signatures = 3;
}

message TxBody {
   ..A list of messages to be executed. The required signers of those messages define
   ..the number and order of elements in AuthInfo's signer_infos and Tx's signatures.
   ..Each required signer address is added to the list only the first time it occurs.
   ./
   ..By convention, the first required signer (usually from the first message) is referred
   ..to as the primary signer and pays the fee for the whole transaction.
    repeated google.protobuf.Any messages = 1;
    string memo = 2;
    int64 timeout_height = 3;
    repeated google.protobuf.Any extension_options = 1023;
}

message AuthInfo {
   ..This list defines the signing modes for the required signers. The number
   ..and order of elements must match the required signers from TxBody's messages.
   ..The first element is the primary signer and the one which pays the fee.
    repeated SignerInfo signer_infos = 1;
   ..The fee can be calculated based on the cost of evaluating the body and doing signature verification of the signers. This can be estimated via simulation.
    Fee fee = 2;
}

message SignerInfo {
   ..The public key is optional for accounts that already exist in state. If unset, the
   ..verifier can use the required signer address for this position and lookup the public key.
    google.protobuf.Any public_key = 1;
   ..ModeInfo describes the signing mode of the signer and is a nested
   ..structure to support nested multisig pubkey's
    ModeInfo mode_info = 2;
   ..sequence is the sequence of the account, which describes the
   ..number of committed transactions signed by a given address. It is used to prevent
   ..replay attacks.
    uint64 sequence = 3;
}

message ModeInfo {
    oneof sum {
        Single single = 1;
        Multi multi = 2;
    }

   ..Single is the mode info for a single signer. It is structured as a message
   ..to allow for additional fields such as locale for SIGN_MODE_TEXTUAL in the future
    message Single {
        SignMode mode = 1;
    }

   ..Multi is the mode info for a multisig public key
    message Multi {
       ..bitarray specifies which keys within the multisig are signing
        CompactBitArray bitarray = 1;
       ..mode_infos is the corresponding modes of the signers of the multisig
       ..which could include nested multisig public keys
        repeated ModeInfo mode_infos = 2;
    }
}

enum SignMode {
    SIGN_MODE_UNSPECIFIED = 0;

    SIGN_MODE_DIRECT = 1;

    SIGN_MODE_TEXTUAL = 2;

    SIGN_MODE_LEGACY_AMINO_JSON = 127;
}
```

以下で説明するように、できるだけ多くの[Tx]を含めるために
`SignDoc`では、` SignerInfo`は署名から分離されているため、
元の署名自体は、署名されたコンテンツの外部に存在します。

私たちの目標は柔軟でスケーラブルなクロスチェーントランザクションであるため
フォーマット、新しいトランザクション処理オプションをできるだけ早く `TxBody`に追加する必要があります
これらのユースケースは、まだ実現されていなくても発見されています。

ここには調整のオーバーヘッドがあるため、 `TxBody`には
`extension_options`フィールドは、任意のトランザクションに使用できます
オプションはまだカバーされていません。それでも、アプリケーション開発者は
[Tx]の重要な上流の改善を試してください。

### サイン

次のすべての署名モードは、次の保証を提供するように設計されています。

-**可鍛性なし**:一度取引すると、 `TxBody`と` AuthInfo`は変更できません
  署名
-**予測可能なガス**:料金を支払う取引に署名する場合、
  最終的なガスは私が署名したものに完全に依存します

これらの保証は、メッセージ署名者に最大の自信を提供します
仲介者による[Tx]の操作は、意味のある変更を引き起こしません。

#### `SIGN_MODE_DIRECT`

[直接]署名動作は、元の[TxBody]バイトをブロードキャストとして署名することです。
電線。これには次の利点があります。

-標準プロトコルを超える最小限の追加クライアント機能が必要です
  バッファの実装
-トランザクションの順応性のために効果的なゼロの抜け穴を残します(つまり、
  署名とエンコード形式の微妙な違いは
  攻撃者によって悪用される可能性があります)

署名は次の `SignDoc`で構成され、再利用されます
`TxBody`と` AuthInfo`は、署名に必要なフィールドのみを追加します。 

```proto
/.types/types.proto
message SignDoc {
   ..A protobuf serialization of a TxBody that matches the representation in TxRaw.
    bytes body = 1;
   ..A protobuf serialization of an AuthInfo that matches the representation in TxRaw.
    bytes auth_info = 2;
    string chain_id = 3;
    uint64 account_number = 4;
}
```

デフォルトモードでログインするために、クライアントは次の手順を実行します。

1.有効なprotobufを使用して、 `TxBody`と` AuthInfo`をシリアル化します。
2. `SignDoc`を作成し、[ADR 027](..adr-027-deterministic-protobuf-serialization.md)を使用してシリアル化します。
3.エンコードされた `SignDoc`バイトに署名します。
4. `TxRaw`を作成し、ブロードキャスト用にシリアル化します。

署名の検証は、より原始的な[TxBody]と[AuthInfo]に基づいています
`TxRaw`でエンコードされたバイトは、["正規化 "](https://github.com/regen-network/canonical-proto3)に基づいていません。
防止することに加えて
アップグレード可能性のいくつかの形式(このドキュメントの後半で説明します)。

署名ベリファイアは次のことを行います。

1. `TxRaw`を逆シリアル化し、` body`と `auth_info`を引き出します。
2.メッセージに基づいて必要な署名者アドレスのリストを作成します。
3.必要な署名者ごとに:
   -ステータスからアカウント番号とシーケンスを抽出します。
   -状態または `AuthInfo`の` signer_infos`から公開鍵を取得します。
   -`SignDoc`を作成し、[ADR 027](..adr-027-deterministic-protobuf-serialization.md)を使用してシリアル化します。
   -シリアル化された[SignDoc]に対して同じリスト位置の署名を確認します。

#### `SIGN_MODE_LEGACY_AMINO`

古いウォレットと交換をサポートするために、AminoJSONは一時的に
トランザクション署名をサポートします。ウォレットとエクスチェンジが
protobufベースの署名にアップグレードする機会がある場合、このオプションは無効になります。存在
同時に、現在のAmino署名を無効にすると、次のようになることが予測されます。
あまりにも多くの破損は実行可能ではありません。これは主に要件であることに注意してください
Cosmos Hubおよびその他のチェーンは、Amino署名をすぐに無効にすることを選択できます。

古い顧客は、現在のAminoを使用してトランザクションに署名できるようになります
JSON形式で、REST `.tx .encode`を使用してprotobufとしてエンコードします
ブロードキャスト前のエンドポイント。

#### `SIGN_MODE_TEXTUAL`

[\#6078](https://github.com/cosmos/cosmos-sdk/issues/6078)で詳しく説明されているように、
人間が読める署名エンコーディング、特にハードウェアが必要
[元帳](https://www.ledger.com)のように表示されるウォレット
トランザクションの内容は、署名する前にユーザーに通知されます。 JSONは試みですが、
理想に達していません。

`SIGN_MODE_TEXTUAL`は、人間が読める形式のプレースホルダーとして意図されています
アミノJSONのエンコーディングが置き換えられます。この新しいエンコーディングはもっと必要です
読みやすさはJSONよりも重要であり、次のようなフォーマットされた文字列に基づいている場合があります。
[メッセージ形式](http://userguide.icu-project.org/formatparse/messages)。

新しい人間が読める形式が影響を受けないようにするため
トランザクションの可鍛性の問題、 `SIGN_MODE_TEXTUAL`
_人間が読めるバイトと元の `SignDoc`_を連結する必要があります
シンボルバイトを生成します。

複数の人間が読める形式をサポートする可能性があります(おそらくローカライズされたメッセージでさえ)
実装中に `SIGN_MODE_TEXTUAL`を渡します。

### 不明なフィールドフィルタリング

protobufメッセージの不明なフィールドは、通常、トランザクションによって拒否されます。
プロセッサの理由:

-重要なデータが不明なフィールドに存在する可能性があります。省略した場合は、
  顧客の予期しない行動を引き起こす
-彼らは、攻撃者がtxサイズを膨らませることができる可鍛性の抜け穴を提案しました
  説明されていないランダムなデータを署名されていないコンテンツ(つまり、メインの[Tx])に追加することにより、
  `TxBody`ではありません)

未知のフィールドを安全に無視することを選択できるシナリオがまだいくつかあります
(https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-624400188)から
新しいクライアントとのエレガントな上位互換性を提供します。

11番目のフィールド番号を設定することをお勧めします(ほとんどのユースケースでは、これは
1024〜2047の範囲)は、安全で重要ではない領域と見なされます
不明な場合は無視してください。

この問題を解決するには、未知のフィールドフィルターが必要です。

-署名されていないコンテンツの不明なフィールドを常に拒否します(つまり、トップレベルの `Tx`と
  `AuthInfo`の符号なし部分(署名モードに基づいて存在する場合)
-すべてのメッセージ(ネストされた `Any`を含む)の不明なフィールドを拒否します。
  11番目のビットが設定されたフィールド 

これには、メッセージバイトを受け入れるカスタムprotobufパーサーを渡す必要がある場合があります
そして `FileDescriptor`sとブール結果を返します。

### 公開鍵エンコーディング

Cosmos SDKの公開鍵は、 `cryptotypes.PubKey`インターフェースを実装します。
他のインターフェース(たとえば、 `BaseAccount.PubKey`や` SignerInfo.PublicKey`)を使用するのと同じように、protobufエンコーディングには `Any`を使用することをお勧めします。
次の公開鍵が実装されています:secp256k1、secp256r1、ed25519およびlegacy-multisignature。 
Ex: 

```proto
message PubKey {
    bytes key = 1;
}
```

`multisig.LegacyAminoPubKey`には、任意のをサポートするための` Any`のメンバー配列があります
protobuf公開鍵タイプ。

アプリケーションは、登録した公開鍵のセットのみを処理しようとする必要があります
テスト済みです。 提供された署名検証アンティハンドラデコレータは
この操作を実行します。

### CLIとREST

現在、RESTおよびCLIハンドラーは、Aminoを介してタイプとTXをエンコードおよびデコードします
特定のAminoコーデックのJSONエンコーディングを使用します。 特定の種類の処理のため
[ADR 019](..adr-019-protobuf-state-encoding.md)で説明したのと同様に、クライアントのインターフェイスにすることができます。
クライアントロジックは、次の方法を知っているだけでなく、コーデックインターフェイスを採用する必要があります。
すべてのタイプを処理しますが、トランザクション、署名、
そしてニュース。  

```go
type AccountRetriever interface {
  GetAccount(clientCtx Context, addr sdk.AccAddress) (client.Account, error)
  GetAccountWithHeight(clientCtx Context, addr sdk.AccAddress) (client.Account, int64, error)
  EnsureExists(clientCtx client.Context, addr sdk.AccAddress) error
  GetAccountNumberSequence(clientCtx client.Context, addr sdk.AccAddress) (uint64, uint64, error)
}

type Generator interface {
  NewTx() TxBuilder
  NewFee() ClientFee
  NewSignature() ClientSignature
  MarshalTx(tx types.Tx) ([]byte, error)
}

type TxBuilder interface {
  GetTx() sdk.Tx

  SetMsgs(...sdk.Msg) error
  GetSignatures() []sdk.Signature
  SetSignatures(...sdk.Signature)
  GetFee() sdk.Fee
  SetFee(sdk.Fee)
  GetMemo() string
  SetMemo(string)
}
```

次に、 `Context`を更新して、新しいフィールドを取得します:` Codec`、 `TxGenerator`、
そして `AccountRetriever`、` AppModuleBasic.GetTxCmd`を更新します
[コンテキスト]。これらすべてのフィールドに事前入力する必要があります。

次に、各クライアントメソッドは[Init]メソッドの1つを使用して再初期化する必要があります
事前入力された[コンテキスト]。 `tx.GenerateOrBroadcastTx`を使用できます
トランザクションを生成またはブロードキャストします。 例えば:

```go
import "github.com/spf13/cobra"
import "github.com/cosmos/cosmos-sdk/client"
import "github.com/cosmos/cosmos-sdk/client/tx"

func NewCmdDoSomething(clientCtx client.Context) *cobra.Command {
	return &cobra.Command{
		RunE: func(cmd *cobra.Command, args []string) error {
			clientCtx := ctx.InitWithInput(cmd.InOrStdin())
			msg := NewSomeMsg{...}
			tx.GenerateOrBroadcastTx(clientCtx, msg)
		},
	}
}
```

## 将来の改善

### `SIGN_MODE_TEXTUAL`仕様

[SIGN_MODE_TEXTUAL]の特定の仕様と実装は、
元帳アプリケーションやその他のウォレットを可能にするための短期的な将来の改善として
AminoJSONから正常に移行できます。

### `SIGN_MODE_DIRECT_AUX`

(\ * https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-628026933にオプション(3)として記録されています)

モード `SIGN_MODE_DIRECT_AUX`を追加できます
複数の署名シナリオをサポートする
単一のトランザクションに収集されていますが、メッセージエディタは収集されません
最終的なトランザクションにどの署名が含まれるかもわかっています。例えば、
3/5のマルチシグニチャウォレットを持っていて、5人全員に[TxBody]を送信したい場合があります
署名者は、誰が最初に署名するかを確認します。署名が3つあるとすぐに行きます
完全なトランザクションを進めて確立します。

`SIGN_MODE_DIRECT`を使用するには、各署名者は
すべての署名者の完全なリストを含む完全な[AuthInfo]に署名し、
それらの署名モードは、上記のシナリオを非常に困難にします。

`SIGN_MODE_DIRECT_AUX`は、[補助]署名者が署名を作成できるようにします
`TxBody`と独自の` PublicKey`のみを使用してください。これにより、完全なリストが可能になります
AuthInfoの署名者は、署名が収集されるまで遅延されます。

[セカンダリ]署名者は、メイン署名者以外の署名者です。
料金。メインの署名者の場合、ガスと料金を計算するには、実際には完全なAuthInfoが必要です
署名者の数と、キーの種類と署名によって異なります。
彼らが使用しているモード。ただし、二次署名者は心配する必要はありません
コストまたはガソリンなので、[TxBody]に署名するだけです。

[SIGN_MODE_DIRECT_AUX]で署名を生成するには、次の手順に従います。

1. `SignDocAux`のエンコード(フィールドをシリアル化する必要があるという要件と同じ)
   にとって):  

```proto
/.types/types.proto
message SignDocAux {
    bytes body_bytes = 1;
   ..PublicKey is included in SignDocAux :
   ..1. as a special case for multisig public keys. For multisig public keys,
   ..the signer should use the top-level multisig public key they are signing
   ..against, not their own public key. This is to prevent against a form
   ..of malleability where a signature could be taken out of context of the
   ..multisig key that was intended to be signed for
   ..2. to guard against scenario where configuration information is encoded
   ..in public keys (it has been proposed) such that two keys can generate
   ..the same signature but have different security properties
   ./
   ..By including it here, the composer of AuthInfo cannot reference the
   ..a public key variant the signer did not intend to use
    PublicKey public_key = 2;
    string chain_id = 3;
    uint64 account_number = 4;
}
```

2.エンコードされた `SignDocAux`バイトに署名します
3.署名と `SignerInfo`をメインの署名者に送信すると、
    最終トランザクションに署名してブロードキャストします([SIGN_MODE_DIRECT]と[AuthInfo]を使用)
    追加)十分な数の署名が収集されたら
### `SIGN_MODE_DIRECT_RELAXED`

(_https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-628026933_のオプション(1)(a)として文書化されています)

これは[SIGN_MODE_DIRECT]の変形であり、複数の署名者は必要ありません
公開鍵と署名モードを事前に調整します。これには代替手段が含まれます
`SignDoc`は上記の` SignDocAux`と似ていますが、有料です。これは将来追加することができます
クライアント開発者が公開鍵とパターンを事前に収集する負担を見つけた場合
面倒すぎる。

## 結果

### ポジティブ

-大幅なパフォーマンスの向上。
-後方および前方タイプの互換性をサポートします。
-クロスランゲージクライアントのサポートが向上しました。
-複数のシグニチャモードにより、より大きなプロトコルの進化が可能になります

### ネガティブ

-`google.protobuf.Any`タイプのURLはトランザクションサイズを増やします
   それは無視できるかもしれません、または圧縮はそれを軽減することができます。

### ニュートラル

## 参照 