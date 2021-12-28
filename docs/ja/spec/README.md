#  仕様

このディレクトリには、Cosmos SDKのモジュールの仕様、Interchain Standards(ICS)およびその他の仕様が含まれています。

Cosmos SDKアプリケーションは、この状態をMerkleストアに保持します。 の更新
ストアは、トランザクション中、およびすべての開始時と終了時に作成される場合があります
ブロック。

## CosmosSDKの仕様

- [Store](./store)-状態を保持するコアMerkleストア。
- [Bech32](./addresss/bech32.md)-CosmosSDKアプリケーションのアドレス形式。

## モジュールの仕様

[モジュールディレクトリ](../../x/README.md)に移動します

## テンダーミント

基盤となるブロックチェーンおよびp2pプロトコルの詳細については、を参照してください。
[Tendermint仕様](https://github.com/tendermint/spec/tree/master/spec)。