# 状態

## 署名情報(活性度)

すべてのブロックには、Tendermintによって提供される`LastCommitInfo`として知られる、前のブロックのバリデーターによる一連の事前コミットが含まれています。`LastCommitInfo`は、総投票権の+2/3からの事前コミットが含まれている限り有効です。

提案者は、追加料金を受け取ることにより、Tendermint`LastCommitInfo`にすべてのバリデーターからの事前コミットを含めるように奨励されています
`LastCommitInfo`に含まれる投票権と+2/3の差に比例します([料金配分(x/distribution/spec/03_begin_block.md)を参照)。

```
type LastCommitInfo struct {
	Round int32
	Votes []VoteInfo
}
```

バリデーターは、自動的に投獄され、スラッシュされる可能性があり、結合が解除されることにより、いくつかのブロックの`LastCommitInfo`に含まれなかった場合にペナルティが課せられます。。

バリデーターの活性活動に関する情報は、`ValidatorSigningInfo`を通じて追跡されます。
次のようにストアでインデックスが作成されます。

- ValidatorSigningInfo:`0x01 | ConsAddrLen (1 字节) | ConsAddress -> ProtocolBuffer(ValSigningInfo)`
- MissedBlocksBitArray:`0x02 | ConsAddrLen (1 字节) |约束地址 | LittleEndianUint64(signArrayIndex) -> VarInt(didMiss)`(varintは数値エンコード形式です)

最初のマッピングにより、バリデーターのコンセンサスアドレスに基づいて、バリデーターの最近の署名情報を簡単に検索できます。

2番目のマッピング(`MissedBlocksBitArray`)は、サイズが` SignedBlocksWindow`のビット配列として機能し、バリデーターがビット配列内の特定のインデックスのブロックを見逃したかどうかを通知します。 ビット配列のインデックスは、リトルエンディアンのuint64として指定されます。
結果は、`0`または` 1`をとる`varint`です。ここで、` 0`は
バリデーターは対応するブロックを見逃しませんでした(署名しました)。`1`はブロックを見逃した(署名しなかった)ことを示します。

`MissedBlocksBitArray`は事前に明示的に初期化されていないことに注意してください。
新しく結合されたバリデーターの最初の`SignedBlocksWindow`ブロックを進むにつれて、キーが追加されます。`SignedBlocksWindow`パラメーターは、バリデーターの活性を追跡するために使用されるスライディングウィンドウのサイズ(ブロック数)を定義します。

バリデーターの活性を追跡するために保存される情報は次のとおりです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/slashing/v1beta1/slashing.proto#L11-L33 