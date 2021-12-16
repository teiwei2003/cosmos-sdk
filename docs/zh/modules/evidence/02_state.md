# 状态

目前，`x/evidence` 模块只存储有效提交的 `Evidence` 状态。
证据状态也存储和导出在 `x/evidence` 模块的 `GenesisState` 中。

```protobuf
// GenesisState defines the evidence module's genesis state.
message GenesisState {
  // evidence defines all the evidence at genesis.
  repeated google.protobuf.Any evidence = 1;
}

```

所有“证据”都通过前缀“KVStore”使用前缀“0x00”(“KeyPrefixEvidence”)检索和存储。 
