# ADR 036:任意のメッセージ署名仕様

## 変更ログ

-2020年10月28日-ドラフト

## 著者

-Antoine Herzog(@antoineherzog)
-ザキマニアン(@zmanian)
-Alexander Bezobchuk(alexanderbez)[1]
-Frojdi Dymylja(@fdymylja)

## 状態

下書き

## 概要

現在、Cosmos SDKでは、Ethereumのように任意のメッセージに署名することに合意していません。この仕様では、CosmosSDKエコシステムのメッセージをオフチェーンで署名および検証する方法を提案します。

この仕様は、すべてのユースケースをカバーすることを目的としています。つまり、cosmos-sdkアプリケーション開発者は、ユーザーに「データ」をシリアル化して表現する方法を決定します。

## 環境

オフチェーンでメッセージに署名できることは、ほとんどすべてのブロックチェーンの基本的な側面であることが証明されています。メッセージにオフチェーンで署名するという概念には、計算コストの節約、トランザクションのスループットとオーバーヘッドの削減など、多くの追加の利点があります。 Cosmosのコンテキストでは、このようなデータに署名するための主なアプリケーションには、検証者の身元を証明するための暗号的に安全で検証可能な方法の提供、および場合によっては他のフレームワークや組織へのリンクが含まれますが、これらに限定されません。さらに、元帳または同様のHSMデバイスを使用してCosmosメッセージに署名できます。

その他のコンテキストとユースケースは、リファレンスリンクにあります。

## 決定

目標は、元帳または同様のHSMデバイスを使用している場合でも、任意のメッセージに署名できるようにすることです。

したがって、署名されたメッセージはCosmos SDKメッセージとほぼ同じである必要がありますが、有効なオンチェーントランザクションになることは**できません**。 `chain-id`、` account_number`、 `sequence`にはすべて無効な値を割り当てることができます。

Cosmos SDK 0.40では、「auth_info」の概念も導入されました。SIGN_MODESを指定できます。

仕様には、SIGN_MODE_DIRECTおよびSIGN_MODE_LEGACY_AMINOをサポートする `auth_info`を含める必要があります。

`offchain`プロト定義を作成するには、` offchain`パッケージを使用して認証モジュールを拡張し、オフラインメッセージを検証して署名する機能を提供します。

オフチェーントランザクションは、次のルールに従います。

-メモは空である必要があります
-nonce、シリアル番号は0に等しくなければなりません
-チェーンIDは「」と同じである必要があります
-コストガスは0に等しくなければなりません
-料金の金額は空の配列である必要があります

上で強調した仕様の違いに加えて、オフチェーントランザクションの検証は、オンチェーントランザクションと同じルールに従います。

`offchain`パッケージに追加された最初のメッセージは` MsgSignData`です。

`MsgSignData`を使用すると、開発者は有効なオフチェーンバイトのみに署名できます。ここで、「署名者」は署名者のアカウントアドレスです。 `Data`は、` text`、 `files`、および` object`を表すことができる任意のバイトです。アプリケーション開発者は、「データ」をどのように逆シリアル化、シリアル化するか、およびそのコンテキストでどのオブジェクトを表すことができるかを決定します。

アプリケーション開発者は、「データ」の処理方法を決定します。処理するのは、シリアル化と逆シリアル化のプロセス、および「データ」が何を表すかです。

プロトタイプの定義: 

```proto
/.MsgSignData defines an arbitrary, general-purpose, off-chain message
message MsgSignData {
   ..Signer is the sdk.AccAddress of the message signer
    bytes Signer = 1 [(gogoproto.jsontag) = "signer", (gogoproto.casttype) = "github.com/cosmos/cosmos-sdk/types.AccAddress"];
   ..Data represents the raw bytes of the content that is signed (text, json, etc)
    bytes Data = 2 [(gogoproto.jsontag) = "data"];
}
```

MsgSignData jsonの例に署名します: 

```json
{
  "type": "cosmos-sdk/StdTx",
  "value": {
    "msg": [
      {
        "type": "sign/MsgSignData",
        "value": {
          "signer": "cosmos1hftz5ugqmpg9243xeegsqqav62f8hnywsjr4xr",
          "data": "cmFuZG9t"
        }
      }
    ],
    "fee": {
      "amount": [],
      "gas": "0"
    },
    "signatures": [
      {
        "pub_key": {
          "type": "tendermint/PubKeySecp256k1",
          "value": "AqnDSiRoFmTPfq97xxEb2VkQ/Hm28cPsqsZm9jEVsYK9"
        },
        "signature": "8y8i34qJakkjse9pOD2De+dnlc4KvFgh0wQpes4eydN66D9kv7cmCEouRrkka9tlW9cAkIL52ErB+6ye7X5aEg=="
      }
    ],
    "memo": ""
  }
}
```

## 結果

リアルタイムチェーンにブロードキャストすることを目的としていないメッセージを作成する方法に関する仕様があります。

### 下位互換性

これは新しいメッセージ仕様定義であるため、下位互換性が維持されます。

### ポジティブ

-オフチェーンメッセージに署名および検証するために複数のアプリケーションで使用できる一般的な形式。
-仕様は原始的です。つまり、仕様に適合するものを制限することなく、すべてのユースケースをカバーできます。
-オフチェーンauthN .authZレイヤー[2]など、より具体的で一般的なユースケースを対象とした他のオフチェーンメッセージ仕様のためのスペースを提供します。
### ネガティブ

-現在の提案では、アカウントアドレスと公開鍵の間に固定の関係が必要です。
-マルチシグニチャアカウントには適用されません。

## さらなる議論

-`MsgSignData`のセキュリティに関して、 `MsgSignData`を使用する開発者は、` Data`に配置されたコンテンツを必要なときに再生できないようにする責任があります。
-オフチェーンパッケージは、アプリケーション、支払いチャネル、一般的なL2ソリューションでの本人確認など、特定のユースケース向けの追加メッセージでさらに拡張されます。

## 参照

1. https://github.com/cosmos/ics/pull/33
2. https://github.com/cosmos/cosmos-sdk/pull/7727#discussion_r515668204
3. https://github.com/cosmos/cosmos-sdk/pull/7727#issuecomment-722478477
4. https://github.com/cosmos/cosmos-sdk/pull/7727#issuecomment-721062923 