# 状態

現在、`x/evidence`モジュールは有効な送信済み`Evidence`のみを状態で保存します。
エビデンス状態は、`x/evidence`モジュールの`GenesisState`にも保存およびエクスポートされます。

```protobuf
//GenesisState defines the evidence module's genesis state.
message GenesisState {
 //evidence defines all the evidence at genesis.
  repeated google.protobuf.Any evidence = 1;
}

```

すべての`Evidence`は、プレフィックス`0x00`(`KeyPrefixEvidence`)を使用してプレフィックス`KVStore`を介して取得および保存されます。
