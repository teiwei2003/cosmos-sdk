# ADR 43:NFT 模块

## 变更日志

- 2021-05-01:初稿
- 2021-07-02:审核更新

## 地位

建议的

## 摘要

该 ADR 定义了 `x/nft` 模块，它是 NFT 的通用实现，与 ERC721 大致“兼容”。 **使用 `x/nft` 模块的应用程序必须实现以下功能**:

- `MsgNewClass` - 接收用户创建类的请求，调用`x/nft`模块的`NewClass`。
- `MsgUpdateClass` - 接收用户更新类的请求，并调用`x/nft`模块的`UpdateClass`。
- `MsgMintNFT` - 接收用户创建 nft 的请求，并调用 `x/nft` 模块的 `MintNFT`。
- `BurnNFT` - 接收用户的烧录nft请求，调用`x/nft`模块的`BurnNFT`。
- `UpdateNFT` - 接收用户更新 nft 的请求，并调用 `x/nft` 模块的 `UpdateNFT`。

## 语境

NFT 不仅仅是加密艺术，这对于为 Cosmos 生态系统积累价值非常有帮助。因此，Cosmos Hub 应实现 NFT 功能并启用统一机制来存储和发送 NFT 的所有权代表，如 https://github.com/cosmos/cosmos-sdk/discussions/9065 中所述。

如 [#9065](https://github.com/cosmos/cosmos-sdk/discussions/9065) 中所述，可以考虑几种潜在的解决方案:

- irismod/nft 和模块/孵化器/nft
- CW721
- 做了 NFT
- 跨NFT

由于 NFT 的功能/用例与其逻辑紧密相连，因此通过定义和实现不同的交易类型，几乎不可能在一个 Cosmos SDK 模块中支持所有 NFT 的用例。

考虑到包括 IBC 和 Gravity Bridge 在内的链间协议的通用用法和兼容性，最好采用通用 NFT 模块设计来处理通用 NFT 逻辑。
这种设计思想可以实现可组合性，即通过导入 NFT 模块，特定应用程序的功能应该由 Cosmos Hub 或其他 Zones 上的其他模块管理。

当前的设计基于 [IRISnet 团队](https://github.com/irisnet/irismod/tree/master/modules/nft) 和 [Cosmos 存储库](https:// github.com/cosmos/modules/tree/master/incubator/nft)。

## 决定

我们创建了一个 `x/nft` 模块，它包含以下功能:

- 存储 NFT 并跟踪其所有权。
- 公开`Keeper` 接口，用于组合模块以传输、铸造和燃烧 NFT。
- 公开外部“消息”接口，供用户转移其 NFT 的所有权。
- 查询 NFT 及其供应信息。

提议的模块是 NFT 应用程序逻辑的基础模块。它的目标是为存储、基本传输功能和 IBC 提供一个公共层。该模块不应单独使用。
相反，应用程序应该创建一个专门的模块来处理应用程序特定的逻辑(例如:NFT ID 构建、版税)、用户级别的铸造和刻录。此外，应用程序专用模块应处理辅助数据以支持应用程序逻辑(例如索引、ORM、业务数据)。 

IBC 上携带的所有数据必须是下面描述的“NFT”或“Class”类型的一部分。 应用程序特定的 NFT 数据应在“NFT.data”中编码以实现跨链完整性。 与 NFT 相关的其他对完整性不重要的对象可以是应用程序特定模块的一部分。

### 类型

我们建议两种主要类型:
+ `Class` -- 描述 NFT 类。 我们可以将其视为智能合约地址。
+ `NFT` -- 代表独特的、不可替代的资产的对象。 每个 NFT 都与一个类相关联。

#### 班级

NFT **类**类似于 ERC-721 智能合约(提供智能合约的描述)，在该合约下可以创建和管理 NFT 的集合。 

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

- `id` 是 NFT 类的字母数字标识符； 它用作存储类的主要索引； _必需的_
- `name` 是 NFT 类的描述性名称； _选修的_
- `symbol` 是 NFT 类交易所通常显示的符号； _选修的_
- `description` 是对 NFT 类的详细描述； _选修的_
- `uri` 是链下存储的类元数据的 URI。 它应该是一个 JSON 文件，其中包含有关 NFT 类和 NFT 数据模式的元数据([OpenSea 示例](https://docs.opensea.io/docs/contract-level-metadata))； _选修的_
- `uri_hash` 是 uri 指向的文档的哈希值； _选修的_
- `data` 是类的特定于应用程序的元数据； _选修的_

#### NFT

我们为“NFT”定义了一个通用模型，如下所示。 

```protobuf
message NFT {
  string class_id           = 1;
  string id                 = 2;
  string uri                = 3;
  string uri_hash           = 4;
  google.protobuf.Any data  = 10;
}
```

- `class_id` 是 NFT 所属的 NFT 类的标识符； _必需的_
- `id` 是 NFT 的字母数字标识符，在其类范围内是唯一的。 它由 NFT 的创建者指定，将来可能会扩展为使用 DID。 `class_id` 结合 `id` 唯一标识一个 NFT，作为存储 NFT 的主索引； _必需的_ 

  ```
  {class_id}/{id} --> NFT (bytes)
  ```

- `uri` 是链下存储的 NFT 元数据的 URI。 应指向包含有关此 NFT 的元数据的 JSON 文件(参考:[ERC721 标准和 OpenSea 扩展](https://docs.opensea.io/docs/metadata-standards))； _必需的_
- `uri_hash` 是 uri 指向的文档的哈希值； _选修的_
- `data` 是 NFT 的应用程序特定数据。 可以通过组合模块使用来指定 NFT 的附加属性； _选修的_

此 ADR 未指定 `data` 可以采用的值； 然而，最佳实践建议上层 NFT 模块明确指定其内容。 尽管该字段的值不提供管理 NFT 记录所需的额外上下文，这意味着该字段可以从技术上从规范中删除，但该字段的存在允许基本的信息/UI 功能。

### `Keeper` 接口 

```go
type Keeper interface {
  NewClass(class Class)
  UpdateClass(class Class)

  Mint(nft NFT，receiver sdk.AccAddress)   // updates totalSupply
  Burn(classId string, nftId string)    // updates totalSupply
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

其他业务逻辑实现应该在组合模块中定义，这些模块导入 `x/nft` 并使用它的 `Keeper`。 

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

`MsgSend` 可用于将 NFT 的所有权转移到另一个地址。

服务器的实现大纲如下: 

```go
type msgServer struct{
  k Keeper
}

func (m msgServer) Send(ctx context.Context, msg *types.MsgSend) (*types.MsgSendResponse, error) {
  // check current ownership
  assertEqual(msg.Sender, m.k.GetOwner(msg.ClassId, msg.Id))

  // transfer ownership
  m.k.Transfer(msg.ClassId, msg.Id, msg.Receiver)

  return &types.MsgSendResponse{}, nil
}
```

`x/nft` 模块的查询服务方法有: 

```proto
service Query {

  // Balance queries the number of NFTs of a given class owned by the owner, same as balanceOf in ERC721
  rpc Balance(QueryBalanceRequest) returns (QueryBalanceResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/balance/{class_id}/{owner}";
  }

  // Owner queries the owner of the NFT based on its class and id, same as ownerOf in ERC721
  rpc Owner(QueryOwnerRequest) returns (QueryOwnerResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/owner/{class_id}/{id}";
  }

  // Supply queries the number of NFTs of a given class, same as totalSupply in ERC721Enumerable
  rpc Supply(QuerySupplyRequest) returns (QuerySupplyResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/supply/{class_id}";
  }

  // NFTsOfClassByOwner queries the NFTs of a given class owned by the owner, similar to tokenOfOwnerByIndex in ERC721Enumerable
  rpc NFTsOfClassByOwner(QueryNFTsOfClassByOwnerRequest) returns (QueryNFTsResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/owned_nfts/{class_id}/{owner}";
  }

  // NFTsOfClass queries all NFTs of a given class, similar to tokenByIndex in ERC721Enumerable
  rpc NFTsOfClass(QueryNFTsOfClassRequest) returns (QueryNFTsResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/nfts/{class_id}";
  }

  // NFT queries an NFT based on its class and id.
  rpc NFT(QueryNFTRequest) returns (QueryNFTResponse) {
    option (google.api.http).get = "/cosmos/nft/v1beta1/nfts/{class_id}/{id}";
  }

  // Class queries an NFT class based on its id
  rpc Class(QueryClassRequest) returns (QueryClassResponse) {
      option (google.api.http).get = "/cosmos/nft/v1beta1/classes/{class_id}";
  }

  // Classes queries all NFT classes
  rpc Classes(QueryClassesRequest) returns (QueryClassesResponse) {
      option (google.api.http).get = "/cosmos/nft/v1beta1/classes";
  }
}

// QueryBalanceRequest is the request type for the Query/Balance RPC method
message QueryBalanceRequest {
  string class_id = 1;
  string owner    = 2;
}

// QueryBalanceResponse is the response type for the Query/Balance RPC method
message QueryBalanceResponse{
  uint64 amount = 1;
}

// QueryOwnerRequest is the request type for the Query/Owner RPC method
message QueryOwnerRequest {
  string class_id = 1;
  string id       = 2;
}

// QueryOwnerResponse is the response type for the Query/Owner RPC method
message QueryOwnerResponse{
  string owner = 1;
}

// QuerySupplyRequest is the request type for the Query/Supply RPC method
message QuerySupplyRequest {
  string class_id = 1;
}

// QuerySupplyResponse is the response type for the Query/Supply RPC method
message QuerySupplyResponse {
  uint64 amount = 1;
}

// QueryNFTsOfClassByOwnerRequest is the request type for the Query/NFTsOfClassByOwner RPC method
message QueryNFTsOfClassByOwnerRequest {
  string                                 class_id   = 1;
  string                                 owner      = 2;
  cosmos.base.query.v1beta1.PageResponse pagination = 3;
}

// QueryNFTsOfClassRequest is the request type for the Query/NFTsOfClass RPC method
message QueryNFTsOfClassRequest {
  string                                 class_id   = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}

// QueryNFTsResponse is the response type for the Query/NFTsOfClass and Query/NFTsOfClassByOwner RPC methods
message QueryNFTsResponse {
  repeated cosmos.nft.v1beta1.NFT        nfts       = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}

// QueryNFTRequest is the request type for the Query/NFT RPC method
message QueryNFTRequest {
  string class_id = 1;
  string id       = 2;
}

// QueryNFTResponse is the response type for the Query/NFT RPC method
message QueryNFTResponse {
  cosmos.nft.v1beta1.NFT nft = 1;
}

// QueryClassRequest is the request type for the Query/Class RPC method
message QueryClassRequest {
  string class_id = 1;
}

// QueryClassResponse is the response type for the Query/Class RPC method
message QueryClassResponse {
  cosmos.nft.v1beta1.Class class = 1;
}

// QueryClassesRequest is the request type for the Query/Classes RPC method
message QueryClassesRequest {
  // pagination defines an optional pagination for the request.
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

// QueryClassesResponse is the response type for the Query/Classes RPC method
message QueryClassesResponse {
  repeated cosmos.nft.v1beta1.Class      classes    = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```

### Interoperability

互操作性就是在模块和链之间重用资产。前一种是通过 ADR-33 实现的:Protobuf 客户端-服务器通信。在撰写本文时，ADR-33 尚未最终确定。后者是由 IBC 实现的。在这里，我们将重点介绍 IBC 方面。
IBC 是按模块实现的。在这里，我们一致认为 NFT 将在 x/nft 中记录和管理。这需要创建一个新的 IBC 标准并实施它。

对于 IBC 互操作性，NFT 自定义模块必须使用 IBC 客户端理解的 NFT 对象类型。因此，对于 x/nft 互操作性，自定义 NFT 实现(例如:x/cryptokitty)应使用规范的 x/nft 模块并将所有 NFT 平衡保持功能代理到 x/nft，否则使用理解的 NFT 对象类型重新实现所有功能由 IBC 客户提供。换句话说:x/nft 成为所有 Cosmos NFT 的标准 NFT 注册表(例如:x/cryptokitty 将在 x/nft 中注册一个 kitty NFT 并使用 x/nft 进行簿记)。这是在使用 x/bank 作为一般资产平衡簿的背景下 [讨论](https://github.com/cosmos/cosmos-sdk/discussions/9065#discussioncomment-873206)。不使用 x/nft 将需要为 IBC 实现另一个模块。 

## Consequences

### Backward Compatibility

No backward incompatibilities.

### Forward Compatibility

该规范符合 NFT 标识符的 ERC-721 智能合约规范。 请注意，ERC-721 定义了基于(合约地址，uint256 tokenId)的唯一性，我们隐含地遵守这一点，因为当前单个模块旨在跟踪 NFT 标识符。 注意:使用(可变)数据字段来确定唯一性是不安全的。 

### Positive

- Cosmos Hub 上可用的 NFT 标识符。
- 能够为 Cosmos Hub 构建不同的 NFT 模块，例如 ERC-721。
- NFT 模块，支持与 IBC 和 Gravity Bridge 等其他跨链基础设施的互操作性 

### Negative

+ x/nft 需要新的 IBC 应用程序
+ 需要 CW721 适配器 
### Neutral

- 其他功能需要更多模块。 例如，NFT 交易功能需要一个托管模块，定义 NFT 属性需要一个收藏模块。 
## Further Discussions

对于 Hub 上的其他类型的应用，未来可以开发更多特定于应用的模块:

- `x/nft/custody`:托管 NFT 以支持交易功能。
- `x/nft/marketplace`:使用 sdk.Coins 买卖 NFT。
- `x/fractional`:为多个利益相关者分割资产(NFT 或其他资产)所有权的模块。 `x/group` 应该适用于大多数情况。

Cosmos 生态系统中的其他网络可以为特定的 NFT 应用程序和用例设计和实现自己的 NFT 模块。

## 参考

- 初步讨论:https://github.com/cosmos/cosmos-sdk/discussions/9065
- x/nft:初始化模块:https://github.com/cosmos/cosmos-sdk/pull/9174
- [ADR 033](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-033-protobuf-inter-module-comm.md) 