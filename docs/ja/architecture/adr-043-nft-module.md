# ADR 43:NFTモジュール

## 変更ログ

-2021-05-01:最初のドラフト
-2021-07-02:レビューの更新

## 状態

提案

## 概要

ADRは `x .nft`モジュールを定義します。これは、NFTの一般的な実装であり、ERC721とほぼ「互換性があります」。 ** `x .nft`モジュールを使用するアプリは、次の機能を実装する必要があります**:

-`MsgNewClass`-クラスを作成するというユーザーのリクエストを受け取り、 `x .nft`モジュールの` NewClass`を呼び出します。
-`MsgUpdateClass`-ユーザーの更新クラス要求を受け取り、 `x .nft`モジュールの` UpdateClass`を呼び出します。
-`MsgMintNFT`-nftを作成するユーザーの要求を受け取り、 `x .nft`モジュールの` MintNFT`を呼び出します。
-`BurnNFT`-ユーザーの書き込みnftリクエストを受信し、 `x .nft`モジュールの` BurnNFT`を呼び出します。
-`UpdateNFT`-nftを更新するユーザーの要求を受け取り、 `x .nft`モジュールの` UpdateNFT`を呼び出します。

## 環境

NFTは単なる暗号技術ではなく、Cosmosエコシステムの価値を蓄積するのに非常に役立ちます。したがって、https://github.com/cosmos/cosmos-sdk/discussions/9065で説明されているように、Cosmos HubはNFT機能を実装し、統合メカニズムがNFTの所有権を保存して送信できるようにする必要があります。

[#9065](https://github.com/cosmos/cosmos-sdk/discussions/9065)で説明されているように、いくつかの考えられる解決策を検討できます。

-irismod .nftおよびmodule .incubator .nft
-CW721
-NFTを行う
-クロスNFT

NFTの機能/ユースケースはロジックと密接に関連しているため、さまざまなトランザクションタイプを定義して実装することにより、1つのCosmosSDKモジュールでNFTのすべてのユースケースをサポートすることはほとんど不可能です。

IBCやGravityBridgeを含むチェーン間プロトコルの一般的な使用法と互換性を考慮すると、一般的なNFTロジックを処理するために一般的なNFTモジュール設計を採用するのが最善です。
この設計アイデアは、構成可能性を実現できます。つまり、NFTモジュールをインポートすることにより、特定のアプリケーションの機能をCosmosハブまたは他のゾーンの他のモジュールで管理する必要があります。

現在の設計は、[IRISnetチーム](https://github.com/irisnet/irismod/tree/master/modules/nft)と[Cosmosリポジトリ](https://github.com/cosmos/modules/tree)に基づいています。 .master .incubator .nft)。

## 決定

次の関数を含む `x .nft`モジュールを作成しました。

-NFTを保存し、その所有権を追跡します。
-パブリック `Keeper`インターフェース。モジュールを組み合わせて、NFTを送信、キャスト、および書き込むために使用されます。
-ユーザーがNFTの所有権を譲渡するための外部「メッセージ」インターフェースを開きます。
-NFTとその供給情報についてお問い合わせください。

提案されたモジュールは、NFTアプリケーションロジックの基本モジュールです。その目標は、ストレージ、基本的な伝送機能、およびIBCに共通のレイヤーを提供することです。このモジュールは単独で使用しないでください。
代わりに、アプリケーションは、アプリケーション固有のロジック(NFT IDの構築、使用料など)、ユーザーレベルのキャスト、および書き込みを処理するための専用モジュールを作成する必要があります。さらに、アプリケーション固有のモジュールは、アプリケーションロジック(インデックス、ORM、ビジネスデータなど)をサポートするために補助データを処理する必要があります。

IBCで伝送されるすべてのデータは、以下で説明する「NFT」または「クラス」タイプの一部である必要があります。クロスチェーンの整合性を実現するには、アプリケーション固有のNFTデータを「NFT.data」にエンコードする必要があります。完全性にとって重要ではないNFTに関連する他のオブジェクトは、アプリケーションの特定のモジュールの一部である可能性があります。

### タイプ

主に2つのタイプをお勧めします。
+ `Class`-NFTクラスについて説明します。スマートコントラクトアドレスと考えることができます。
+ `NFT`-ユニークでかけがえのない資産を表すオブジェクト。各NFTはクラスに関連付けられています。

#### クラス

NFT **クラス**はERC-721スマートコントラクト(スマートコントラクトの説明を提供します)に似ており、その下でNFTのコレクションを作成および管理できます。 

```protobuf
message Class {
  string id          = 1;
  string name        = 2;
  string symbol      = 3;
  string description = 4;
  string uri         = 5;
  string uri_hash    = 6;
  google.protobuf.Any data = 7;
}
```

-`id`はNFTクラスの英数字の識別子であり、ストレージクラスのメインインデックスとして使用されます; _Required_
-`name`はNFTクラスの説明的な名前です; _optional_
-`symbol`は通常NFT取引所に表示されるシンボルです; _optional_
-`description`はNFTクラスの詳細な説明です; _optional_
-`uri`は、チェーンの下に格納されているクラスメタデータのURIです。 NFTクラスとNFTデータモデルに関するメタデータを含むJSONファイルである必要があります([OpenSeaの例](https://docs.opensea.io/docs/contract-level-metadata)); _optional_
-`uri_hash`は、uriが指すドキュメントのハッシュ値です; _optional_
-`data`は、クラスのアプリケーション固有のメタデータです。_optional_

#### NFT

以下に示すように、「NFT」の一般的なモデルを定義しました。  

```protobuf
message NFT {
  string class_id           = 1;
  string id                 = 2;
  string uri                = 3;
  string uri_hash           = 4;
  google.protobuf.Any data  = 10;
}
```

-`class_id`は、NFTが属するNFTクラスの識別子です。_Required_
-`id`は、NFTの英数字の識別子であり、そのクラス範囲内で一意です。 これはNFTの作成者によって指定され、将来DIDを使用するように拡張される可能性があります。 `class_id`を` id`と組み合わせて、NFTを格納するためのメインインデックスとしてNFTを一意に識別します; _Required_ 

  ```
  {class_id}/{id} --> NFT (bytes)
  ```

-`uri`は、チェーンの下に保存されているNFTメタデータのURIです。 このNFTに関するメタデータを含むJSONファイルを指す必要があります(参照:[ERC721標準およびOpenSea拡張機能](https://docs.opensea.io/docs/metadata-standards)); _Required_
-`uri_hash`は、uriが指すドキュメントのハッシュ値です; _optional_
-`data`はNFTのアプリケーション固有のデータです。 組み合わせたモジュールを使用して、NFTの追加属性を指定できます; _optional_

このADRは、 `data`が取ることができる値を指定しませんが、ベストプラクティスでは、上位レベルのNFTモジュールがその内容を明示的に指定することをお勧めします。 このフィールドの値は、NFTレコードを管理するために必要な追加のコンテキストを提供しません。つまり、このフィールドは仕様から技術的に削除できますが、このフィールドの存在により、基本的な情報.UI機能が可能になります。

### `Keeper`インターフェース 

```go
type Keeper interface {
  NewClass(class Class)
  UpdateClass(class Class)

  Mint(nft NFT，receiver sdk.AccAddress)  ..updates totalSupply
  Burn(classId string, nftId string)   ..updates totalSupply
  Update(nft NFT)
  Transfer(classId string, nftId string, receiver sdk.AccAddress)

  GetClass(classId string) Class
  GetClasses() []Class

  GetNFT(classId string, nftId string) NFT
  GetNFTsOfClassByOwner(classId string, owner sdk.AccAddress) []NFT
  GetNFTsOfClass(classId string) []NFT

  GetOwner(classId string, nftId string) sdk.AccAddress
  GetBalance(classId string, owner sdk.AccAddress) uint64
  GetTotalSupply(classId string) uint64
}
```

他のビジネスロジックの実装は、 `x .nft`をインポートし、その` Keeper`を使用する複合モジュールで定義する必要があります。 

### `Msg` Service

```protobuf
service Msg {
  rpc Send(MsgSend)         returns (MsgSendResponse);
}

message MsgSend {
  string class_id = 1;
  string id       = 2;
  string sender   = 3;
  string reveiver = 4;
}
message MsgSendResponse {}
```

`MsgSend`を使用して、NFTの所有権を別のアドレスに譲渡できます。

サーバーの実装概要は次のとおりです。 

```go
type msgServer struct{
  k Keeper
}

func (m msgServer) Send(ctx context.Context, msg *types.MsgSend) (*types.MsgSendResponse, error) {
 ..check current ownership
  assertEqual(msg.Sender, m.k.GetOwner(msg.ClassId, msg.Id))

 ..transfer ownership
  m.k.Transfer(msg.ClassId, msg.Id, msg.Receiver)

  return &types.MsgSendResponse{}, nil
}
```

`x .nft`モジュールのクエリサービスメソッドは次のとおりです。 

```proto
service Query {

 ..Balance queries the number of NFTs of a given class owned by the owner, same as balanceOf in ERC721
  rpc Balance(QueryBalanceRequest) returns (QueryBalanceResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/balance/{class_id}/{owner}";
  }

 ..Owner queries the owner of the NFT based on its class and id, same as ownerOf in ERC721
  rpc Owner(QueryOwnerRequest) returns (QueryOwnerResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/owner/{class_id}/{id}";
  }

 ..Supply queries the number of NFTs of a given class, same as totalSupply in ERC721Enumerable
  rpc Supply(QuerySupplyRequest) returns (QuerySupplyResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/supply/{class_id}";
  }

 ..NFTsOfClassByOwner queries the NFTs of a given class owned by the owner, similar to tokenOfOwnerByIndex in ERC721Enumerable
  rpc NFTsOfClassByOwner(QueryNFTsOfClassByOwnerRequest) returns (QueryNFTsResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/owned_nfts/{class_id}/{owner}";
  }

 ..NFTsOfClass queries all NFTs of a given class, similar to tokenByIndex in ERC721Enumerable
  rpc NFTsOfClass(QueryNFTsOfClassRequest) returns (QueryNFTsResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/nfts/{class_id}";
  }

 ..NFT queries an NFT based on its class and id.
  rpc NFT(QueryNFTRequest) returns (QueryNFTResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/nfts/{class_id}/{id}";
  }

 ..Class queries an NFT class based on its id
  rpc Class(QueryClassRequest) returns (QueryClassResponse) {
      option (google.api.http).get = "/cosmos/nft/v1beta1/classes/{class_id}";
  }

 ..Classes queries all NFT classes
  rpc Classes(QueryClassesRequest) returns (QueryClassesResponse) {
      option (google.api.http).get = "/cosmos/nft/v1beta1/classes";
  }
}

/.QueryBalanceRequest is the request type for the Query/Balance RPC method
message QueryBalanceRequest {
  string class_id = 1;
  string owner    = 2;
}

/.QueryBalanceResponse is the response type for the Query/Balance RPC method
message QueryBalanceResponse{
  uint64 amount = 1;
}

/.QueryOwnerRequest is the request type for the Query/Owner RPC method
message QueryOwnerRequest {
  string class_id = 1;
  string id       = 2;
}

/.QueryOwnerResponse is the response type for the Query/Owner RPC method
message QueryOwnerResponse{
  string owner = 1;
}

/.QuerySupplyRequest is the request type for the Query/Supply RPC method
message QuerySupplyRequest {
  string class_id = 1;
}

/.QuerySupplyResponse is the response type for the Query/Supply RPC method
message QuerySupplyResponse {
  uint64 amount = 1;
}

/.QueryNFTsOfClassByOwnerRequest is the request type for the Query/NFTsOfClassByOwner RPC method
message QueryNFTsOfClassByOwnerRequest {
  string                                 class_id   = 1;
  string                                 owner      = 2;
  cosmos.base.query.v1beta1.PageResponse pagination = 3;
}

/.QueryNFTsOfClassRequest is the request type for the Query/NFTsOfClass RPC method
message QueryNFTsOfClassRequest {
  string                                 class_id   = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}

/.QueryNFTsResponse is the response type for the Query/NFTsOfClass and Query/NFTsOfClassByOwner RPC methods
message QueryNFTsResponse {
  repeated cosmos.nft.v1beta1.NFT        nfts       = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}

/.QueryNFTRequest is the request type for the Query/NFT RPC method
message QueryNFTRequest {
  string class_id = 1;
  string id       = 2;
}

/.QueryNFTResponse is the response type for the Query/NFT RPC method
message QueryNFTResponse {
  cosmos.nft.v1beta1.NFT nft = 1;
}

/.QueryClassRequest is the request type for the Query/Class RPC method
message QueryClassRequest {
  string class_id = 1;
}

/.QueryClassResponse is the response type for the Query/Class RPC method
message QueryClassResponse {
  cosmos.nft.v1beta1.Class class = 1;
}

/.QueryClassesRequest is the request type for the Query/Classes RPC method
message QueryClassesRequest {
 ..pagination defines an optional pagination for the request.
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

/.QueryClassesResponse is the response type for the Query/Classes RPC method
message QueryClassesResponse {
  repeated cosmos.nft.v1beta1.Class      classes    = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```

### 相互運用性

相互運用性とは、モジュールとチェーンの間でアセットを再利用することです。前者は、ADR-33:Protobufクライアント/サーバー通信によって実現されます。執筆時点では、ADR-33はまだ完成していません。後者はIBCによって実装されています。ここでは、IBCの側面に焦点を当てます。
IBCはモジュールに実装されています。ここでは、NFTがx .nftで記録および管理されることに同意します。これには、新しいIBC標準を作成して実装する必要があります。

IBCの相互運用性のために、NFTカスタムモジュールは、IBCクライアントが理解できるNFTオブジェクトタイプを使用する必要があります。したがって、x .nftの相互運用性のために、カスタムNFT実装(例:x .cryptokitty)は、標準化されたx .nftモジュールを使用し、すべてのNFTバランス維持機能をx .nftにプロキシする必要があります。それ以外の場合は、理解されているNFTオブジェクトタイプを使用してすべての機能を再実行します。 IBCのお客様から提供されています。言い換えると、x .nftはすべてのCosmosNFTの標準NFTレジストリになります(たとえば、x .cryptokittyはキティNFTをx .nftに登録し、簿記にx .nftを使用します)。これは、x .bankを一般的な貸借対照表として使用する場合に使用されます[ディスカッション](https://github.com/cosmos/cosmos-sdk/discussions/9065#discussioncomment-873206)。 x .nftを使用しない場合は、IBC用に別のモジュールを実装する必要があります。

## 結果

### 下位互換性

後方互換性はありません。

### 上位互換性

この仕様は、NFT識別子のERC-721スマートコントラクト仕様に準拠しています。 ERC-721は(契約アドレス、uint256 tokenId)に基づいて一意性を定義し、現在単一のモジュールがNFT識別子を追跡するように設計されているため、これに暗黙的に準拠していることに注意してください。注:(変数)データフィールドを使用して一意性を判断することは安全ではありません。

### ポジティブ

-Cosmosハブで利用可能なNFT識別子。
-ERC-721などのCosmosHub用のさまざまなNFTモジュールを構築する機能。
-NFTモジュール、IBCやGravityBridgeなどの他のクロスチェーンインフラストラクチャとの相互運用性をサポート

### ネガティブ

+ x .nftには新しいIBCアプリケーションが必要です
+ CW721アダプターが必要
### ニュートラル

-他の機能には、より多くのモジュールが必要です。たとえば、NFTトランザクション機能には管理モジュールが必要であり、NFTプロパティの定義には収集モジュールが必要です。
## さらなる議論

ハブ上の他のタイプのアプリケーションについては、将来、より多くのアプリケーション固有のモジュールを開発できます。

-`x .nft .custody`:トランザクション機能をサポートするためのホストNFT。
-`x .nft .marketplace`:sdk.Coinsを使用してNFTを売買します。
-`x .fractional`:複数の利害関係者の資産(NFTまたはその他の資産)の所有権を分割するためのモジュール。 `x .group`はほとんどの状況に適しているはずです。

Cosmosエコシステムの他のネットワークは、特定のNFTアプリケーションおよびユースケース向けに独自のNFTモジュールを設計および実装できます。

## 参照する

-予備的なディスカッション:https://github.com/cosmos/cosmos-sdk/discussions/9065
-x .nft:初期化モジュール:https://github.com/cosmos/cosmos-sdk/pull/9174
-[ADR 033](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-033-protobuf-inter-module-comm.md) 