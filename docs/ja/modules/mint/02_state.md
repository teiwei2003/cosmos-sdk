# 状態

## ミンター

ミンターは、現在のインフレ情報を保持するためのスペースです。

- ミンター: `0x00 -> ProtocolBuffer(minter)`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc7/proto/cosmos/mint/v1beta1/mint.proto#L8-L19

## パラメータ

ミンティングパラメータは、グローバルパラメータストアに保持されます。

- パラメータ:`mint/params -> legacy_amino(params)`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc7/proto/cosmos/mint/v1beta1/mint.proto#L21-L53 