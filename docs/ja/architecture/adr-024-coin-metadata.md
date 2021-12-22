# ADR 024:コインメタデータ

## 変更ログ

2020年5月19日:最初のドラフト

## 状態

提案

## 環境

Cosmos SDKのアセットは、「amount」と「denom」で構成される「Coins」タイプで表されます。
「金額」は、任意の大きい値または小さい値にすることができます。さらに、CosmosSDKは
アカウントベースのモデルには、基本アカウントとモジュールアカウントの2種類のメインアカウントがあります。
すべてのアカウントタイプには、「コイン」で構成される一連の残高があります。 `x .bank`モジュールの保持
すべてのアカウントのすべての残高を追跡し、1つのアカウントの残高の合計供給を追跡します
応用。

残高の「金額」に関して、Cosmos SDKは、静的および固定の額面単位を想定しています。
金種自体に関係なく。つまり、クライアントとアプリケーションはCosmos-SDKベースで構築されています
チェーンは、より豊かなユーザーエクスペリエンスを提供するために、任意の金種単位を定義して使用することを選択できますが、
txまたは操作がCosmosSDKステートマシンに到達すると、 `amount`は単一として扱われます
ユニット。たとえば、Cosmos Hub(Gaia)の場合、クライアントは1 ATOM = 10 ^ 6 uatomと想定しているため、すべてのtxと
Cosmos SDKの操作は、10 ^ 6の単位で機能します。

これは明らかに、特にネットワークの相互運用性が向上し、
その結果、資産タイプの総数が増加しました。 `x .bank`を保持することをお勧めします
各「デノム」のメタデータを追跡して、顧客、ウォレットプロバイダー、およびエクスプローラーが
UXを実行し、金種単位に関する仮定を行うための要件を削除します。

## 決定

`x .bank`モジュールは、` denom`、特に "base"または
最小単位-CosmosSDKステートマシンで使用される単位。

メタデータには、ゼロ以外の長さの宗派リストを含めることもできます。各エントリに含まれる名前
宗派 `denom`、ベースインデックスおよびエイリアスのリスト。エントリは
「1denom = 10 ^ exponent base_denom」として解釈されます(たとえば、「1 ETH = 10 ^ 18wei」および「1uatom = 10 ^ 0uatom」)。

顧客にとって非常に重要な2つの宗派があります:最小の「ベース」
人間のコミュニケーションで通常言及される単位である可能な単位と「ディスプレイ」
そして通信します。これらのフィールドの値は、金種リストのエントリにリンクされています。

`denom_units`および` display`エントリのリストは、管理によって変更できます。

したがって、タイプは次のように定義できます。 

```protobuf
message DenomUnit {
  string denom    = 1;
  uint32 exponent = 2;  
  repeated string aliases = 3;
}

message Metadata {
  string description = 1;
  repeated DenomUnit denom_units = 2;
  string base = 3;
  string display = 4;
}
```

たとえば、ATOMのメタデータは次のように定義できます。 

```json
{
  "description": "The native staking token of the Cosmos Hub.",
  "denom_units": [
    {
      "denom": "uatom",
      "exponent": 0,
      "aliases": [
        "microatom"
      ],
    },
    {
      "denom": "matom",
      "exponent": 3,
      "aliases": [
        "milliatom"
      ]
    },
    {
      "denom": "atom",
      "exponent": 6,
    }
  ],
  "base": "uatom",
  "display": "atom",
}
```

上記のメタデータが与えられると、クライアントは次のことを推測できます。

-4.3atom = 4.3 *(10 ^ 6)= 4,300,000uatom
-タグリストの表示名は文字列「atom」が使用できます。
-バランス4300000は、4,300,000uatomまたは4,300matomまたは4.3atomとして表示できます。
    クライアントの作成者が作成しなかった場合は、4.3atomの `display`単位が適切なデフォルト値です。
    別の表現を選択するという明確な決定。

クライアントは、CLIおよびRESTインターフェースを介して名前でメタデータを照会できる必要があります。 存在
さらに、これらのインターフェイスにハンドラーを追加して、任意のユニットを別の特定のユニットに変換します。
この基本的なフレームワークはすでにCosmosSDKに存在しているためです。

最後に、メタデータが `x .bank`モジュールの` GenesisState`に存在することを確認する必要があります。これも
ベース `denom`によってインデックス付けされます。  

```go
type GenesisState struct {
  SendEnabled   bool        `json:"send_enabled" yaml:"send_enabled"`
  Balances      []Balance   `json:"balances" yaml:"balances"`
  Supply        sdk.Coins   `json:"supply" yaml:"supply"`
  DenomMetadata []Metadata  `json:"denom_metadata" yaml:"denom_metadata"`
}
```

## 将来の仕事

顧客が資産を基本的な金種に変換する必要を回避できるようにするために-手動または
エンドポイントを通じて、特定の単位入力の自動変換をサポートすることを検討できます。
## 結果

### ポジティブ

-顧客、ウォレットプロバイダー、ブロックエクスプローラーに追加データを提供する
    ユーザーエクスペリエンスを改善し、仮定の必要性を排除するための資産の種類
    宗派単位。

### ネガティブ

-`x .bank`モジュールに必要な少量の追加ストレージスペース。 量
    総資産の量はすべきではないため、追加のストレージの量は最小限にする必要があります
    大きい。

### ニュートラル

## 参照 