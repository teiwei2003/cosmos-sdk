# Protobuf 文档
<a name="top"></a>

## 目录

- [cosmos/auth/v1beta1/auth.proto](#cosmos/auth/v1beta1/auth.proto)
    - [BaseAccount](#cosmos.auth.v1beta1.BaseAccount)
    - [ModuleAccount](#cosmos.auth.v1beta1.ModuleAccount)
    - [Params](#cosmos.auth.v1beta1.Params)
  
- [cosmos/auth/v1beta1/genesis.proto](#cosmos/auth/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.auth.v1beta1.GenesisState)
  
- [cosmos/base/query/v1beta1/pagination.proto](#cosmos/base/query/v1beta1/pagination.proto)
    - [PageRequest](#cosmos.base.query.v1beta1.PageRequest)
    - [PageResponse](#cosmos.base.query.v1beta1.PageResponse)
  
- [cosmos/auth/v1beta1/query.proto](#cosmos/auth/v1beta1/query.proto)
    - [AddressBytesToStringRequest](#cosmos.auth.v1beta1.AddressBytesToStringRequest)
    - [AddressBytesToStringResponse](#cosmos.auth.v1beta1.AddressBytesToStringResponse)
    - [AddressStringToBytesRequest](#cosmos.auth.v1beta1.AddressStringToBytesRequest)
    - [AddressStringToBytesResponse](#cosmos.auth.v1beta1.AddressStringToBytesResponse)
    - [Bech32PrefixRequest](#cosmos.auth.v1beta1.Bech32PrefixRequest)
    - [Bech32PrefixResponse](#cosmos.auth.v1beta1.Bech32PrefixResponse)
    - [QueryAccountRequest](#cosmos.auth.v1beta1.QueryAccountRequest)
    - [QueryAccountResponse](#cosmos.auth.v1beta1.QueryAccountResponse)
    - [QueryAccountsRequest](#cosmos.auth.v1beta1.QueryAccountsRequest)
    - [QueryAccountsResponse](#cosmos.auth.v1beta1.QueryAccountsResponse)
    - [QueryModuleAccountsRequest](#cosmos.auth.v1beta1.QueryModuleAccountsRequest)
    - [QueryModuleAccountsResponse](#cosmos.auth.v1beta1.QueryModuleAccountsResponse)
    - [QueryParamsRequest](#cosmos.auth.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.auth.v1beta1.QueryParamsResponse)
  
    - [Query](#cosmos.auth.v1beta1.Query)
  
- [cosmos/authz/v1beta1/authz.proto](#cosmos/authz/v1beta1/authz.proto)
    - [GenericAuthorization](#cosmos.authz.v1beta1.GenericAuthorization)
    - [Grant](#cosmos.authz.v1beta1.Grant)
  
- [cosmos/authz/v1beta1/event.proto](#cosmos/authz/v1beta1/event.proto)
    - [EventGrant](#cosmos.authz.v1beta1.EventGrant)
    - [EventRevoke](#cosmos.authz.v1beta1.EventRevoke)
  
- [cosmos/authz/v1beta1/genesis.proto](#cosmos/authz/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.authz.v1beta1.GenesisState)
    - [GrantAuthorization](#cosmos.authz.v1beta1.GrantAuthorization)
  
- [cosmos/authz/v1beta1/query.proto](#cosmos/authz/v1beta1/query.proto)
    - [QueryGranterGrantsRequest](#cosmos.authz.v1beta1.QueryGranterGrantsRequest)
    - [QueryGranterGrantsResponse](#cosmos.authz.v1beta1.QueryGranterGrantsResponse)
    - [QueryGrantsRequest](#cosmos.authz.v1beta1.QueryGrantsRequest)
    - [QueryGrantsResponse](#cosmos.authz.v1beta1.QueryGrantsResponse)
  
    - [Query](#cosmos.authz.v1beta1.Query)
  
- [cosmos/authz/v1beta1/tx.proto](#cosmos/authz/v1beta1/tx.proto)
    - [MsgExec](#cosmos.authz.v1beta1.MsgExec)
    - [MsgExecResponse](#cosmos.authz.v1beta1.MsgExecResponse)
    - [MsgGrant](#cosmos.authz.v1beta1.MsgGrant)
    - [MsgGrantResponse](#cosmos.authz.v1beta1.MsgGrantResponse)
    - [MsgRevoke](#cosmos.authz.v1beta1.MsgRevoke)
    - [MsgRevokeResponse](#cosmos.authz.v1beta1.MsgRevokeResponse)
  
    - [Msg](#cosmos.authz.v1beta1.Msg)
  
- [cosmos/base/v1beta1/coin.proto](#cosmos/base/v1beta1/coin.proto)
    - [Coin](#cosmos.base.v1beta1.Coin)
    - [DecCoin](#cosmos.base.v1beta1.DecCoin)
    - [DecProto](#cosmos.base.v1beta1.DecProto)
    - [IntProto](#cosmos.base.v1beta1.IntProto)
  
- [cosmos/bank/v1beta1/authz.proto](#cosmos/bank/v1beta1/authz.proto)
    - [SendAuthorization](#cosmos.bank.v1beta1.SendAuthorization)
  
- [cosmos/bank/v1beta1/bank.proto](#cosmos/bank/v1beta1/bank.proto)
    - [DenomUnit](#cosmos.bank.v1beta1.DenomUnit)
    - [Input](#cosmos.bank.v1beta1.Input)
    - [Metadata](#cosmos.bank.v1beta1.Metadata)
    - [Output](#cosmos.bank.v1beta1.Output)
    - [Params](#cosmos.bank.v1beta1.Params)
    - [SendEnabled](#cosmos.bank.v1beta1.SendEnabled)
    - [Supply](#cosmos.bank.v1beta1.Supply)
  
- [cosmos/bank/v1beta1/genesis.proto](#cosmos/bank/v1beta1/genesis.proto)
    - [Balance](#cosmos.bank.v1beta1.Balance)
    - [GenesisState](#cosmos.bank.v1beta1.GenesisState)
  
- [cosmos/bank/v1beta1/query.proto](#cosmos/bank/v1beta1/query.proto)
    - [DenomOwner](#cosmos.bank.v1beta1.DenomOwner)
    - [QueryAllBalancesRequest](#cosmos.bank.v1beta1.QueryAllBalancesRequest)
    - [QueryAllBalancesResponse](#cosmos.bank.v1beta1.QueryAllBalancesResponse)
    - [QueryBalanceRequest](#cosmos.bank.v1beta1.QueryBalanceRequest)
    - [QueryBalanceResponse](#cosmos.bank.v1beta1.QueryBalanceResponse)
    - [QueryDenomMetadataRequest](#cosmos.bank.v1beta1.QueryDenomMetadataRequest)
    - [QueryDenomMetadataResponse](#cosmos.bank.v1beta1.QueryDenomMetadataResponse)
    - [QueryDenomOwnersRequest](#cosmos.bank.v1beta1.QueryDenomOwnersRequest)
    - [QueryDenomOwnersResponse](#cosmos.bank.v1beta1.QueryDenomOwnersResponse)
    - [QueryDenomsMetadataRequest](#cosmos.bank.v1beta1.QueryDenomsMetadataRequest)
    - [QueryDenomsMetadataResponse](#cosmos.bank.v1beta1.QueryDenomsMetadataResponse)
    - [QueryParamsRequest](#cosmos.bank.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.bank.v1beta1.QueryParamsResponse)
    - [QuerySupplyOfRequest](#cosmos.bank.v1beta1.QuerySupplyOfRequest)
    - [QuerySupplyOfResponse](#cosmos.bank.v1beta1.QuerySupplyOfResponse)
    - [QueryTotalSupplyRequest](#cosmos.bank.v1beta1.QueryTotalSupplyRequest)
    - [QueryTotalSupplyResponse](#cosmos.bank.v1beta1.QueryTotalSupplyResponse)
  
    - [Query](#cosmos.bank.v1beta1.Query)
  
- [cosmos/bank/v1beta1/tx.proto](#cosmos/bank/v1beta1/tx.proto)
    - [MsgMultiSend](#cosmos.bank.v1beta1.MsgMultiSend)
    - [MsgMultiSendResponse](#cosmos.bank.v1beta1.MsgMultiSendResponse)
    - [MsgSend](#cosmos.bank.v1beta1.MsgSend)
    - [MsgSendResponse](#cosmos.bank.v1beta1.MsgSendResponse)
  
    - [Msg](#cosmos.bank.v1beta1.Msg)
  
- [cosmos/base/abci/v1beta1/abci.proto](#cosmos/base/abci/v1beta1/abci.proto)
    - [ABCIMessageLog](#cosmos.base.abci.v1beta1.ABCIMessageLog)
    - [Attribute](#cosmos.base.abci.v1beta1.Attribute)
    - [GasInfo](#cosmos.base.abci.v1beta1.GasInfo)
    - [MsgData](#cosmos.base.abci.v1beta1.MsgData)
    - [Result](#cosmos.base.abci.v1beta1.Result)
    - [SearchTxsResult](#cosmos.base.abci.v1beta1.SearchTxsResult)
    - [SimulationResponse](#cosmos.base.abci.v1beta1.SimulationResponse)
    - [StringEvent](#cosmos.base.abci.v1beta1.StringEvent)
    - [TxMsgData](#cosmos.base.abci.v1beta1.TxMsgData)
    - [TxResponse](#cosmos.base.abci.v1beta1.TxResponse)
  
- [cosmos/base/kv/v1beta1/kv.proto](#cosmos/base/kv/v1beta1/kv.proto)
    - [Pair](#cosmos.base.kv.v1beta1.Pair)
    - [Pairs](#cosmos.base.kv.v1beta1.Pairs)
  
- [cosmos/base/reflection/v1beta1/reflection.proto](#cosmos/base/reflection/v1beta1/reflection.proto)
    - [ListAllInterfacesRequest](#cosmos.base.reflection.v1beta1.ListAllInterfacesRequest)
    - [ListAllInterfacesResponse](#cosmos.base.reflection.v1beta1.ListAllInterfacesResponse)
    - [ListImplementationsRequest](#cosmos.base.reflection.v1beta1.ListImplementationsRequest)
    - [ListImplementationsResponse](#cosmos.base.reflection.v1beta1.ListImplementationsResponse)
  
    - [ReflectionService](#cosmos.base.reflection.v1beta1.ReflectionService)
  
- [cosmos/base/reflection/v2alpha1/reflection.proto](#cosmos/base/reflection/v2alpha1/reflection.proto)
    - [AppDescriptor](#cosmos.base.reflection.v2alpha1.AppDescriptor)
    - [AuthnDescriptor](#cosmos.base.reflection.v2alpha1.AuthnDescriptor)
    - [ChainDescriptor](#cosmos.base.reflection.v2alpha1.ChainDescriptor)
    - [CodecDescriptor](#cosmos.base.reflection.v2alpha1.CodecDescriptor)
    - [ConfigurationDescriptor](#cosmos.base.reflection.v2alpha1.ConfigurationDescriptor)
    - [GetAuthnDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetAuthnDescriptorRequest)
    - [GetAuthnDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetAuthnDescriptorResponse)
    - [GetChainDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetChainDescriptorRequest)
    - [GetChainDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetChainDescriptorResponse)
    - [GetCodecDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetCodecDescriptorRequest)
    - [GetCodecDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetCodecDescriptorResponse)
    - [GetConfigurationDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorRequest)
    - [GetConfigurationDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorResponse)
    - [GetQueryServicesDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorRequest)
    - [GetQueryServicesDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorResponse)
    - [GetTxDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetTxDescriptorRequest)
    - [GetTxDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetTxDescriptorResponse)
    - [InterfaceAcceptingMessageDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceAcceptingMessageDescriptor)
    - [InterfaceDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceDescriptor)
    - [InterfaceImplementerDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceImplementerDescriptor)
    - [MsgDescriptor](#cosmos.base.reflection.v2alpha1.MsgDescriptor)
    - [QueryMethodDescriptor](#cosmos.base.reflection.v2alpha1.QueryMethodDescriptor)
    - [QueryServiceDescriptor](#cosmos.base.reflection.v2alpha1.QueryServiceDescriptor)
    - [QueryServicesDescriptor](#cosmos.base.reflection.v2alpha1.QueryServicesDescriptor)
    - [SigningModeDescriptor](#cosmos.base.reflection.v2alpha1.SigningModeDescriptor)
    - [TxDescriptor](#cosmos.base.reflection.v2alpha1.TxDescriptor)
  
    - [ReflectionService](#cosmos.base.reflection.v2alpha1.ReflectionService)
  
- [cosmos/base/snapshots/v1beta1/snapshot.proto](#cosmos/base/snapshots/v1beta1/snapshot.proto)
    - [Metadata](#cosmos.base.snapshots.v1beta1.Metadata)
    - [Snapshot](#cosmos.base.snapshots.v1beta1.Snapshot)
  
- [cosmos/base/store/v1beta1/commit_info.proto](#cosmos/base/store/v1beta1/commit_info.proto)
    - [CommitID](#cosmos.base.store.v1beta1.CommitID)
    - [CommitInfo](#cosmos.base.store.v1beta1.CommitInfo)
    - [StoreInfo](#cosmos.base.store.v1beta1.StoreInfo)
  
- [cosmos/base/store/v1beta1/listening.proto](#cosmos/base/store/v1beta1/listening.proto)
    - [StoreKVPair](#cosmos.base.store.v1beta1.StoreKVPair)
  
- [cosmos/base/store/v1beta1/snapshot.proto](#cosmos/base/store/v1beta1/snapshot.proto)
    - [SnapshotIAVLItem](#cosmos.base.store.v1beta1.SnapshotIAVLItem)
    - [SnapshotItem](#cosmos.base.store.v1beta1.SnapshotItem)
    - [SnapshotStoreItem](#cosmos.base.store.v1beta1.SnapshotStoreItem)
  
- [cosmos/base/tendermint/v1beta1/query.proto](#cosmos/base/tendermint/v1beta1/query.proto)
    - [GetBlockByHeightRequest](#cosmos.base.tendermint.v1beta1.GetBlockByHeightRequest)
    - [GetBlockByHeightResponse](#cosmos.base.tendermint.v1beta1.GetBlockByHeightResponse)
    - [GetLatestBlockRequest](#cosmos.base.tendermint.v1beta1.GetLatestBlockRequest)
    - [GetLatestBlockResponse](#cosmos.base.tendermint.v1beta1.GetLatestBlockResponse)
    - [GetLatestValidatorSetRequest](#cosmos.base.tendermint.v1beta1.GetLatestValidatorSetRequest)
    - [GetLatestValidatorSetResponse](#cosmos.base.tendermint.v1beta1.GetLatestValidatorSetResponse)
    - [GetNodeInfoRequest](#cosmos.base.tendermint.v1beta1.GetNodeInfoRequest)
    - [GetNodeInfoResponse](#cosmos.base.tendermint.v1beta1.GetNodeInfoResponse)
    - [GetSyncingRequest](#cosmos.base.tendermint.v1beta1.GetSyncingRequest)
    - [GetSyncingResponse](#cosmos.base.tendermint.v1beta1.GetSyncingResponse)
    - [GetValidatorSetByHeightRequest](#cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightRequest)
    - [GetValidatorSetByHeightResponse](#cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightResponse)
    - [Module](#cosmos.base.tendermint.v1beta1.Module)
    - [Validator](#cosmos.base.tendermint.v1beta1.Validator)
    - [VersionInfo](#cosmos.base.tendermint.v1beta1.VersionInfo)
  
    - [Service](#cosmos.base.tendermint.v1beta1.Service)
  
- [cosmos/capability/v1beta1/capability.proto](#cosmos/capability/v1beta1/capability.proto)
    - [Capability](#cosmos.capability.v1beta1.Capability)
    - [CapabilityOwners](#cosmos.capability.v1beta1.CapabilityOwners)
    - [Owner](#cosmos.capability.v1beta1.Owner)
  
- [cosmos/capability/v1beta1/genesis.proto](#cosmos/capability/v1beta1/genesis.proto)
    - [GenesisOwners](#cosmos.capability.v1beta1.GenesisOwners)
    - [GenesisState](#cosmos.capability.v1beta1.GenesisState)
  
- [cosmos/crisis/v1beta1/genesis.proto](#cosmos/crisis/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.crisis.v1beta1.GenesisState)
  
- [cosmos/crisis/v1beta1/tx.proto](#cosmos/crisis/v1beta1/tx.proto)
    - [MsgVerifyInvariant](#cosmos.crisis.v1beta1.MsgVerifyInvariant)
    - [MsgVerifyInvariantResponse](#cosmos.crisis.v1beta1.MsgVerifyInvariantResponse)
  
    - [Msg](#cosmos.crisis.v1beta1.Msg)
  
- [cosmos/crypto/ed25519/keys.proto](#cosmos/crypto/ed25519/keys.proto)
    - [PrivKey](#cosmos.crypto.ed25519.PrivKey)
    - [PubKey](#cosmos.crypto.ed25519.PubKey)
  
- [cosmos/crypto/hd/v1/hd.proto](#cosmos/crypto/hd/v1/hd.proto)
    - [BIP44Params](#cosmos.crypto.hd.v1.BIP44Params)
  
- [cosmos/crypto/keyring/v1/record.proto](#cosmos/crypto/keyring/v1/record.proto)
    - [Record](#cosmos.crypto.keyring.v1.Record)
    - [Record.Ledger](#cosmos.crypto.keyring.v1.Record.Ledger)
    - [Record.Local](#cosmos.crypto.keyring.v1.Record.Local)
    - [Record.Multi](#cosmos.crypto.keyring.v1.Record.Multi)
    - [Record.Offline](#cosmos.crypto.keyring.v1.Record.Offline)
  
- [cosmos/crypto/multisig/keys.proto](#cosmos/crypto/multisig/keys.proto)
    - [LegacyAminoPubKey](#cosmos.crypto.multisig.LegacyAminoPubKey)
  
- [cosmos/crypto/multisig/v1beta1/multisig.proto](#cosmos/crypto/multisig/v1beta1/multisig.proto)
    - [CompactBitArray](#cosmos.crypto.multisig.v1beta1.CompactBitArray)
    - [MultiSignature](#cosmos.crypto.multisig.v1beta1.MultiSignature)
  
- [cosmos/crypto/secp256k1/keys.proto](#cosmos/crypto/secp256k1/keys.proto)
    - [PrivKey](#cosmos.crypto.secp256k1.PrivKey)
    - [PubKey](#cosmos.crypto.secp256k1.PubKey)
  
- [cosmos/crypto/secp256r1/keys.proto](#cosmos/crypto/secp256r1/keys.proto)
    - [PrivKey](#cosmos.crypto.secp256r1.PrivKey)
    - [PubKey](#cosmos.crypto.secp256r1.PubKey)
  
- [cosmos/distribution/v1beta1/distribution.proto](#cosmos/distribution/v1beta1/distribution.proto)
    - [CommunityPoolSpendProposal](#cosmos.distribution.v1beta1.CommunityPoolSpendProposal)
    - [CommunityPoolSpendProposalWithDeposit](#cosmos.distribution.v1beta1.CommunityPoolSpendProposalWithDeposit)
    - [DelegationDelegatorReward](#cosmos.distribution.v1beta1.DelegationDelegatorReward)
    - [DelegatorStartingInfo](#cosmos.distribution.v1beta1.DelegatorStartingInfo)
    - [FeePool](#cosmos.distribution.v1beta1.FeePool)
    - [Params](#cosmos.distribution.v1beta1.Params)
    - [ValidatorAccumulatedCommission](#cosmos.distribution.v1beta1.ValidatorAccumulatedCommission)
    - [ValidatorCurrentRewards](#cosmos.distribution.v1beta1.ValidatorCurrentRewards)
    - [ValidatorHistoricalRewards](#cosmos.distribution.v1beta1.ValidatorHistoricalRewards)
    - [ValidatorOutstandingRewards](#cosmos.distribution.v1beta1.ValidatorOutstandingRewards)
    - [ValidatorSlashEvent](#cosmos.distribution.v1beta1.ValidatorSlashEvent)
    - [ValidatorSlashEvents](#cosmos.distribution.v1beta1.ValidatorSlashEvents)
  
- [cosmos/distribution/v1beta1/genesis.proto](#cosmos/distribution/v1beta1/genesis.proto)
    - [DelegatorStartingInfoRecord](#cosmos.distribution.v1beta1.DelegatorStartingInfoRecord)
    - [DelegatorWithdrawInfo](#cosmos.distribution.v1beta1.DelegatorWithdrawInfo)
    - [GenesisState](#cosmos.distribution.v1beta1.GenesisState)
    - [ValidatorAccumulatedCommissionRecord](#cosmos.distribution.v1beta1.ValidatorAccumulatedCommissionRecord)
    - [ValidatorCurrentRewardsRecord](#cosmos.distribution.v1beta1.ValidatorCurrentRewardsRecord)
    - [ValidatorHistoricalRewardsRecord](#cosmos.distribution.v1beta1.ValidatorHistoricalRewardsRecord)
    - [ValidatorOutstandingRewardsRecord](#cosmos.distribution.v1beta1.ValidatorOutstandingRewardsRecord)
    - [ValidatorSlashEventRecord](#cosmos.distribution.v1beta1.ValidatorSlashEventRecord)
  
- [cosmos/distribution/v1beta1/query.proto](#cosmos/distribution/v1beta1/query.proto)
    - [QueryCommunityPoolRequest](#cosmos.distribution.v1beta1.QueryCommunityPoolRequest)
    - [QueryCommunityPoolResponse](#cosmos.distribution.v1beta1.QueryCommunityPoolResponse)
    - [QueryDelegationRewardsRequest](#cosmos.distribution.v1beta1.QueryDelegationRewardsRequest)
    - [QueryDelegationRewardsResponse](#cosmos.distribution.v1beta1.QueryDelegationRewardsResponse)
    - [QueryDelegationTotalRewardsRequest](#cosmos.distribution.v1beta1.QueryDelegationTotalRewardsRequest)
    - [QueryDelegationTotalRewardsResponse](#cosmos.distribution.v1beta1.QueryDelegationTotalRewardsResponse)
    - [QueryDelegatorValidatorsRequest](#cosmos.distribution.v1beta1.QueryDelegatorValidatorsRequest)
    - [QueryDelegatorValidatorsResponse](#cosmos.distribution.v1beta1.QueryDelegatorValidatorsResponse)
    - [QueryDelegatorWithdrawAddressRequest](#cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressRequest)
    - [QueryDelegatorWithdrawAddressResponse](#cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressResponse)
    - [QueryParamsRequest](#cosmos.distribution.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.distribution.v1beta1.QueryParamsResponse)
    - [QueryValidatorCommissionRequest](#cosmos.distribution.v1beta1.QueryValidatorCommissionRequest)
    - [QueryValidatorCommissionResponse](#cosmos.distribution.v1beta1.QueryValidatorCommissionResponse)
    - [QueryValidatorOutstandingRewardsRequest](#cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsRequest)
    - [QueryValidatorOutstandingRewardsResponse](#cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsResponse)
    - [QueryValidatorSlashesRequest](#cosmos.distribution.v1beta1.QueryValidatorSlashesRequest)
    - [QueryValidatorSlashesResponse](#cosmos.distribution.v1beta1.QueryValidatorSlashesResponse)
  
    - [Query](#cosmos.distribution.v1beta1.Query)
  
- [cosmos/distribution/v1beta1/tx.proto](#cosmos/distribution/v1beta1/tx.proto)
    - [MsgFundCommunityPool](#cosmos.distribution.v1beta1.MsgFundCommunityPool)
    - [MsgFundCommunityPoolResponse](#cosmos.distribution.v1beta1.MsgFundCommunityPoolResponse)
    - [MsgSetWithdrawAddress](#cosmos.distribution.v1beta1.MsgSetWithdrawAddress)
    - [MsgSetWithdrawAddressResponse](#cosmos.distribution.v1beta1.MsgSetWithdrawAddressResponse)
    - [MsgWithdrawDelegatorReward](#cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward)
    - [MsgWithdrawDelegatorRewardResponse](#cosmos.distribution.v1beta1.MsgWithdrawDelegatorRewardResponse)
    - [MsgWithdrawValidatorCommission](#cosmos.distribution.v1beta1.MsgWithdrawValidatorCommission)
    - [MsgWithdrawValidatorCommissionResponse](#cosmos.distribution.v1beta1.MsgWithdrawValidatorCommissionResponse)
  
    - [Msg](#cosmos.distribution.v1beta1.Msg)
  
- [cosmos/evidence/v1beta1/evidence.proto](#cosmos/evidence/v1beta1/evidence.proto)
    - [Equivocation](#cosmos.evidence.v1beta1.Equivocation)
  
- [cosmos/evidence/v1beta1/genesis.proto](#cosmos/evidence/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.evidence.v1beta1.GenesisState)
  
- [cosmos/evidence/v1beta1/query.proto](#cosmos/evidence/v1beta1/query.proto)
    - [QueryAllEvidenceRequest](#cosmos.evidence.v1beta1.QueryAllEvidenceRequest)
    - [QueryAllEvidenceResponse](#cosmos.evidence.v1beta1.QueryAllEvidenceResponse)
    - [QueryEvidenceRequest](#cosmos.evidence.v1beta1.QueryEvidenceRequest)
    - [QueryEvidenceResponse](#cosmos.evidence.v1beta1.QueryEvidenceResponse)
  
    - [Query](#cosmos.evidence.v1beta1.Query)
  
- [cosmos/evidence/v1beta1/tx.proto](#cosmos/evidence/v1beta1/tx.proto)
    - [MsgSubmitEvidence](#cosmos.evidence.v1beta1.MsgSubmitEvidence)
    - [MsgSubmitEvidenceResponse](#cosmos.evidence.v1beta1.MsgSubmitEvidenceResponse)
  
    - [Msg](#cosmos.evidence.v1beta1.Msg)
  
- [cosmos/feegrant/v1beta1/feegrant.proto](#cosmos/feegrant/v1beta1/feegrant.proto)
    - [AllowedMsgAllowance](#cosmos.feegrant.v1beta1.AllowedMsgAllowance)
    - [BasicAllowance](#cosmos.feegrant.v1beta1.BasicAllowance)
    - [Grant](#cosmos.feegrant.v1beta1.Grant)
    - [PeriodicAllowance](#cosmos.feegrant.v1beta1.PeriodicAllowance)
  
- [cosmos/feegrant/v1beta1/genesis.proto](#cosmos/feegrant/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.feegrant.v1beta1.GenesisState)
  
- [cosmos/feegrant/v1beta1/query.proto](#cosmos/feegrant/v1beta1/query.proto)
    - [QueryAllowanceRequest](#cosmos.feegrant.v1beta1.QueryAllowanceRequest)
    - [QueryAllowanceResponse](#cosmos.feegrant.v1beta1.QueryAllowanceResponse)
    - [QueryAllowancesRequest](#cosmos.feegrant.v1beta1.QueryAllowancesRequest)
    - [QueryAllowancesResponse](#cosmos.feegrant.v1beta1.QueryAllowancesResponse)
  
    - [Query](#cosmos.feegrant.v1beta1.Query)
  
- [cosmos/feegrant/v1beta1/tx.proto](#cosmos/feegrant/v1beta1/tx.proto)
    - [MsgGrantAllowance](#cosmos.feegrant.v1beta1.MsgGrantAllowance)
    - [MsgGrantAllowanceResponse](#cosmos.feegrant.v1beta1.MsgGrantAllowanceResponse)
    - [MsgRevokeAllowance](#cosmos.feegrant.v1beta1.MsgRevokeAllowance)
    - [MsgRevokeAllowanceResponse](#cosmos.feegrant.v1beta1.MsgRevokeAllowanceResponse)
  
    - [Msg](#cosmos.feegrant.v1beta1.Msg)
  
- [cosmos/genutil/v1beta1/genesis.proto](#cosmos/genutil/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.genutil.v1beta1.GenesisState)
  
- [cosmos/gov/v1beta1/gov.proto](#cosmos/gov/v1beta1/gov.proto)
    - [Deposit](#cosmos.gov.v1beta1.Deposit)
    - [DepositParams](#cosmos.gov.v1beta1.DepositParams)
    - [Proposal](#cosmos.gov.v1beta1.Proposal)
    - [TallyParams](#cosmos.gov.v1beta1.TallyParams)
    - [TallyResult](#cosmos.gov.v1beta1.TallyResult)
    - [TextProposal](#cosmos.gov.v1beta1.TextProposal)
    - [Vote](#cosmos.gov.v1beta1.Vote)
    - [VotingParams](#cosmos.gov.v1beta1.VotingParams)
    - [WeightedVoteOption](#cosmos.gov.v1beta1.WeightedVoteOption)
  
    - [ProposalStatus](#cosmos.gov.v1beta1.ProposalStatus)
    - [VoteOption](#cosmos.gov.v1beta1.VoteOption)
  
- [cosmos/gov/v1beta1/genesis.proto](#cosmos/gov/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.gov.v1beta1.GenesisState)
  
- [cosmos/gov/v1beta1/query.proto](#cosmos/gov/v1beta1/query.proto)
    - [QueryDepositRequest](#cosmos.gov.v1beta1.QueryDepositRequest)
    - [QueryDepositResponse](#cosmos.gov.v1beta1.QueryDepositResponse)
    - [QueryDepositsRequest](#cosmos.gov.v1beta1.QueryDepositsRequest)
    - [QueryDepositsResponse](#cosmos.gov.v1beta1.QueryDepositsResponse)
    - [QueryParamsRequest](#cosmos.gov.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.gov.v1beta1.QueryParamsResponse)
    - [QueryProposalRequest](#cosmos.gov.v1beta1.QueryProposalRequest)
    - [QueryProposalResponse](#cosmos.gov.v1beta1.QueryProposalResponse)
    - [QueryProposalsRequest](#cosmos.gov.v1beta1.QueryProposalsRequest)
    - [QueryProposalsResponse](#cosmos.gov.v1beta1.QueryProposalsResponse)
    - [QueryTallyResultRequest](#cosmos.gov.v1beta1.QueryTallyResultRequest)
    - [QueryTallyResultResponse](#cosmos.gov.v1beta1.QueryTallyResultResponse)
    - [QueryVoteRequest](#cosmos.gov.v1beta1.QueryVoteRequest)
    - [QueryVoteResponse](#cosmos.gov.v1beta1.QueryVoteResponse)
    - [QueryVotesRequest](#cosmos.gov.v1beta1.QueryVotesRequest)
    - [QueryVotesResponse](#cosmos.gov.v1beta1.QueryVotesResponse)
  
    - [Query](#cosmos.gov.v1beta1.Query)
  
- [cosmos/gov/v1beta1/tx.proto](#cosmos/gov/v1beta1/tx.proto)
    - [MsgDeposit](#cosmos.gov.v1beta1.MsgDeposit)
    - [MsgDepositResponse](#cosmos.gov.v1beta1.MsgDepositResponse)
    - [MsgSubmitProposal](#cosmos.gov.v1beta1.MsgSubmitProposal)
    - [MsgSubmitProposalResponse](#cosmos.gov.v1beta1.MsgSubmitProposalResponse)
    - [MsgVote](#cosmos.gov.v1beta1.MsgVote)
    - [MsgVoteResponse](#cosmos.gov.v1beta1.MsgVoteResponse)
    - [MsgVoteWeighted](#cosmos.gov.v1beta1.MsgVoteWeighted)
    - [MsgVoteWeightedResponse](#cosmos.gov.v1beta1.MsgVoteWeightedResponse)
  
    - [Msg](#cosmos.gov.v1beta1.Msg)
  
- [cosmos/gov/v1beta2/gov.proto](#cosmos/gov/v1beta2/gov.proto)
    - [Deposit](#cosmos.gov.v1beta2.Deposit)
    - [DepositParams](#cosmos.gov.v1beta2.DepositParams)
    - [Proposal](#cosmos.gov.v1beta2.Proposal)
    - [TallyParams](#cosmos.gov.v1beta2.TallyParams)
    - [TallyResult](#cosmos.gov.v1beta2.TallyResult)
    - [Vote](#cosmos.gov.v1beta2.Vote)
    - [VotingParams](#cosmos.gov.v1beta2.VotingParams)
    - [WeightedVoteOption](#cosmos.gov.v1beta2.WeightedVoteOption)
  
    - [ProposalStatus](#cosmos.gov.v1beta2.ProposalStatus)
    - [VoteOption](#cosmos.gov.v1beta2.VoteOption)
  
- [cosmos/gov/v1beta2/genesis.proto](#cosmos/gov/v1beta2/genesis.proto)
    - [GenesisState](#cosmos.gov.v1beta2.GenesisState)
  
- [cosmos/gov/v1beta2/query.proto](#cosmos/gov/v1beta2/query.proto)
    - [QueryDepositRequest](#cosmos.gov.v1beta2.QueryDepositRequest)
    - [QueryDepositResponse](#cosmos.gov.v1beta2.QueryDepositResponse)
    - [QueryDepositsRequest](#cosmos.gov.v1beta2.QueryDepositsRequest)
    - [QueryDepositsResponse](#cosmos.gov.v1beta2.QueryDepositsResponse)
    - [QueryParamsRequest](#cosmos.gov.v1beta2.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.gov.v1beta2.QueryParamsResponse)
    - [QueryProposalRequest](#cosmos.gov.v1beta2.QueryProposalRequest)
    - [QueryProposalResponse](#cosmos.gov.v1beta2.QueryProposalResponse)
    - [QueryProposalsRequest](#cosmos.gov.v1beta2.QueryProposalsRequest)
    - [QueryProposalsResponse](#cosmos.gov.v1beta2.QueryProposalsResponse)
    - [QueryTallyResultRequest](#cosmos.gov.v1beta2.QueryTallyResultRequest)
    - [QueryTallyResultResponse](#cosmos.gov.v1beta2.QueryTallyResultResponse)
    - [QueryVoteRequest](#cosmos.gov.v1beta2.QueryVoteRequest)
    - [QueryVoteResponse](#cosmos.gov.v1beta2.QueryVoteResponse)
    - [QueryVotesRequest](#cosmos.gov.v1beta2.QueryVotesRequest)
    - [QueryVotesResponse](#cosmos.gov.v1beta2.QueryVotesResponse)
  
    - [Query](#cosmos.gov.v1beta2.Query)
  
- [cosmos/gov/v1beta2/tx.proto](#cosmos/gov/v1beta2/tx.proto)
    - [MsgDeposit](#cosmos.gov.v1beta2.MsgDeposit)
    - [MsgDepositResponse](#cosmos.gov.v1beta2.MsgDepositResponse)
    - [MsgSubmitProposal](#cosmos.gov.v1beta2.MsgSubmitProposal)
    - [MsgSubmitProposalResponse](#cosmos.gov.v1beta2.MsgSubmitProposalResponse)
    - [MsgVote](#cosmos.gov.v1beta2.MsgVote)
    - [MsgVoteResponse](#cosmos.gov.v1beta2.MsgVoteResponse)
    - [MsgVoteWeighted](#cosmos.gov.v1beta2.MsgVoteWeighted)
    - [MsgVoteWeightedResponse](#cosmos.gov.v1beta2.MsgVoteWeightedResponse)
  
    - [Msg](#cosmos.gov.v1beta2.Msg)
  
- [cosmos/group/v1beta1/events.proto](#cosmos/group/v1beta1/events.proto)
    - [EventCreateGroup](#cosmos.group.v1beta1.EventCreateGroup)
    - [EventCreateGroupAccount](#cosmos.group.v1beta1.EventCreateGroupAccount)
    - [EventCreateProposal](#cosmos.group.v1beta1.EventCreateProposal)
    - [EventExec](#cosmos.group.v1beta1.EventExec)
    - [EventUpdateGroup](#cosmos.group.v1beta1.EventUpdateGroup)
    - [EventUpdateGroupAccount](#cosmos.group.v1beta1.EventUpdateGroupAccount)
    - [EventVote](#cosmos.group.v1beta1.EventVote)
  
- [cosmos/group/v1beta1/types.proto](#cosmos/group/v1beta1/types.proto)
    - [GroupAccountInfo](#cosmos.group.v1beta1.GroupAccountInfo)
    - [GroupInfo](#cosmos.group.v1beta1.GroupInfo)
    - [GroupMember](#cosmos.group.v1beta1.GroupMember)
    - [Member](#cosmos.group.v1beta1.Member)
    - [Members](#cosmos.group.v1beta1.Members)
    - [Proposal](#cosmos.group.v1beta1.Proposal)
    - [Tally](#cosmos.group.v1beta1.Tally)
    - [ThresholdDecisionPolicy](#cosmos.group.v1beta1.ThresholdDecisionPolicy)
    - [Vote](#cosmos.group.v1beta1.Vote)
  
    - [Choice](#cosmos.group.v1beta1.Choice)
    - [Proposal.ExecutorResult](#cosmos.group.v1beta1.Proposal.ExecutorResult)
    - [Proposal.Result](#cosmos.group.v1beta1.Proposal.Result)
    - [Proposal.Status](#cosmos.group.v1beta1.Proposal.Status)
  
- [cosmos/group/v1beta1/query.proto](#cosmos/group/v1beta1/query.proto)
    - [QueryGroupAccountInfoRequest](#cosmos.group.v1beta1.QueryGroupAccountInfoRequest)
    - [QueryGroupAccountInfoResponse](#cosmos.group.v1beta1.QueryGroupAccountInfoResponse)
    - [QueryGroupAccountsByAdminRequest](#cosmos.group.v1beta1.QueryGroupAccountsByAdminRequest)
    - [QueryGroupAccountsByAdminResponse](#cosmos.group.v1beta1.QueryGroupAccountsByAdminResponse)
    - [QueryGroupAccountsByGroupRequest](#cosmos.group.v1beta1.QueryGroupAccountsByGroupRequest)
    - [QueryGroupAccountsByGroupResponse](#cosmos.group.v1beta1.QueryGroupAccountsByGroupResponse)
    - [QueryGroupInfoRequest](#cosmos.group.v1beta1.QueryGroupInfoRequest)
    - [QueryGroupInfoResponse](#cosmos.group.v1beta1.QueryGroupInfoResponse)
    - [QueryGroupMembersRequest](#cosmos.group.v1beta1.QueryGroupMembersRequest)
    - [QueryGroupMembersResponse](#cosmos.group.v1beta1.QueryGroupMembersResponse)
    - [QueryGroupsByAdminRequest](#cosmos.group.v1beta1.QueryGroupsByAdminRequest)
    - [QueryGroupsByAdminResponse](#cosmos.group.v1beta1.QueryGroupsByAdminResponse)
    - [QueryProposalRequest](#cosmos.group.v1beta1.QueryProposalRequest)
    - [QueryProposalResponse](#cosmos.group.v1beta1.QueryProposalResponse)
    - [QueryProposalsByGroupAccountRequest](#cosmos.group.v1beta1.QueryProposalsByGroupAccountRequest)
    - [QueryProposalsByGroupAccountResponse](#cosmos.group.v1beta1.QueryProposalsByGroupAccountResponse)
    - [QueryVoteByProposalVoterRequest](#cosmos.group.v1beta1.QueryVoteByProposalVoterRequest)
    - [QueryVoteByProposalVoterResponse](#cosmos.group.v1beta1.QueryVoteByProposalVoterResponse)
    - [QueryVotesByProposalRequest](#cosmos.group.v1beta1.QueryVotesByProposalRequest)
    - [QueryVotesByProposalResponse](#cosmos.group.v1beta1.QueryVotesByProposalResponse)
    - [QueryVotesByVoterRequest](#cosmos.group.v1beta1.QueryVotesByVoterRequest)
    - [QueryVotesByVoterResponse](#cosmos.group.v1beta1.QueryVotesByVoterResponse)
  
    - [Query](#cosmos.group.v1beta1.Query)
  
- [cosmos/group/v1beta1/tx.proto](#cosmos/group/v1beta1/tx.proto)
    - [MsgCreateGroup](#cosmos.group.v1beta1.MsgCreateGroup)
    - [MsgCreateGroupAccount](#cosmos.group.v1beta1.MsgCreateGroupAccount)
    - [MsgCreateGroupAccountResponse](#cosmos.group.v1beta1.MsgCreateGroupAccountResponse)
    - [MsgCreateGroupResponse](#cosmos.group.v1beta1.MsgCreateGroupResponse)
    - [MsgCreateProposal](#cosmos.group.v1beta1.MsgCreateProposal)
    - [MsgCreateProposalResponse](#cosmos.group.v1beta1.MsgCreateProposalResponse)
    - [MsgExec](#cosmos.group.v1beta1.MsgExec)
    - [MsgExecResponse](#cosmos.group.v1beta1.MsgExecResponse)
    - [MsgUpdateGroupAccountAdmin](#cosmos.group.v1beta1.MsgUpdateGroupAccountAdmin)
    - [MsgUpdateGroupAccountAdminResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountAdminResponse)
    - [MsgUpdateGroupAccountDecisionPolicy](#cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicy)
    - [MsgUpdateGroupAccountDecisionPolicyResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicyResponse)
    - [MsgUpdateGroupAccountMetadata](#cosmos.group.v1beta1.MsgUpdateGroupAccountMetadata)
    - [MsgUpdateGroupAccountMetadataResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountMetadataResponse)
    - [MsgUpdateGroupAdmin](#cosmos.group.v1beta1.MsgUpdateGroupAdmin)
    - [MsgUpdateGroupAdminResponse](#cosmos.group.v1beta1.MsgUpdateGroupAdminResponse)
    - [MsgUpdateGroupMembers](#cosmos.group.v1beta1.MsgUpdateGroupMembers)
    - [MsgUpdateGroupMembersResponse](#cosmos.group.v1beta1.MsgUpdateGroupMembersResponse)
    - [MsgUpdateGroupMetadata](#cosmos.group.v1beta1.MsgUpdateGroupMetadata)
    - [MsgUpdateGroupMetadataResponse](#cosmos.group.v1beta1.MsgUpdateGroupMetadataResponse)
    - [MsgVote](#cosmos.group.v1beta1.MsgVote)
    - [MsgVoteResponse](#cosmos.group.v1beta1.MsgVoteResponse)
  
    - [Exec](#cosmos.group.v1beta1.Exec)
  
    - [Msg](#cosmos.group.v1beta1.Msg)
  
- [cosmos/mint/v1beta1/mint.proto](#cosmos/mint/v1beta1/mint.proto)
    - [Minter](#cosmos.mint.v1beta1.Minter)
    - [Params](#cosmos.mint.v1beta1.Params)
  
- [cosmos/mint/v1beta1/genesis.proto](#cosmos/mint/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.mint.v1beta1.GenesisState)
  
- [cosmos/mint/v1beta1/query.proto](#cosmos/mint/v1beta1/query.proto)
    - [QueryAnnualProvisionsRequest](#cosmos.mint.v1beta1.QueryAnnualProvisionsRequest)
    - [QueryAnnualProvisionsResponse](#cosmos.mint.v1beta1.QueryAnnualProvisionsResponse)
    - [QueryInflationRequest](#cosmos.mint.v1beta1.QueryInflationRequest)
    - [QueryInflationResponse](#cosmos.mint.v1beta1.QueryInflationResponse)
    - [QueryParamsRequest](#cosmos.mint.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.mint.v1beta1.QueryParamsResponse)
  
    - [Query](#cosmos.mint.v1beta1.Query)
  
- [cosmos/nft/v1beta1/event.proto](#cosmos/nft/v1beta1/event.proto)
    - [EventBurn](#cosmos.nft.v1beta1.EventBurn)
    - [EventMint](#cosmos.nft.v1beta1.EventMint)
    - [EventSend](#cosmos.nft.v1beta1.EventSend)
  
- [cosmos/nft/v1beta1/nft.proto](#cosmos/nft/v1beta1/nft.proto)
    - [Class](#cosmos.nft.v1beta1.Class)
    - [NFT](#cosmos.nft.v1beta1.NFT)
  
- [cosmos/nft/v1beta1/genesis.proto](#cosmos/nft/v1beta1/genesis.proto)
    - [Entry](#cosmos.nft.v1beta1.Entry)
    - [GenesisState](#cosmos.nft.v1beta1.GenesisState)
  
- [cosmos/nft/v1beta1/query.proto](#cosmos/nft/v1beta1/query.proto)
    - [QueryBalanceRequest](#cosmos.nft.v1beta1.QueryBalanceRequest)
    - [QueryBalanceResponse](#cosmos.nft.v1beta1.QueryBalanceResponse)
    - [QueryClassRequest](#cosmos.nft.v1beta1.QueryClassRequest)
    - [QueryClassResponse](#cosmos.nft.v1beta1.QueryClassResponse)
    - [QueryClassesRequest](#cosmos.nft.v1beta1.QueryClassesRequest)
    - [QueryClassesResponse](#cosmos.nft.v1beta1.QueryClassesResponse)
    - [QueryNFTRequest](#cosmos.nft.v1beta1.QueryNFTRequest)
    - [QueryNFTResponse](#cosmos.nft.v1beta1.QueryNFTResponse)
    - [QueryNFTsOfClassRequest](#cosmos.nft.v1beta1.QueryNFTsOfClassRequest)
    - [QueryNFTsOfClassResponse](#cosmos.nft.v1beta1.QueryNFTsOfClassResponse)
    - [QueryOwnerRequest](#cosmos.nft.v1beta1.QueryOwnerRequest)
    - [QueryOwnerResponse](#cosmos.nft.v1beta1.QueryOwnerResponse)
    - [QuerySupplyRequest](#cosmos.nft.v1beta1.QuerySupplyRequest)
    - [QuerySupplyResponse](#cosmos.nft.v1beta1.QuerySupplyResponse)
  
    - [Query](#cosmos.nft.v1beta1.Query)
  
- [cosmos/nft/v1beta1/tx.proto](#cosmos/nft/v1beta1/tx.proto)
    - [MsgSend](#cosmos.nft.v1beta1.MsgSend)
    - [MsgSendResponse](#cosmos.nft.v1beta1.MsgSendResponse)
  
    - [Msg](#cosmos.nft.v1beta1.Msg)
  
- [cosmos/params/v1beta1/params.proto](#cosmos/params/v1beta1/params.proto)
    - [ParamChange](#cosmos.params.v1beta1.ParamChange)
    - [ParameterChangeProposal](#cosmos.params.v1beta1.ParameterChangeProposal)
  
- [cosmos/params/v1beta1/query.proto](#cosmos/params/v1beta1/query.proto)
    - [QueryParamsRequest](#cosmos.params.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.params.v1beta1.QueryParamsResponse)
    - [QuerySubspacesRequest](#cosmos.params.v1beta1.QuerySubspacesRequest)
    - [QuerySubspacesResponse](#cosmos.params.v1beta1.QuerySubspacesResponse)
    - [Subspace](#cosmos.params.v1beta1.Subspace)
  
    - [Query](#cosmos.params.v1beta1.Query)
  
- [cosmos/slashing/v1beta1/slashing.proto](#cosmos/slashing/v1beta1/slashing.proto)
    - [Params](#cosmos.slashing.v1beta1.Params)
    - [ValidatorSigningInfo](#cosmos.slashing.v1beta1.ValidatorSigningInfo)
  
- [cosmos/slashing/v1beta1/genesis.proto](#cosmos/slashing/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.slashing.v1beta1.GenesisState)
    - [MissedBlock](#cosmos.slashing.v1beta1.MissedBlock)
    - [SigningInfo](#cosmos.slashing.v1beta1.SigningInfo)
    - [ValidatorMissedBlocks](#cosmos.slashing.v1beta1.ValidatorMissedBlocks)
  
- [cosmos/slashing/v1beta1/query.proto](#cosmos/slashing/v1beta1/query.proto)
    - [QueryParamsRequest](#cosmos.slashing.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.slashing.v1beta1.QueryParamsResponse)
    - [QuerySigningInfoRequest](#cosmos.slashing.v1beta1.QuerySigningInfoRequest)
    - [QuerySigningInfoResponse](#cosmos.slashing.v1beta1.QuerySigningInfoResponse)
    - [QuerySigningInfosRequest](#cosmos.slashing.v1beta1.QuerySigningInfosRequest)
    - [QuerySigningInfosResponse](#cosmos.slashing.v1beta1.QuerySigningInfosResponse)
  
    - [Query](#cosmos.slashing.v1beta1.Query)
  
- [cosmos/slashing/v1beta1/tx.proto](#cosmos/slashing/v1beta1/tx.proto)
    - [MsgUnjail](#cosmos.slashing.v1beta1.MsgUnjail)
    - [MsgUnjailResponse](#cosmos.slashing.v1beta1.MsgUnjailResponse)
  
    - [Msg](#cosmos.slashing.v1beta1.Msg)
  
- [cosmos/staking/v1beta1/authz.proto](#cosmos/staking/v1beta1/authz.proto)
    - [StakeAuthorization](#cosmos.staking.v1beta1.StakeAuthorization)
    - [StakeAuthorization.Validators](#cosmos.staking.v1beta1.StakeAuthorization.Validators)
  
    - [AuthorizationType](#cosmos.staking.v1beta1.AuthorizationType)
  
- [cosmos/staking/v1beta1/staking.proto](#cosmos/staking/v1beta1/staking.proto)
    - [Commission](#cosmos.staking.v1beta1.Commission)
    - [CommissionRates](#cosmos.staking.v1beta1.CommissionRates)
    - [DVPair](#cosmos.staking.v1beta1.DVPair)
    - [DVPairs](#cosmos.staking.v1beta1.DVPairs)
    - [DVVTriplet](#cosmos.staking.v1beta1.DVVTriplet)
    - [DVVTriplets](#cosmos.staking.v1beta1.DVVTriplets)
    - [Delegation](#cosmos.staking.v1beta1.Delegation)
    - [DelegationResponse](#cosmos.staking.v1beta1.DelegationResponse)
    - [Description](#cosmos.staking.v1beta1.Description)
    - [HistoricalInfo](#cosmos.staking.v1beta1.HistoricalInfo)
    - [Params](#cosmos.staking.v1beta1.Params)
    - [Pool](#cosmos.staking.v1beta1.Pool)
    - [Redelegation](#cosmos.staking.v1beta1.Redelegation)
    - [RedelegationEntry](#cosmos.staking.v1beta1.RedelegationEntry)
    - [RedelegationEntryResponse](#cosmos.staking.v1beta1.RedelegationEntryResponse)
    - [RedelegationResponse](#cosmos.staking.v1beta1.RedelegationResponse)
    - [UnbondingDelegation](#cosmos.staking.v1beta1.UnbondingDelegation)
    - [UnbondingDelegationEntry](#cosmos.staking.v1beta1.UnbondingDelegationEntry)
    - [ValAddresses](#cosmos.staking.v1beta1.ValAddresses)
    - [Validator](#cosmos.staking.v1beta1.Validator)
  
    - [BondStatus](#cosmos.staking.v1beta1.BondStatus)
  
- [cosmos/staking/v1beta1/genesis.proto](#cosmos/staking/v1beta1/genesis.proto)
    - [GenesisState](#cosmos.staking.v1beta1.GenesisState)
    - [LastValidatorPower](#cosmos.staking.v1beta1.LastValidatorPower)
  
- [cosmos/staking/v1beta1/query.proto](#cosmos/staking/v1beta1/query.proto)
    - [QueryDelegationRequest](#cosmos.staking.v1beta1.QueryDelegationRequest)
    - [QueryDelegationResponse](#cosmos.staking.v1beta1.QueryDelegationResponse)
    - [QueryDelegatorDelegationsRequest](#cosmos.staking.v1beta1.QueryDelegatorDelegationsRequest)
    - [QueryDelegatorDelegationsResponse](#cosmos.staking.v1beta1.QueryDelegatorDelegationsResponse)
    - [QueryDelegatorUnbondingDelegationsRequest](#cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsRequest)
    - [QueryDelegatorUnbondingDelegationsResponse](#cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsResponse)
    - [QueryDelegatorValidatorRequest](#cosmos.staking.v1beta1.QueryDelegatorValidatorRequest)
    - [QueryDelegatorValidatorResponse](#cosmos.staking.v1beta1.QueryDelegatorValidatorResponse)
    - [QueryDelegatorValidatorsRequest](#cosmos.staking.v1beta1.QueryDelegatorValidatorsRequest)
    - [QueryDelegatorValidatorsResponse](#cosmos.staking.v1beta1.QueryDelegatorValidatorsResponse)
    - [QueryHistoricalInfoRequest](#cosmos.staking.v1beta1.QueryHistoricalInfoRequest)
    - [QueryHistoricalInfoResponse](#cosmos.staking.v1beta1.QueryHistoricalInfoResponse)
    - [QueryParamsRequest](#cosmos.staking.v1beta1.QueryParamsRequest)
    - [QueryParamsResponse](#cosmos.staking.v1beta1.QueryParamsResponse)
    - [QueryPoolRequest](#cosmos.staking.v1beta1.QueryPoolRequest)
    - [QueryPoolResponse](#cosmos.staking.v1beta1.QueryPoolResponse)
    - [QueryRedelegationsRequest](#cosmos.staking.v1beta1.QueryRedelegationsRequest)
    - [QueryRedelegationsResponse](#cosmos.staking.v1beta1.QueryRedelegationsResponse)
    - [QueryUnbondingDelegationRequest](#cosmos.staking.v1beta1.QueryUnbondingDelegationRequest)
    - [QueryUnbondingDelegationResponse](#cosmos.staking.v1beta1.QueryUnbondingDelegationResponse)
    - [QueryValidatorDelegationsRequest](#cosmos.staking.v1beta1.QueryValidatorDelegationsRequest)
    - [QueryValidatorDelegationsResponse](#cosmos.staking.v1beta1.QueryValidatorDelegationsResponse)
    - [QueryValidatorRequest](#cosmos.staking.v1beta1.QueryValidatorRequest)
    - [QueryValidatorResponse](#cosmos.staking.v1beta1.QueryValidatorResponse)
    - [QueryValidatorUnbondingDelegationsRequest](#cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsRequest)
    - [QueryValidatorUnbondingDelegationsResponse](#cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsResponse)
    - [QueryValidatorsRequest](#cosmos.staking.v1beta1.QueryValidatorsRequest)
    - [QueryValidatorsResponse](#cosmos.staking.v1beta1.QueryValidatorsResponse)
  
    - [Query](#cosmos.staking.v1beta1.Query)
  
- [cosmos/staking/v1beta1/tx.proto](#cosmos/staking/v1beta1/tx.proto)
    - [MsgBeginRedelegate](#cosmos.staking.v1beta1.MsgBeginRedelegate)
    - [MsgBeginRedelegateResponse](#cosmos.staking.v1beta1.MsgBeginRedelegateResponse)
    - [MsgCreateValidator](#cosmos.staking.v1beta1.MsgCreateValidator)
    - [MsgCreateValidatorResponse](#cosmos.staking.v1beta1.MsgCreateValidatorResponse)
    - [MsgDelegate](#cosmos.staking.v1beta1.MsgDelegate)
    - [MsgDelegateResponse](#cosmos.staking.v1beta1.MsgDelegateResponse)
    - [MsgEditValidator](#cosmos.staking.v1beta1.MsgEditValidator)
    - [MsgEditValidatorResponse](#cosmos.staking.v1beta1.MsgEditValidatorResponse)
    - [MsgUndelegate](#cosmos.staking.v1beta1.MsgUndelegate)
    - [MsgUndelegateResponse](#cosmos.staking.v1beta1.MsgUndelegateResponse)
  
    - [Msg](#cosmos.staking.v1beta1.Msg)
  
- [cosmos/tx/signing/v1beta1/signing.proto](#cosmos/tx/signing/v1beta1/signing.proto)
    - [SignatureDescriptor](#cosmos.tx.signing.v1beta1.SignatureDescriptor)
    - [SignatureDescriptor.Data](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data)
    - [SignatureDescriptor.Data.Multi](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Multi)
    - [SignatureDescriptor.Data.Single](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Single)
    - [SignatureDescriptors](#cosmos.tx.signing.v1beta1.SignatureDescriptors)
  
    - [SignMode](#cosmos.tx.signing.v1beta1.SignMode)
  
- [cosmos/tx/v1beta1/tx.proto](#cosmos/tx/v1beta1/tx.proto)
    - [AuthInfo](#cosmos.tx.v1beta1.AuthInfo)
    - [AuxSignerData](#cosmos.tx.v1beta1.AuxSignerData)
    - [Fee](#cosmos.tx.v1beta1.Fee)
    - [ModeInfo](#cosmos.tx.v1beta1.ModeInfo)
    - [ModeInfo.Multi](#cosmos.tx.v1beta1.ModeInfo.Multi)
    - [ModeInfo.Single](#cosmos.tx.v1beta1.ModeInfo.Single)
    - [SignDoc](#cosmos.tx.v1beta1.SignDoc)
    - [SignDocDirectAux](#cosmos.tx.v1beta1.SignDocDirectAux)
    - [SignerInfo](#cosmos.tx.v1beta1.SignerInfo)
    - [Tip](#cosmos.tx.v1beta1.Tip)
    - [Tx](#cosmos.tx.v1beta1.Tx)
    - [TxBody](#cosmos.tx.v1beta1.TxBody)
    - [TxRaw](#cosmos.tx.v1beta1.TxRaw)
  
- [cosmos/tx/v1beta1/service.proto](#cosmos/tx/v1beta1/service.proto)
    - [BroadcastTxRequest](#cosmos.tx.v1beta1.BroadcastTxRequest)
    - [BroadcastTxResponse](#cosmos.tx.v1beta1.BroadcastTxResponse)
    - [GetTxRequest](#cosmos.tx.v1beta1.GetTxRequest)
    - [GetTxResponse](#cosmos.tx.v1beta1.GetTxResponse)
    - [GetTxsEventRequest](#cosmos.tx.v1beta1.GetTxsEventRequest)
    - [GetTxsEventResponse](#cosmos.tx.v1beta1.GetTxsEventResponse)
    - [SimulateRequest](#cosmos.tx.v1beta1.SimulateRequest)
    - [SimulateResponse](#cosmos.tx.v1beta1.SimulateResponse)
  
    - [BroadcastMode](#cosmos.tx.v1beta1.BroadcastMode)
    - [OrderBy](#cosmos.tx.v1beta1.OrderBy)
  
    - [Service](#cosmos.tx.v1beta1.Service)
  
- [cosmos/upgrade/v1beta1/upgrade.proto](#cosmos/upgrade/v1beta1/upgrade.proto)
    - [CancelSoftwareUpgradeProposal](#cosmos.upgrade.v1beta1.CancelSoftwareUpgradeProposal)
    - [ModuleVersion](#cosmos.upgrade.v1beta1.ModuleVersion)
    - [Plan](#cosmos.upgrade.v1beta1.Plan)
    - [SoftwareUpgradeProposal](#cosmos.upgrade.v1beta1.SoftwareUpgradeProposal)
  
- [cosmos/upgrade/v1beta1/query.proto](#cosmos/upgrade/v1beta1/query.proto)
    - [QueryAppliedPlanRequest](#cosmos.upgrade.v1beta1.QueryAppliedPlanRequest)
    - [QueryAppliedPlanResponse](#cosmos.upgrade.v1beta1.QueryAppliedPlanResponse)
    - [QueryCurrentPlanRequest](#cosmos.upgrade.v1beta1.QueryCurrentPlanRequest)
    - [QueryCurrentPlanResponse](#cosmos.upgrade.v1beta1.QueryCurrentPlanResponse)
    - [QueryModuleVersionsRequest](#cosmos.upgrade.v1beta1.QueryModuleVersionsRequest)
    - [QueryModuleVersionsResponse](#cosmos.upgrade.v1beta1.QueryModuleVersionsResponse)
    - [QueryUpgradedConsensusStateRequest](#cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateRequest)
    - [QueryUpgradedConsensusStateResponse](#cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateResponse)
  
    - [Query](#cosmos.upgrade.v1beta1.Query)
  
- [cosmos/vesting/v1beta1/vesting.proto](#cosmos/vesting/v1beta1/vesting.proto)
    - [BaseVestingAccount](#cosmos.vesting.v1beta1.BaseVestingAccount)
    - [ContinuousVestingAccount](#cosmos.vesting.v1beta1.ContinuousVestingAccount)
    - [DelayedVestingAccount](#cosmos.vesting.v1beta1.DelayedVestingAccount)
    - [Period](#cosmos.vesting.v1beta1.Period)
    - [PeriodicVestingAccount](#cosmos.vesting.v1beta1.PeriodicVestingAccount)
    - [PermanentLockedAccount](#cosmos.vesting.v1beta1.PermanentLockedAccount)
  
- [cosmos/vesting/v1beta1/tx.proto](#cosmos/vesting/v1beta1/tx.proto)
    - [MsgCreatePeriodicVestingAccount](#cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccount)
    - [MsgCreatePeriodicVestingAccountResponse](#cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccountResponse)
    - [MsgCreateVestingAccount](#cosmos.vesting.v1beta1.MsgCreateVestingAccount)
    - [MsgCreateVestingAccountResponse](#cosmos.vesting.v1beta1.MsgCreateVestingAccountResponse)
  
    - [Msg](#cosmos.vesting.v1beta1.Msg)
  
- [Scalar Value Types](#scalar-value-types)



<a name="cosmos/auth/v1beta1/auth.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/auth/v1beta1/auth.proto



<a name="cosmos.auth.v1beta1.BaseAccount"></a>

### BaseAccount
BaseAccount 定义了基本帐户类型。 它包含所有必要的字段
用于基本帐户功能。 任何自定义帐户类型都应扩展此
附加功能的类型(例如归属)。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  |  |
| `pub_key` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `account_number` | [uint64](#uint64) |  |  |
| `sequence` | [uint64](#uint64) |  |  |






<a name="cosmos.auth.v1beta1.ModuleAccount"></a>

### ModuleAccount
ModuleAccount 为在池中持有硬币的模块定义了一个帐户。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_account` | [BaseAccount](#cosmos.auth.v1beta1.BaseAccount) |  |  |
| `name` | [string](#string) |  |  |
| `permissions` | [string](#string) | repeated |  |






<a name="cosmos.auth.v1beta1.Params"></a>

### Params
Params 定义了 auth 模块的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `max_memo_characters` | [uint64](#uint64) |  |  |
| `tx_sig_limit` | [uint64](#uint64) |  |  |
| `tx_size_cost_per_byte` | [uint64](#uint64) |  |  |
| `sig_verify_cost_ed25519` | [uint64](#uint64) |  |  |
| `sig_verify_cost_secp256k1` | [uint64](#uint64) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/auth/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/auth/v1beta1/genesis.proto



<a name="cosmos.auth.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 auth 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.auth.v1beta1.Params) |  | params defines all the paramaters of the module. |
| `accounts` | [google.protobuf.Any](#google.protobuf.Any) | repeated | accounts are the accounts present at genesis. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/query/v1beta1/pagination.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/query/v1beta1/pagination.proto



<a name="cosmos.base.query.v1beta1.PageRequest"></a>

### PageRequest
PageRequest 将嵌入到 gRPC 请求消息中以实现高效分页。
Ex:

 message SomeRequest {
         Foo some_parameter = 1;
         PageRequest pagination = 2;
 }


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  | key is a value returned in PageResponse.next_key to begin querying the next page most efficiently. Only one of offset or key should be set. |
| `offset` | [uint64](#uint64) |  | offset is a numeric offset that can be used when key is unavailable. It is less efficient than using key. Only one of offset or key should be set. |
| `limit` | [uint64](#uint64) |  | limit is the total number of results to be returned in the result page. If left empty it will default to a value to be set by each app. |
| `count_total` | [bool](#bool) |  | count_total is set to true to indicate that the result set should include a count of the total number of items available for pagination in UIs. count_total is only respected when offset is used. It is ignored when key is set. |
| `reverse` | [bool](#bool) |  | reverse is set to true if results are to be returned in the descending order.

Since: cosmos-sdk 0.43 |






<a name="cosmos.base.query.v1beta1.PageResponse"></a>

### PageResponse
PageResponse 将嵌入到 gRPC 响应消息中，其中相应的请求消息已使用 PageRequest。

 message SomeResponse {
         repeated Bar results = 1;
         PageResponse page = 2;
 }


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `next_key` | [bytes](#bytes) |  | next_key is the key to be passed to PageRequest.key to query the next page most efficiently |
| `total` | [uint64](#uint64) |  | total is total number of results available if PageRequest.count_total was set, its value is undefined otherwise |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/auth/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/auth/v1beta1/query.proto



<a name="cosmos.auth.v1beta1.AddressBytesToStringRequest"></a>

### AddressBytesToStringRequest
AddressBytesToStringRequest 是 AddressString rpc 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address_bytes` | [bytes](#bytes) |  |  |






<a name="cosmos.auth.v1beta1.AddressBytesToStringResponse"></a>

### AddressBytesToStringResponse
AddressBytesToStringResponse 是 AddressString rpc 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address_string` | [string](#string) |  |  |






<a name="cosmos.auth.v1beta1.AddressStringToBytesRequest"></a>

### AddressStringToBytesRequest
AddressStringToBytesRequest 是 AccountBytes rpc 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address_string` | [string](#string) |  |  |






<a name="cosmos.auth.v1beta1.AddressStringToBytesResponse"></a>

### AddressStringToBytesResponse
AddressStringToBytesResponse 是 AddressBytes rpc 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address_bytes` | [bytes](#bytes) |  |  |






<a name="cosmos.auth.v1beta1.Bech32PrefixRequest"></a>

### Bech32PrefixRequest
Bech32PrefixRequest 是 Bech32Prefix rpc 方法的请求类型






<a name="cosmos.auth.v1beta1.Bech32PrefixResponse"></a>

### Bech32PrefixResponse
Bech32PrefixResponse 是 Bech32Prefix rpc 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `bech32_prefix` | [string](#string) |  |  |






<a name="cosmos.auth.v1beta1.QueryAccountRequest"></a>

### QueryAccountRequest
QueryAccountRequest 是 Query/Account RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address defines the address to query for. |






<a name="cosmos.auth.v1beta1.QueryAccountResponse"></a>

### QueryAccountResponse
QueryAccountResponse 是 Query/Account RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `account` | [google.protobuf.Any](#google.protobuf.Any) |  | account defines the account of the corresponding address. |






<a name="cosmos.auth.v1beta1.QueryAccountsRequest"></a>

### QueryAccountsRequest
QueryAccountsRequest 是 Query/Accounts RPC 方法的请求类型。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.auth.v1beta1.QueryAccountsResponse"></a>

### QueryAccountsResponse
QueryAccountsResponse 是 Query/Accounts RPC 方法的响应类型。

自: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `accounts` | [google.protobuf.Any](#google.protobuf.Any) | repeated | accounts are the existing accounts |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.auth.v1beta1.QueryModuleAccountsRequest"></a>

### QueryModuleAccountsRequest
QueryModuleAccountsRequest 是 Query/ModuleAccounts RPC 方法的请求类型。






<a name="cosmos.auth.v1beta1.QueryModuleAccountsResponse"></a>

### QueryModuleAccountsResponse
QueryModuleAccountsResponse 是 Query/ModuleAccounts RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `accounts` | [google.protobuf.Any](#google.protobuf.Any) | repeated |  |






<a name="cosmos.auth.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。






<a name="cosmos.auth.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.auth.v1beta1.Params) |  | params defines the parameters of the module. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.auth.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Accounts` | [QueryAccountsRequest](#cosmos.auth.v1beta1.QueryAccountsRequest) | [QueryAccountsResponse](#cosmos.auth.v1beta1.QueryAccountsResponse) | Accounts returns all the existing accounts

Since: cosmos-sdk 0.43 | GET|/cosmos/auth/v1beta1/accounts|
| `Account` | [QueryAccountRequest](#cosmos.auth.v1beta1.QueryAccountRequest) | [QueryAccountResponse](#cosmos.auth.v1beta1.QueryAccountResponse) | Account returns account details based on address. | GET|/cosmos/auth/v1beta1/accounts/{address}|
| `Params` | [QueryParamsRequest](#cosmos.auth.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.auth.v1beta1.QueryParamsResponse) | Params queries all parameters. | GET|/cosmos/auth/v1beta1/params|
| `ModuleAccounts` | [QueryModuleAccountsRequest](#cosmos.auth.v1beta1.QueryModuleAccountsRequest) | [QueryModuleAccountsResponse](#cosmos.auth.v1beta1.QueryModuleAccountsResponse) | ModuleAccounts returns all the existing module accounts. | GET|/cosmos/auth/v1beta1/module_accounts|
| `Bech32Prefix` | [Bech32PrefixRequest](#cosmos.auth.v1beta1.Bech32PrefixRequest) | [Bech32PrefixResponse](#cosmos.auth.v1beta1.Bech32PrefixResponse) | Bech32 queries bech32Prefix | GET|/cosmos/auth/v1beta1/bech32|
| `AddressBytesToString` | [AddressBytesToStringRequest](#cosmos.auth.v1beta1.AddressBytesToStringRequest) | [AddressBytesToStringResponse](#cosmos.auth.v1beta1.AddressBytesToStringResponse) | AddressBytesToString converts Account Address bytes to string | GET|/cosmos/auth/v1beta1/bech32/{address_bytes}|
| `AddressStringToBytes` | [AddressStringToBytesRequest](#cosmos.auth.v1beta1.AddressStringToBytesRequest) | [AddressStringToBytesResponse](#cosmos.auth.v1beta1.AddressStringToBytesResponse) | AddressStringToBytes converts Address string to bytes | GET|/cosmos/auth/v1beta1/bech32/{address_string}|

 <!-- end services -->



<a name="cosmos/authz/v1beta1/authz.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/authz/v1beta1/authz.proto
自: cosmos-sdk 0.43


<a name="cosmos.authz.v1beta1.GenericAuthorization"></a>

### GenericAuthorization
GenericAuthorization 授予受让人不受限制的权限，可以代表受让人的帐户执行所提供的方法。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg` | [string](#string) |  | Msg, identified by it's type URL, to grant unrestricted permissions to execute |






<a name="cosmos.authz.v1beta1.Grant"></a>

### Grant
授予权限以执行具有到期时间的提供方法。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `authorization` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `expiration` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/authz/v1beta1/event.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/authz/v1beta1/event.proto
Since: cosmos-sdk 0.43


<a name="cosmos.authz.v1beta1.EventGrant"></a>

### EventGrant
EventGrant 在 Msg/Grant 上发出


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg_type_url` | [string](#string) |  | Msg type URL for which an autorization is granted |
| `granter` | [string](#string) |  | Granter account address |
| `grantee` | [string](#string) |  | Grantee account address |






<a name="cosmos.authz.v1beta1.EventRevoke"></a>

### EventRevoke
EventRevoke在Msg/Revoke上发出


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg_type_url` | [string](#string) |  | Msg type URL for which an autorization is revoked |
| `granter` | [string](#string) |  | Granter account address |
| `grantee` | [string](#string) |  | Grantee account address |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/authz/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/authz/v1beta1/genesis.proto
Since: cosmos-sdk 0.43


<a name="cosmos.authz.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 authz 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `authorization` | [GrantAuthorization](#cosmos.authz.v1beta1.GrantAuthorization) | repeated |  |






<a name="cosmos.authz.v1beta1.GrantAuthorization"></a>

### GrantAuthorization
GrantAuthorization 定义了 GenesisState/GrantAuthorization 类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  |  |
| `grantee` | [string](#string) |  |  |
| `authorization` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `expiration` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/authz/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/authz/v1beta1/query.proto
Since: cosmos-sdk 0.43


<a name="cosmos.authz.v1beta1.QueryGranterGrantsRequest"></a>

### QueryGranterGrantsRequest
QueryGranterGrantsRequest 是 Query/GranterGrants RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  |  |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |






<a name="cosmos.authz.v1beta1.QueryGranterGrantsResponse"></a>

### QueryGranterGrantsResponse
QueryGranterGrantsResponse 是 Query/GranterGrants RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `grants` | [Grant](#cosmos.authz.v1beta1.Grant) | repeated | authorizations is a list of grants granted for grantee by granter. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |






<a name="cosmos.authz.v1beta1.QueryGrantsRequest"></a>

### QueryGrantsRequest
QueryGrantsRequest是Query/Grants RPC方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  |  |
| `grantee` | [string](#string) |  |  |
| `msg_type_url` | [string](#string) |  | Optional, msg_type_url, when set, will query only grants matching given msg type. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |






<a name="cosmos.authz.v1beta1.QueryGrantsResponse"></a>

### QueryGrantsResponse
QueryGrantsResponse 是 Query/Authorizations RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `grants` | [Grant](#cosmos.authz.v1beta1.Grant) | repeated | authorizations is a list of grants granted for grantee by granter. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.authz.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Grants` | [QueryGrantsRequest](#cosmos.authz.v1beta1.QueryGrantsRequest) | [QueryGrantsResponse](#cosmos.authz.v1beta1.QueryGrantsResponse) | Returns list of `Authorization`, granted to the grantee by the granter. | GET|/cosmos/authz/v1beta1/grants|
| `GranterGrants` | [QueryGranterGrantsRequest](#cosmos.authz.v1beta1.QueryGranterGrantsRequest) | [QueryGranterGrantsResponse](#cosmos.authz.v1beta1.QueryGranterGrantsResponse) | GranterGrants returns list of `Authorization`, granted by granter. | GET|/cosmos/authz/v1beta1/grants/{granter}|

 <!-- end services -->



<a name="cosmos/authz/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/authz/v1beta1/tx.proto
Since: cosmos-sdk 0.43


<a name="cosmos.authz.v1beta1.MsgExec"></a>

### MsgExec
MsgExec 尝试使用授予受让人的授权来执行提供的消息。 每条消息应该只有一个与授权授予者相对应的签名者。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `grantee` | [string](#string) |  |  |
| `msgs` | [google.protobuf.Any](#google.protobuf.Any) | repeated | Authorization Msg requests to execute. Each msg must implement Authorization interface The x/authz will try to find a grant matching (msg.signers[0], grantee, MsgTypeURL(msg)) triple and validate it. |






<a name="cosmos.authz.v1beta1.MsgExecResponse"></a>

### MsgExecResponse
MsgExecResponse 定义了 Msg/MsgExecResponse 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `results` | [bytes](#bytes) | repeated |  |






<a name="cosmos.authz.v1beta1.MsgGrant"></a>

### MsgGrant
MsgGrant 是 Grant 方法的请求类型。 它代表授予者声明对授予者的授权，并具有提供的到期时间。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  |  |
| `grantee` | [string](#string) |  |  |
| `grant` | [Grant](#cosmos.authz.v1beta1.Grant) |  |  |






<a name="cosmos.authz.v1beta1.MsgGrantResponse"></a>

### MsgGrantResponse
消息授予响应定义消息/消息授予响应类型。






<a name="cosmos.authz.v1beta1.MsgRevoke"></a>

### MsgRevoke
MsgRevoke 撤销授予者帐户上提供的 sdk.Msg 类型的任何授权，该授权已授予被授予者。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  |  |
| `grantee` | [string](#string) |  |  |
| `msg_type_url` | [string](#string) |  |  |






<a name="cosmos.authz.v1beta1.MsgRevokeResponse"></a>

### MsgRevokeResponse
MsgRevokeResponse 定义了 Msg/MsgRevokeResponse 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.authz.v1beta1.Msg"></a>

### Msg
Msg定义了authz Msg服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Grant` | [MsgGrant](#cosmos.authz.v1beta1.MsgGrant) | [MsgGrantResponse](#cosmos.authz.v1beta1.MsgGrantResponse) | Grant grants the provided authorization to the grantee on the granter's account with the provided expiration time. If there is already a grant for the given (granter, grantee, Authorization) triple, then the grant will be overwritten. | |
| `Exec` | [MsgExec](#cosmos.authz.v1beta1.MsgExec) | [MsgExecResponse](#cosmos.authz.v1beta1.MsgExecResponse) | Exec attempts to execute the provided messages using authorizations granted to the grantee. Each message should have only one signer corresponding to the granter of the authorization. | |
| `Revoke` | [MsgRevoke](#cosmos.authz.v1beta1.MsgRevoke) | [MsgRevokeResponse](#cosmos.authz.v1beta1.MsgRevokeResponse) | Revoke revokes any authorization corresponding to the provided method name on the granter's account that has been granted to the grantee. | |

 <!-- end services -->



<a name="cosmos/base/v1beta1/coin.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/v1beta1/coin.proto



<a name="cosmos.base.v1beta1.Coin"></a>

### Coin
Coin 定义了一个带有面额和数量的代币。

注意:amount 字段是一个 Int，它实现了 gogoproto 所需的自定义方法签名。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  |  |
| `amount` | [string](#string) |  |  |






<a name="cosmos.base.v1beta1.DecCoin"></a>

### DecCoin
DecCoin 定义了一个带有面额和十进制金额的代币。

注意:amount 字段是一个 Dec，它实现了 gogoproto 所需的自定义方法签名。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  |  |
| `amount` | [string](#string) |  |  |






<a name="cosmos.base.v1beta1.DecProto"></a>

### DecProto
DecProto 定义了一个围绕 Dec 对象的 Protobuf 包装器。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `dec` | [string](#string) |  |  |






<a name="cosmos.base.v1beta1.IntProto"></a>

### IntProto
IntProto 定义了一个围绕 Int 对象的 Protobuf 包装器。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `int` | [string](#string) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/bank/v1beta1/authz.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/bank/v1beta1/authz.proto



<a name="cosmos.bank.v1beta1.SendAuthorization"></a>

### SendAuthorization
SendAuthorization 允许受赠者从受赠者的帐户中花费最多 pay_limit 硬币。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `spend_limit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/bank/v1beta1/bank.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/bank/v1beta1/bank.proto



<a name="cosmos.bank.v1beta1.DenomUnit"></a>

### DenomUnit
DenomUnit 表示一个结构体，它描述了基本标记的给定面额单位。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  | denom represents the string name of the given denom unit (e.g uatom). |
| `exponent` | [uint32](#uint32) |  | exponent represents power of 10 exponent that one must raise the base_denom to in order to equal the given DenomUnit's denom 1 denom = 10^exponent base_denom (e.g. with a base_denom of uatom, one can create a DenomUnit of 'atom' with exponent = 6, thus: 1 atom = 10^6 uatom). |
| `aliases` | [string](#string) | repeated | aliases is a list of string aliases for the given denom |






<a name="cosmos.bank.v1beta1.Input"></a>

### Input
输入模型交易输入。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  |  |
| `coins` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.bank.v1beta1.Metadata"></a>

### Metadata
元数据表示描述基本令牌的结构。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `description` | [string](#string) |  |  |
| `denom_units` | [DenomUnit](#cosmos.bank.v1beta1.DenomUnit) | repeated | denom_units represents the list of DenomUnit's for a given coin |
| `base` | [string](#string) |  | base represents the base denom (should be the DenomUnit with exponent = 0). |
| `display` | [string](#string) |  | display indicates the suggested denom that should be displayed in clients. |
| `name` | [string](#string) |  | name defines the name of the token (eg: Cosmos Atom)

Since: cosmos-sdk 0.43 |
| `symbol` | [string](#string) |  | symbol is the token symbol usually shown on exchanges (eg: ATOM). This can be the same as the display.

Since: cosmos-sdk 0.43 |
| `uri` | [string](#string) |  | URI to a document (on or off-chain) that contains additional information. Optional.

Since: cosmos-sdk 0.45 |
| `uri_hash` | [string](#string) |  | URIHash is a sha256 hash of a document pointed by URI. It's used to verify that the document didn't change. Optional.

Since: cosmos-sdk 0.45 |






<a name="cosmos.bank.v1beta1.Output"></a>

### Output
输出模型交易输出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  |  |
| `coins` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.bank.v1beta1.Params"></a>

### Params
Params 定义了银行模块的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `send_enabled` | [SendEnabled](#cosmos.bank.v1beta1.SendEnabled) | repeated |  |
| `default_send_enabled` | [bool](#bool) |  |  |






<a name="cosmos.bank.v1beta1.SendEnabled"></a>

### SendEnabled
SendEnabled 将硬币 denom 映射到 send_enabled 状态(denom 是否可发送)。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  |  |
| `enabled` | [bool](#bool) |  |  |






<a name="cosmos.bank.v1beta1.Supply"></a>

### Supply
Supply 代表一个结构，它被动地跟踪网络中的总供应量。
由于供应按 denom 索引，因此不推荐使用此消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `total` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/bank/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/bank/v1beta1/genesis.proto



<a name="cosmos.bank.v1beta1.Balance"></a>

### Balance
Balance 定义了在银行模块的创世状态中使用的帐户地址和余额对。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address of the balance holder. |
| `coins` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | coins defines the different coins this balance holds. |






<a name="cosmos.bank.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了银行模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.bank.v1beta1.Params) |  | params defines all the paramaters of the module. |
| `balances` | [Balance](#cosmos.bank.v1beta1.Balance) | repeated | balances is an array containing the balances of all the accounts. |
| `supply` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | supply represents the total supply. If it is left empty, then supply will be calculated based on the provided balances. Otherwise, it will be used to validate that the sum of the balances equals this amount. |
| `denom_metadata` | [Metadata](#cosmos.bank.v1beta1.Metadata) | repeated | denom_metadata defines the metadata of the differents coins. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/bank/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/bank/v1beta1/query.proto



<a name="cosmos.bank.v1beta1.DenomOwner"></a>

### DenomOwner
DenomOwner 定义表示拥有或持有特定计价代币的帐户的结构。 它包含计价代币的账户地址和账户余额。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address defines the address that owns a particular denomination. |
| `balance` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  | balance is the balance of the denominated coin for an account. |






<a name="cosmos.bank.v1beta1.QueryAllBalancesRequest"></a>

### QueryAllBalancesRequest
QueryBalanceRequest 是 Query/AllBalances RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address to query balances for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.bank.v1beta1.QueryAllBalancesResponse"></a>

### QueryAllBalancesResponse
QueryAllBalancesResponse 是 Query/AllBalances RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `balances` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | balances is the balances of all the coins. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.bank.v1beta1.QueryBalanceRequest"></a>

### QueryBalanceRequest
QueryBalanceRequest 是 Query/Balance RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address to query balances for. |
| `denom` | [string](#string) |  | denom is the coin denom to query balances for. |






<a name="cosmos.bank.v1beta1.QueryBalanceResponse"></a>

### QueryBalanceResponse
QueryBalanceResponse 是 Query/Balance RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `balance` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  | balance is the balance of the coin. |






<a name="cosmos.bank.v1beta1.QueryDenomMetadataRequest"></a>

### QueryDenomMetadataRequest
QueryDenomMetadataRequest 是 Query/DenomMetadata RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  | denom is the coin denom to query the metadata for. |






<a name="cosmos.bank.v1beta1.QueryDenomMetadataResponse"></a>

### QueryDenomMetadataResponse
QueryDenomMetadataResponse 是 Query/DenomMetadata RPC 方法的响应类型。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `metadata` | [Metadata](#cosmos.bank.v1beta1.Metadata) |  | metadata describes and provides all the client information for the requested token. |






<a name="cosmos.bank.v1beta1.QueryDenomOwnersRequest"></a>

### QueryDenomOwnersRequest
QueryDenomOwnersRequest 定义了 DenomOwners RPC 查询的请求类型，该查询查询特定面额的所有账户持有人的分页集合。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  | denom defines the coin denomination to query all account holders for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.bank.v1beta1.QueryDenomOwnersResponse"></a>

### QueryDenomOwnersResponse
QueryDenomOwnersResponse 定义了 DenomOwners RPC 查询的 RPC 响应。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom_owners` | [DenomOwner](#cosmos.bank.v1beta1.DenomOwner) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.bank.v1beta1.QueryDenomsMetadataRequest"></a>

### QueryDenomsMetadataRequest
QueryDenomsMetadataRequest 是 Query/DenomsMetadata RPC 方法的请求类型。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.bank.v1beta1.QueryDenomsMetadataResponse"></a>

### QueryDenomsMetadataResponse
QueryDenomsMetadataResponse 是 Query/DenomsMetadata RPC 方法的响应类型。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `metadatas` | [Metadata](#cosmos.bank.v1beta1.Metadata) | repeated | metadata provides the client information for all the registered tokens. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.bank.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 定义了查询 x/bank 参数的请求类型。






<a name="cosmos.bank.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 定义了查询 x/bank 参数的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.bank.v1beta1.Params) |  |  |






<a name="cosmos.bank.v1beta1.QuerySupplyOfRequest"></a>

### QuerySupplyOfRequest
QuerySupplyOfRequest 是 Query/SupplyOf RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `denom` | [string](#string) |  | denom is the coin denom to query balances for. |






<a name="cosmos.bank.v1beta1.QuerySupplyOfResponse"></a>

### QuerySupplyOfResponse
QuerySupplyOfResponse 是 Query/SupplyOf RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  | amount is the supply of the coin. |






<a name="cosmos.bank.v1beta1.QueryTotalSupplyRequest"></a>

### QueryTotalSupplyRequest
QueryTotalSupplyRequest 是 Query/TotalSupply RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request.

Since: cosmos-sdk 0.43 |






<a name="cosmos.bank.v1beta1.QueryTotalSupplyResponse"></a>

### QueryTotalSupplyResponse
QueryTotalSupplyResponse 是 Query/TotalSupply RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `supply` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | supply is the supply of the coins |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response.

Since: cosmos-sdk 0.43 |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.bank.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Balance` | [QueryBalanceRequest](#cosmos.bank.v1beta1.QueryBalanceRequest) | [QueryBalanceResponse](#cosmos.bank.v1beta1.QueryBalanceResponse) | Balance queries the balance of a single coin for a single account. | GET|/cosmos/bank/v1beta1/balances/{address}/by_denom|
| `AllBalances` | [QueryAllBalancesRequest](#cosmos.bank.v1beta1.QueryAllBalancesRequest) | [QueryAllBalancesResponse](#cosmos.bank.v1beta1.QueryAllBalancesResponse) | AllBalances queries the balance of all coins for a single account. | GET|/cosmos/bank/v1beta1/balances/{address}|
| `TotalSupply` | [QueryTotalSupplyRequest](#cosmos.bank.v1beta1.QueryTotalSupplyRequest) | [QueryTotalSupplyResponse](#cosmos.bank.v1beta1.QueryTotalSupplyResponse) | TotalSupply queries the total supply of all coins. | GET|/cosmos/bank/v1beta1/supply|
| `SupplyOf` | [QuerySupplyOfRequest](#cosmos.bank.v1beta1.QuerySupplyOfRequest) | [QuerySupplyOfResponse](#cosmos.bank.v1beta1.QuerySupplyOfResponse) | SupplyOf queries the supply of a single coin. | GET|/cosmos/bank/v1beta1/supply/{denom}|
| `Params` | [QueryParamsRequest](#cosmos.bank.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.bank.v1beta1.QueryParamsResponse) | Params queries the parameters of x/bank module. | GET|/cosmos/bank/v1beta1/params|
| `DenomMetadata` | [QueryDenomMetadataRequest](#cosmos.bank.v1beta1.QueryDenomMetadataRequest) | [QueryDenomMetadataResponse](#cosmos.bank.v1beta1.QueryDenomMetadataResponse) | DenomsMetadata queries the client metadata of a given coin denomination. | GET|/cosmos/bank/v1beta1/denoms_metadata/{denom}|
| `DenomsMetadata` | [QueryDenomsMetadataRequest](#cosmos.bank.v1beta1.QueryDenomsMetadataRequest) | [QueryDenomsMetadataResponse](#cosmos.bank.v1beta1.QueryDenomsMetadataResponse) | DenomsMetadata queries the client metadata for all registered coin denominations. | GET|/cosmos/bank/v1beta1/denoms_metadata|
| `DenomOwners` | [QueryDenomOwnersRequest](#cosmos.bank.v1beta1.QueryDenomOwnersRequest) | [QueryDenomOwnersResponse](#cosmos.bank.v1beta1.QueryDenomOwnersResponse) | DenomOwners queries for all account addresses that own a particular token denomination. | GET|/cosmos/bank/v1beta1/denom_owners/{denom}|

 <!-- end services -->



<a name="cosmos/bank/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/bank/v1beta1/tx.proto



<a name="cosmos.bank.v1beta1.MsgMultiSend"></a>

### MsgMultiSend
MsgMultiSend 表示任意多进多出的发送消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `inputs` | [Input](#cosmos.bank.v1beta1.Input) | repeated |  |
| `outputs` | [Output](#cosmos.bank.v1beta1.Output) | repeated |  |






<a name="cosmos.bank.v1beta1.MsgMultiSendResponse"></a>

### MsgMultiSendResponse
MsgMultiSendResponse 定义了 Msg/MultiSend 响应类型。






<a name="cosmos.bank.v1beta1.MsgSend"></a>

### MsgSend
MsgSend 表示将硬币从一个帐户发送到另一个帐户的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `from_address` | [string](#string) |  |  |
| `to_address` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.bank.v1beta1.MsgSendResponse"></a>

### MsgSendResponse
MsgSendResponse 定义了 Msg/Send 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.bank.v1beta1.Msg"></a>

### Msg
Msg 定义了银行消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Send` | [MsgSend](#cosmos.bank.v1beta1.MsgSend) | [MsgSendResponse](#cosmos.bank.v1beta1.MsgSendResponse) | Send defines a method for sending coins from one account to another account. | |
| `MultiSend` | [MsgMultiSend](#cosmos.bank.v1beta1.MsgMultiSend) | [MsgMultiSendResponse](#cosmos.bank.v1beta1.MsgMultiSendResponse) | MultiSend defines a method for sending coins from some accounts to other accounts. | |

 <!-- end services -->



<a name="cosmos/base/abci/v1beta1/abci.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/abci/v1beta1/abci.proto



<a name="cosmos.base.abci.v1beta1.ABCIMessageLog"></a>

### ABCIMessageLog
ABCIMessageLog 定义了一个包含索引的 tx ABCI 消息日志的结构。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg_index` | [uint32](#uint32) |  |  |
| `log` | [string](#string) |  |  |
| `events` | [StringEvent](#cosmos.base.abci.v1beta1.StringEvent) | repeated | Events contains a slice of Event objects that were emitted during some execution. |






<a name="cosmos.base.abci.v1beta1.Attribute"></a>

### Attribute
Attribute 定义了一个属性包装器，其中键和值是字符串而不是原始字节。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [string](#string) |  |  |
| `value` | [string](#string) |  |  |






<a name="cosmos.base.abci.v1beta1.GasInfo"></a>

### GasInfo
GasInfo 定义了 tx 执行GAS费上下文。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `gas_wanted` | [uint64](#uint64) |  | GasWanted is the maximum units of work we allow this tx to perform. |
| `gas_used` | [uint64](#uint64) |  | GasUsed is the amount of gas actually consumed. |






<a name="cosmos.base.abci.v1beta1.MsgData"></a>

### MsgData
MsgData 定义了消息执行期间在 Result 对象中返回的数据。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg_type` | [string](#string) |  |  |
| `data` | [bytes](#bytes) |  |  |






<a name="cosmos.base.abci.v1beta1.Result"></a>

### Result
结果是 ResponseFormat 和 ResponseCheckTx 的并集。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `data` | [bytes](#bytes) |  | **Deprecated.** Data is any data returned from message or handler execution. It MUST be length prefixed in order to separate data from multiple message executions. Deprecated. This field is still populated, but prefer msg_response instead because it also contains the Msg response typeURL. |
| `log` | [string](#string) |  | Log contains the log information from message or handler execution. |
| `events` | [tendermint.abci.Event](#tendermint.abci.Event) | repeated | Events contains a slice of Event objects that were emitted during message or handler execution. |
| `msg_responses` | [google.protobuf.Any](#google.protobuf.Any) | repeated | msg_responses contains the Msg handler responses type packed in Anys.

Since: cosmos-sdk 0.45 |






<a name="cosmos.base.abci.v1beta1.SearchTxsResult"></a>

### SearchTxsResult
SearchTxsResult 定义了一个用于查询 txs pageable 的结构


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `total_count` | [uint64](#uint64) |  | Count of all txs |
| `count` | [uint64](#uint64) |  | Count of txs in current page |
| `page_number` | [uint64](#uint64) |  | Index of current page, start from 1 |
| `page_total` | [uint64](#uint64) |  | Count of total pages |
| `limit` | [uint64](#uint64) |  | Max count txs per page |
| `txs` | [TxResponse](#cosmos.base.abci.v1beta1.TxResponse) | repeated | List of txs in current page |






<a name="cosmos.base.abci.v1beta1.SimulationResponse"></a>

### SimulationResponse
SimulationResponse 定义了成功模拟事务时生成的响应。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `gas_info` | [GasInfo](#cosmos.base.abci.v1beta1.GasInfo) |  |  |
| `result` | [Result](#cosmos.base.abci.v1beta1.Result) |  |  |






<a name="cosmos.base.abci.v1beta1.StringEvent"></a>

### StringEvent
StringEvent 定义了事件对象包装器，其中所有属性都包含字符串而不是原始字节的键/值对。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `type` | [string](#string) |  |  |
| `attributes` | [Attribute](#cosmos.base.abci.v1beta1.Attribute) | repeated |  |






<a name="cosmos.base.abci.v1beta1.TxMsgData"></a>

### TxMsgData
TxMsgData 定义了一个 MsgData 列表。 对于每条消息，事务都有一个 MsgData 对象。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `data` | [MsgData](#cosmos.base.abci.v1beta1.MsgData) | repeated | **Deprecated.** data field is deprecated and not populated. |
| `msg_responses` | [google.protobuf.Any](#google.protobuf.Any) | repeated | msg_responses contains the Msg handler responses packed into Anys.

Since: cosmos-sdk 0.45 |






<a name="cosmos.base.abci.v1beta1.TxResponse"></a>

### TxResponse
TxResponse 定义了一个包含相关交易数据和元数据的结构。 标签是字符串化的，日志是 JSON 解码的。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  | The block height |
| `txhash` | [string](#string) |  | The transaction hash. |
| `codespace` | [string](#string) |  | Namespace for the Code |
| `code` | [uint32](#uint32) |  | Response code. |
| `data` | [string](#string) |  | Result bytes, if any. |
| `raw_log` | [string](#string) |  | The output of the application's logger (raw string). May be non-deterministic. |
| `logs` | [ABCIMessageLog](#cosmos.base.abci.v1beta1.ABCIMessageLog) | repeated | The output of the application's logger (typed). May be non-deterministic. |
| `info` | [string](#string) |  | Additional information. May be non-deterministic. |
| `gas_wanted` | [int64](#int64) |  | Amount of gas requested for transaction. |
| `gas_used` | [int64](#int64) |  | Amount of gas consumed by transaction. |
| `tx` | [google.protobuf.Any](#google.protobuf.Any) |  | The request transaction bytes. |
| `timestamp` | [string](#string) |  | Time of the previous block. For heights > 1, it's the weighted median of the timestamps of the valid votes in the block.LastCommit. For height == 1, it's genesis time. |
| `events` | [tendermint.abci.Event](#tendermint.abci.Event) | repeated | Events defines all the events emitted by processing a transaction. Note, these events include those emitted by processing all the messages and those emitted from the ante handler. Whereas Logs contains the events, with additional metadata, emitted only by processing the messages.

Since: cosmos-sdk 0.42.11, 0.44.5, 0.45 |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/kv/v1beta1/kv.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/kv/v1beta1/kv.proto



<a name="cosmos.base.kv.v1beta1.Pair"></a>

### Pair
Pair 定义了一个键/值字节元组。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |
| `value` | [bytes](#bytes) |  |  |






<a name="cosmos.base.kv.v1beta1.Pairs"></a>

### Pairs
Pairs 定义了 Pair 对象的重复切片。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pairs` | [Pair](#cosmos.base.kv.v1beta1.Pair) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/reflection/v1beta1/reflection.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/reflection/v1beta1/reflection.proto



<a name="cosmos.base.reflection.v1beta1.ListAllInterfacesRequest"></a>

### ListAllInterfacesRequest
ListAllInterfacesRequest 是 ListAllInterfaces RPC 的请求类型。






<a name="cosmos.base.reflection.v1beta1.ListAllInterfacesResponse"></a>

### ListAllInterfacesResponse
ListAllInterfacesResponse 是 ListAllInterfaces RPC 的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `interface_names` | [string](#string) | repeated | interface_names is an array of all the registered interfaces. |






<a name="cosmos.base.reflection.v1beta1.ListImplementationsRequest"></a>

### ListImplementationsRequest
ListImplementationsRequest 是 ListImplementations RPC 的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `interface_name` | [string](#string) |  | interface_name defines the interface to query the implementations for. |






<a name="cosmos.base.reflection.v1beta1.ListImplementationsResponse"></a>

### ListImplementationsResponse
ListImplementationsResponse 是 ListImplementations RPC 的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `implementation_message_names` | [string](#string) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.base.reflection.v1beta1.ReflectionService"></a>

### ReflectionService
ReflectionService 定义了一个接口反射服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `ListAllInterfaces` | [ListAllInterfacesRequest](#cosmos.base.reflection.v1beta1.ListAllInterfacesRequest) | [ListAllInterfacesResponse](#cosmos.base.reflection.v1beta1.ListAllInterfacesResponse) | ListAllInterfaces lists all the interfaces registered in the interface registry. | GET|/cosmos/base/reflection/v1beta1/interfaces|
| `ListImplementations` | [ListImplementationsRequest](#cosmos.base.reflection.v1beta1.ListImplementationsRequest) | [ListImplementationsResponse](#cosmos.base.reflection.v1beta1.ListImplementationsResponse) | ListImplementations list all the concrete types that implement a given interface. | GET|/cosmos/base/reflection/v1beta1/interfaces/{interface_name}/implementations|

 <!-- end services -->



<a name="cosmos/base/reflection/v2alpha1/reflection.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/reflection/v2alpha1/reflection.proto
Since: cosmos-sdk 0.43


<a name="cosmos.base.reflection.v2alpha1.AppDescriptor"></a>

### AppDescriptor
AppDescriptor 描述了一个基于 cosmos-sdk 的应用程序


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `authn` | [AuthnDescriptor](#cosmos.base.reflection.v2alpha1.AuthnDescriptor) |  | AuthnDescriptor provides information on how to authenticate transactions on the application NOTE: experimental and subject to change in future releases. |
| `chain` | [ChainDescriptor](#cosmos.base.reflection.v2alpha1.ChainDescriptor) |  | chain provides the chain descriptor |
| `codec` | [CodecDescriptor](#cosmos.base.reflection.v2alpha1.CodecDescriptor) |  | codec provides metadata information regarding codec related types |
| `configuration` | [ConfigurationDescriptor](#cosmos.base.reflection.v2alpha1.ConfigurationDescriptor) |  | configuration provides metadata information regarding the sdk.Config type |
| `query_services` | [QueryServicesDescriptor](#cosmos.base.reflection.v2alpha1.QueryServicesDescriptor) |  | query_services provides metadata information regarding the available queriable endpoints |
| `tx` | [TxDescriptor](#cosmos.base.reflection.v2alpha1.TxDescriptor) |  | tx provides metadata information regarding how to send transactions to the given application |






<a name="cosmos.base.reflection.v2alpha1.AuthnDescriptor"></a>

### AuthnDescriptor
AuthnDescriptor 提供有关如何在不依赖在线 RPC GetTxMetadata 和 CombineUnsignedTxAndSignatures 的情况下签署交易的信息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `sign_modes` | [SigningModeDescriptor](#cosmos.base.reflection.v2alpha1.SigningModeDescriptor) | repeated | sign_modes defines the supported signature algorithm |






<a name="cosmos.base.reflection.v2alpha1.ChainDescriptor"></a>

### ChainDescriptor
ChainDescriptor 描述应用的链信息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `id` | [string](#string) |  | id is the chain id |






<a name="cosmos.base.reflection.v2alpha1.CodecDescriptor"></a>

### CodecDescriptor
CodecDescriptor 描述注册的接口并提供有关类型的元数据信息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `interfaces` | [InterfaceDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceDescriptor) | repeated | interfaces is a list of the registerted interfaces descriptors |






<a name="cosmos.base.reflection.v2alpha1.ConfigurationDescriptor"></a>

### ConfigurationDescriptor
ConfigurationDescriptor 包含有关 sdk.Config 的元数据信息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `bech32_account_address_prefix` | [string](#string) |  | bech32_account_address_prefix is the account address prefix |






<a name="cosmos.base.reflection.v2alpha1.GetAuthnDescriptorRequest"></a>

### GetAuthnDescriptorRequest
GetAuthnDescriptorRequest 是用于 GetAuthnDescriptor RPC 的请求






<a name="cosmos.base.reflection.v2alpha1.GetAuthnDescriptorResponse"></a>

### GetAuthnDescriptorResponse
GetAuthnDescriptorResponse 是 GetAuthnDescriptor RPC 返回的响应


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `authn` | [AuthnDescriptor](#cosmos.base.reflection.v2alpha1.AuthnDescriptor) |  | authn describes how to authenticate to the application when sending transactions |






<a name="cosmos.base.reflection.v2alpha1.GetChainDescriptorRequest"></a>

### GetChainDescriptorRequest
GetCodecDescriptorRequest 是用于 GetCodecDescriptor RPC 的请求






<a name="cosmos.base.reflection.v2alpha1.GetChainDescriptorResponse"></a>

### GetChainDescriptorResponse
GetChainDescriptorResponse 是 GetChainDescriptorRPC 返回的响应。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `chain` | [ChainDescriptor](#cosmos.base.reflection.v2alpha1.ChainDescriptor) |  | chain describes application chain information |






<a name="cosmos.base.reflection.v2alpha1.GetCodecDescriptorRequest"></a>

### GetCodecDescriptorRequest
GetCodecDescriptorRequest 是用于 GetCodecDescriptorRPC 的请求。






<a name="cosmos.base.reflection.v2alpha1.GetCodecDescriptorResponse"></a>

### GetCodecDescriptorResponse
GetCodecDescriptorResponse 是 GetCodecDescriptorRPC 返回的响应。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `codec` | [CodecDescriptor](#cosmos.base.reflection.v2alpha1.CodecDescriptor) |  | codec describes the application codec such as registered interfaces and implementations |






<a name="cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorRequest"></a>

### GetConfigurationDescriptorRequest
GetConfigurationDescriptorRequest 是用于 GetConfigurationDescriptor RPC 的请求






<a name="cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorResponse"></a>

### GetConfigurationDescriptorResponse
GetConfigurationDescriptorResponse 是 GetConfigurationDescriptor RPC 返回的响应


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `config` | [ConfigurationDescriptor](#cosmos.base.reflection.v2alpha1.ConfigurationDescriptor) |  | config describes the application's sdk.Config |






<a name="cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorRequest"></a>

### GetQueryServicesDescriptorRequest
GetQueryServicesDescriptorRequest 是用于 GetQueryServicesDescriptor RPC 的请求






<a name="cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorResponse"></a>

### GetQueryServicesDescriptorResponse
GetQueryServicesDescriptorResponse 是 GetQueryServicesDescriptor RPC 返回的响应

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `queries` | [QueryServicesDescriptor](#cosmos.base.reflection.v2alpha1.QueryServicesDescriptor) |  | queries provides information on the available queryable services |






<a name="cosmos.base.reflection.v2alpha1.GetTxDescriptorRequest"></a>

### GetTxDescriptorRequest
GetTxDescriptorRequest 是用于 GetTxDescriptor RPC 的请求






<a name="cosmos.base.reflection.v2alpha1.GetTxDescriptorResponse"></a>

### GetTxDescriptorResponse
GetTxDescriptorResponse 是 GetTxDescriptor RPC 返回的响应


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tx` | [TxDescriptor](#cosmos.base.reflection.v2alpha1.TxDescriptor) |  | tx provides information on msgs that can be forwarded to the application alongside the accepted transaction protobuf type |






<a name="cosmos.base.reflection.v2alpha1.InterfaceAcceptingMessageDescriptor"></a>

### InterfaceAcceptingMessageDescriptor
InterfaceAcceptingMessageDescriptor 描述了一个 protobuf 消息，其中包含一个表示为 google.protobuf.Any 的接口


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `fullname` | [string](#string) |  | fullname is the protobuf fullname of the type containing the interface |
| `field_descriptor_names` | [string](#string) | repeated | field_descriptor_names is a list of the protobuf name (not fullname) of the field which contains the interface as google.protobuf.Any (the interface is the same, but it can be in multiple fields of the same proto message) |






<a name="cosmos.base.reflection.v2alpha1.InterfaceDescriptor"></a>

### InterfaceDescriptor
InterfaceDescriptor 描述了一个接口的实现


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `fullname` | [string](#string) |  | fullname is the name of the interface |
| `interface_accepting_messages` | [InterfaceAcceptingMessageDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceAcceptingMessageDescriptor) | repeated | interface_accepting_messages contains information regarding the proto messages which contain the interface as google.protobuf.Any field |
| `interface_implementers` | [InterfaceImplementerDescriptor](#cosmos.base.reflection.v2alpha1.InterfaceImplementerDescriptor) | repeated | interface_implementers is a list of the descriptors of the interface implementers |






<a name="cosmos.base.reflection.v2alpha1.InterfaceImplementerDescriptor"></a>

### InterfaceImplementerDescriptor
InterfaceImplementerDescriptor 描述了一个接口实现者


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `fullname` | [string](#string) |  | fullname is the protobuf queryable name of the interface implementer |
| `type_url` | [string](#string) |  | type_url defines the type URL used when marshalling the type as any this is required so we can provide type safe google.protobuf.Any marshalling and unmarshalling, making sure that we don't accept just 'any' type in our interface fields |






<a name="cosmos.base.reflection.v2alpha1.MsgDescriptor"></a>

### MsgDescriptor
MsgDescriptor 描述了一个可以与事务一起传递的 cosmos-sdk 消息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `msg_type_url` | [string](#string) |  | msg_type_url contains the TypeURL of a sdk.Msg. |






<a name="cosmos.base.reflection.v2alpha1.QueryMethodDescriptor"></a>

### QueryMethodDescriptor
QueryMethodDescriptor 描述了查询服务的可查询方法，除了方法名称和tendermint 可查询路径之外没有提供其他信息，因为它与 grpc 反射服务是多余的

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | name is the protobuf name (not fullname) of the method |
| `full_query_path` | [string](#string) |  | full_query_path is the path that can be used to query this method via tendermint abci.Query |






<a name="cosmos.base.reflection.v2alpha1.QueryServiceDescriptor"></a>

### QueryServiceDescriptor
QueryServiceDescriptor 描述了一个 cosmos-sdk 可查询服务


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `fullname` | [string](#string) |  | fullname is the protobuf fullname of the service descriptor |
| `is_module` | [bool](#bool) |  | is_module describes if this service is actually exposed by an application's module |
| `methods` | [QueryMethodDescriptor](#cosmos.base.reflection.v2alpha1.QueryMethodDescriptor) | repeated | methods provides a list of query service methods |






<a name="cosmos.base.reflection.v2alpha1.QueryServicesDescriptor"></a>

### QueryServicesDescriptor
QueryServicesDescriptor 包含 cosmos-sdk 可查询服务列表


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `query_services` | [QueryServiceDescriptor](#cosmos.base.reflection.v2alpha1.QueryServiceDescriptor) | repeated | query_services is a list of cosmos-sdk QueryServiceDescriptor |






<a name="cosmos.base.reflection.v2alpha1.SigningModeDescriptor"></a>

### SigningModeDescriptor
SigningModeDescriptor 提供有关应用程序签名流程的信息 NOTE(fdymylja):这里我们可以提供有关如何在给定 SigningModeDescriptor 的情况下签署消息的完整流程，但最好再考虑一下


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | name defines the unique name of the signing mode |
| `number` | [int32](#int32) |  | number is the unique int32 identifier for the sign_mode enum |
| `authn_info_provider_method_fullname` | [string](#string) |  | authn_info_provider_method_fullname defines the fullname of the method to call to get the metadata required to authenticate using the provided sign_modes |






<a name="cosmos.base.reflection.v2alpha1.TxDescriptor"></a>

### TxDescriptor
TxDescriptor 描述接受的交易类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `fullname` | [string](#string) |  | fullname is the protobuf fullname of the raw transaction type (for instance the tx.Tx type) it is not meant to support polymorphism of transaction types, it is supposed to be used by reflection clients to understand if they can handle a specific transaction type in an application. |
| `msgs` | [MsgDescriptor](#cosmos.base.reflection.v2alpha1.MsgDescriptor) | repeated | msgs lists the accepted application messages (sdk.Msg) |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.base.reflection.v2alpha1.ReflectionService"></a>

### ReflectionService
ReflectionService 为应用程序反射定义了一个服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `GetAuthnDescriptor` | [GetAuthnDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetAuthnDescriptorRequest) | [GetAuthnDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetAuthnDescriptorResponse) | GetAuthnDescriptor returns information on how to authenticate transactions in the application NOTE: this RPC is still experimental and might be subject to breaking changes or removal in future releases of the cosmos-sdk. | GET|/cosmos/base/reflection/v1beta1/app_descriptor/authn|
| `GetChainDescriptor` | [GetChainDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetChainDescriptorRequest) | [GetChainDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetChainDescriptorResponse) | GetChainDescriptor returns the description of the chain | GET|/cosmos/base/reflection/v1beta1/app_descriptor/chain|
| `GetCodecDescriptor` | [GetCodecDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetCodecDescriptorRequest) | [GetCodecDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetCodecDescriptorResponse) | GetCodecDescriptor returns the descriptor of the codec of the application | GET|/cosmos/base/reflection/v1beta1/app_descriptor/codec|
| `GetConfigurationDescriptor` | [GetConfigurationDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorRequest) | [GetConfigurationDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetConfigurationDescriptorResponse) | GetConfigurationDescriptor returns the descriptor for the sdk.Config of the application | GET|/cosmos/base/reflection/v1beta1/app_descriptor/configuration|
| `GetQueryServicesDescriptor` | [GetQueryServicesDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorRequest) | [GetQueryServicesDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetQueryServicesDescriptorResponse) | GetQueryServicesDescriptor returns the available gRPC queryable services of the application | GET|/cosmos/base/reflection/v1beta1/app_descriptor/query_services|
| `GetTxDescriptor` | [GetTxDescriptorRequest](#cosmos.base.reflection.v2alpha1.GetTxDescriptorRequest) | [GetTxDescriptorResponse](#cosmos.base.reflection.v2alpha1.GetTxDescriptorResponse) | GetTxDescriptor returns information on the used transaction object and available msgs that can be used | GET|/cosmos/base/reflection/v1beta1/app_descriptor/tx_descriptor|

 <!-- end services -->



<a name="cosmos/base/snapshots/v1beta1/snapshot.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/snapshots/v1beta1/snapshot.proto



<a name="cosmos.base.snapshots.v1beta1.Metadata"></a>

### Metadata
元数据包含特定于 SDK 的快照元数据。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `chunk_hashes` | [bytes](#bytes) | repeated | SHA-256 chunk hashes |






<a name="cosmos.base.snapshots.v1beta1.Snapshot"></a>

### Snapshot
快照包含 Tendermint 状态同步快照信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [uint64](#uint64) |  |  |
| `format` | [uint32](#uint32) |  |  |
| `chunks` | [uint32](#uint32) |  |  |
| `hash` | [bytes](#bytes) |  |  |
| `metadata` | [Metadata](#cosmos.base.snapshots.v1beta1.Metadata) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/store/v1beta1/commit_info.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/store/v1beta1/commit_info.proto



<a name="cosmos.base.store.v1beta1.CommitID"></a>

### CommitID
提交 ID 定义提交特定存储时的提交信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `version` | [int64](#int64) |  |  |
| `hash` | [bytes](#bytes) |  |  |






<a name="cosmos.base.store.v1beta1.CommitInfo"></a>

### CommitInfo
CommitInfo 定义了多存储在提交版本/高度时使用的提交信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `version` | [int64](#int64) |  |  |
| `store_infos` | [StoreInfo](#cosmos.base.store.v1beta1.StoreInfo) | repeated |  |






<a name="cosmos.base.store.v1beta1.StoreInfo"></a>

### StoreInfo
StoreInfo 定义特定于商店的提交信息。 它包含存储名称和提交 ID 之间的引用。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  |  |
| `commit_id` | [CommitID](#cosmos.base.store.v1beta1.CommitID) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/store/v1beta1/listening.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/store/v1beta1/listening.proto



<a name="cosmos.base.store.v1beta1.StoreKVPair"></a>

### StoreKVPair
StoreKVPair 是一个 KVStore KVPair，用于监听状态变化(Sets 和 Deletes)，它可选地包含用于原始 KVStore 的 StoreKey 和一个用于区分 Sets 和 Deletes 的布尔标志

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `store_key` | [string](#string) |  | the store key for the KVStore this pair originates from |
| `delete` | [bool](#bool) |  | true indicates a delete operation, false indicates a set operation |
| `key` | [bytes](#bytes) |  |  |
| `value` | [bytes](#bytes) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/store/v1beta1/snapshot.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/store/v1beta1/snapshot.proto



<a name="cosmos.base.store.v1beta1.SnapshotIAVLItem"></a>

### SnapshotIAVLItem
SnapshotIAVLItem 是导出的 IAVL 节点。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |
| `value` | [bytes](#bytes) |  |  |
| `version` | [int64](#int64) |  |  |
| `height` | [int32](#int32) |  |  |






<a name="cosmos.base.store.v1beta1.SnapshotItem"></a>

### SnapshotItem
SnapshotItem 是包含在 rootmulti.Store 快照中的项目。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `store` | [SnapshotStoreItem](#cosmos.base.store.v1beta1.SnapshotStoreItem) |  |  |
| `iavl` | [SnapshotIAVLItem](#cosmos.base.store.v1beta1.SnapshotIAVLItem) |  |  |






<a name="cosmos.base.store.v1beta1.SnapshotStoreItem"></a>

### SnapshotStoreItem
SnapshotStoreItem 包含有关快照存储的元数据。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/base/tendermint/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/base/tendermint/v1beta1/query.proto



<a name="cosmos.base.tendermint.v1beta1.GetBlockByHeightRequest"></a>

### GetBlockByHeightRequest
GetBlockByHeightRequest 是 Query/GetBlockByHeight RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  |  |






<a name="cosmos.base.tendermint.v1beta1.GetBlockByHeightResponse"></a>

### GetBlockByHeightResponse
GetBlockByHeightResponse 是 Query/GetBlockByHeight RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `block_id` | [tendermint.types.BlockID](#tendermint.types.BlockID) |  |  |
| `block` | [tendermint.types.Block](#tendermint.types.Block) |  |  |






<a name="cosmos.base.tendermint.v1beta1.GetLatestBlockRequest"></a>

### GetLatestBlockRequest
GetLatestBlockRequest 是 Query/GetLatestBlock RPC 方法的请求类型。






<a name="cosmos.base.tendermint.v1beta1.GetLatestBlockResponse"></a>

### GetLatestBlockResponse
GetLatestBlockResponse 是 Query/GetLatestBlock RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `block_id` | [tendermint.types.BlockID](#tendermint.types.BlockID) |  |  |
| `block` | [tendermint.types.Block](#tendermint.types.Block) |  |  |






<a name="cosmos.base.tendermint.v1beta1.GetLatestValidatorSetRequest"></a>

### GetLatestValidatorSetRequest
GetLatestValidatorSetRequest 是 Query/GetValidatorSetByHeight RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |






<a name="cosmos.base.tendermint.v1beta1.GetLatestValidatorSetResponse"></a>

### GetLatestValidatorSetResponse
GetLatestValidatorSetResponse 是 Query/GetValidatorSetByHeight RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `block_height` | [int64](#int64) |  |  |
| `validators` | [Validator](#cosmos.base.tendermint.v1beta1.Validator) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |






<a name="cosmos.base.tendermint.v1beta1.GetNodeInfoRequest"></a>

### GetNodeInfoRequest
GetNodeInfoRequest 是 Query/GetNodeInfo RPC 方法的请求类型。






<a name="cosmos.base.tendermint.v1beta1.GetNodeInfoResponse"></a>

### GetNodeInfoResponse
GetNodeInfoResponse 是 Query/GetNodeInfo RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `node_info` | [tendermint.p2p.NodeInfo](#tendermint.p2p.NodeInfo) |  |  |
| `application_version` | [VersionInfo](#cosmos.base.tendermint.v1beta1.VersionInfo) |  |  |






<a name="cosmos.base.tendermint.v1beta1.GetSyncingRequest"></a>

### GetSyncingRequest
GetSyncingRequest 是 Query/GetSyncing RPC 方法的请求类型。






<a name="cosmos.base.tendermint.v1beta1.GetSyncingResponse"></a>

### GetSyncingResponse
GetSyncingResponse 是 Query/GetSyncing RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `syncing` | [bool](#bool) |  |  |






<a name="cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightRequest"></a>

### GetValidatorSetByHeightRequest
GetValidatorSetByHeightRequest 是 Query/GetValidatorSetByHeight RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  |  |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |






<a name="cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightResponse"></a>

### GetValidatorSetByHeightResponse
GetValidatorSetByHeightResponse 是 Query/GetValidatorSetByHeight RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `block_height` | [int64](#int64) |  |  |
| `validators` | [Validator](#cosmos.base.tendermint.v1beta1.Validator) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |






<a name="cosmos.base.tendermint.v1beta1.Module"></a>

### Module
Module 是 VersionInfo 的类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `path` | [string](#string) |  | module path |
| `version` | [string](#string) |  | module version |
| `sum` | [string](#string) |  | checksum |






<a name="cosmos.base.tendermint.v1beta1.Validator"></a>

### Validator
Validator 是validator-set的类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  |  |
| `pub_key` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `voting_power` | [int64](#int64) |  |  |
| `proposer_priority` | [int64](#int64) |  |  |






<a name="cosmos.base.tendermint.v1beta1.VersionInfo"></a>

### VersionInfo
VersionInfo 是 GetNodeInfoResponse 消息的类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  |  |
| `app_name` | [string](#string) |  |  |
| `version` | [string](#string) |  |  |
| `git_commit` | [string](#string) |  |  |
| `build_tags` | [string](#string) |  |  |
| `go_version` | [string](#string) |  |  |
| `build_deps` | [Module](#cosmos.base.tendermint.v1beta1.Module) | repeated |  |
| `cosmos_sdk_version` | [string](#string) |  | Since: cosmos-sdk 0.43 |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.base.tendermint.v1beta1.Service"></a>

### Service
Service 定义了用于tendermint 查询的gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `GetNodeInfo` | [GetNodeInfoRequest](#cosmos.base.tendermint.v1beta1.GetNodeInfoRequest) | [GetNodeInfoResponse](#cosmos.base.tendermint.v1beta1.GetNodeInfoResponse) | GetNodeInfo queries the current node info. | GET|/cosmos/base/tendermint/v1beta1/node_info|
| `GetSyncing` | [GetSyncingRequest](#cosmos.base.tendermint.v1beta1.GetSyncingRequest) | [GetSyncingResponse](#cosmos.base.tendermint.v1beta1.GetSyncingResponse) | GetSyncing queries node syncing. | GET|/cosmos/base/tendermint/v1beta1/syncing|
| `GetLatestBlock` | [GetLatestBlockRequest](#cosmos.base.tendermint.v1beta1.GetLatestBlockRequest) | [GetLatestBlockResponse](#cosmos.base.tendermint.v1beta1.GetLatestBlockResponse) | GetLatestBlock returns the latest block. | GET|/cosmos/base/tendermint/v1beta1/blocks/latest|
| `GetBlockByHeight` | [GetBlockByHeightRequest](#cosmos.base.tendermint.v1beta1.GetBlockByHeightRequest) | [GetBlockByHeightResponse](#cosmos.base.tendermint.v1beta1.GetBlockByHeightResponse) | GetBlockByHeight queries block for given height. | GET|/cosmos/base/tendermint/v1beta1/blocks/{height}|
| `GetLatestValidatorSet` | [GetLatestValidatorSetRequest](#cosmos.base.tendermint.v1beta1.GetLatestValidatorSetRequest) | [GetLatestValidatorSetResponse](#cosmos.base.tendermint.v1beta1.GetLatestValidatorSetResponse) | GetLatestValidatorSet queries latest validator-set. | GET|/cosmos/base/tendermint/v1beta1/validatorsets/latest|
| `GetValidatorSetByHeight` | [GetValidatorSetByHeightRequest](#cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightRequest) | [GetValidatorSetByHeightResponse](#cosmos.base.tendermint.v1beta1.GetValidatorSetByHeightResponse) | GetValidatorSetByHeight queries validator-set at a given height. | GET|/cosmos/base/tendermint/v1beta1/validatorsets/{height}|

 <!-- end services -->



<a name="cosmos/capability/v1beta1/capability.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/capability/v1beta1/capability.proto



<a name="cosmos.capability.v1beta1.Capability"></a>

### Capability
能力定义了对象能力的实现。 提供给 Capability 的索引必须是全局唯一的。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `index` | [uint64](#uint64) |  |  |






<a name="cosmos.capability.v1beta1.CapabilityOwners"></a>

### CapabilityOwners
CapabilityOwners 定义了单个 Capability 的一组所有者。 所有者集必须是唯一的。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `owners` | [Owner](#cosmos.capability.v1beta1.Owner) | repeated |  |






<a name="cosmos.capability.v1beta1.Owner"></a>

### Owner
Owner 定义单个功能所有者。 所有者由能力名称和模块名称定义。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `module` | [string](#string) |  |  |
| `name` | [string](#string) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/capability/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/capability/v1beta1/genesis.proto



<a name="cosmos.capability.v1beta1.GenesisOwners"></a>

### GenesisOwners
GenesisOwners 定义了能力所有者及其相应的索引。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `index` | [uint64](#uint64) |  | index is the index of the capability owner. |
| `index_owners` | [CapabilityOwners](#cosmos.capability.v1beta1.CapabilityOwners) |  | index_owners are the owners at the given index. |






<a name="cosmos.capability.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了能力模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `index` | [uint64](#uint64) |  | index is the capability global index. |
| `owners` | [GenesisOwners](#cosmos.capability.v1beta1.GenesisOwners) | repeated | owners represents a map from index to owners of the capability index index key is string to allow amino marshalling. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crisis/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crisis/v1beta1/genesis.proto



<a name="cosmos.crisis.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了危机模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `constant_fee` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  | constant_fee is the fee used to verify the invariant in the crisis module. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crisis/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crisis/v1beta1/tx.proto



<a name="cosmos.crisis.v1beta1.MsgVerifyInvariant"></a>

### MsgVerifyInvariant
MsgVerifyInvariant 表示验证特定不变性的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `sender` | [string](#string) |  |  |
| `invariant_module_name` | [string](#string) |  |  |
| `invariant_route` | [string](#string) |  |  |






<a name="cosmos.crisis.v1beta1.MsgVerifyInvariantResponse"></a>

### MsgVerifyInvariantResponse
MsgVerifyInvariantResponse 定义了 Msg/VerifyInvariant 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.crisis.v1beta1.Msg"></a>

### Msg
Msg 定义了银行消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `VerifyInvariant` | [MsgVerifyInvariant](#cosmos.crisis.v1beta1.MsgVerifyInvariant) | [MsgVerifyInvariantResponse](#cosmos.crisis.v1beta1.MsgVerifyInvariantResponse) | VerifyInvariant defines a method to verify a particular invariance. | |

 <!-- end services -->



<a name="cosmos/crypto/ed25519/keys.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/ed25519/keys.proto



<a name="cosmos.crypto.ed25519.PrivKey"></a>

### PrivKey
已弃用:PrivKey 定义了 ed25519 私钥。
注意:ed25519 密钥不得在 SDK 应用程序中使用，除非在tendermint 验证器上下文中。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |






<a name="cosmos.crypto.ed25519.PubKey"></a>

### PubKey
PubKey 是一个 ed25519 公钥，用于处理 SDK 中的 Tendermint 密钥。
任何序列化和 SDK 兼容性都需要它。
它不能用于非 Tendermint 键上下文，因为它没有实现 ADR-28。 尽管如此，您希望在应用程序用户级别使用 ed25519，然后您必须创建一个新的 proto 消息并遵循 ADR-28 进行地址构建。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/hd/v1/hd.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/hd/v1/hd.proto



<a name="cosmos.crypto.hd.v1.BIP44Params"></a>

### BIP44Params
BIP44Params 用作记录中分类帐项中的路径字段。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `purpose` | [uint32](#uint32) |  | purpose is a constant set to 44' (or 0x8000002C) following the BIP43 recommendation |
| `coin_type` | [uint32](#uint32) |  | coin_type is a constant that improves privacy |
| `account` | [uint32](#uint32) |  | account splits the key space into independent user identities |
| `change` | [bool](#bool) |  | change is a constant used for public derivation. Constant 0 is used for external chain and constant 1 for internal chain. |
| `address_index` | [uint32](#uint32) |  | address_index is used as child index in BIP32 derivation |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/keyring/v1/record.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/keyring/v1/record.proto



<a name="cosmos.crypto.keyring.v1.Record"></a>

### Record
Record 用于表示密钥环中的密钥。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | name represents a name of Record |
| `pub_key` | [google.protobuf.Any](#google.protobuf.Any) |  | pub_key represents a public key in any format |
| `local` | [Record.Local](#cosmos.crypto.keyring.v1.Record.Local) |  | local stores the public information about a locally stored key |
| `ledger` | [Record.Ledger](#cosmos.crypto.keyring.v1.Record.Ledger) |  | ledger stores the public information about a Ledger key |
| `multi` | [Record.Multi](#cosmos.crypto.keyring.v1.Record.Multi) |  | Multi does not store any information. |
| `offline` | [Record.Offline](#cosmos.crypto.keyring.v1.Record.Offline) |  | Offline does not store any information. |






<a name="cosmos.crypto.keyring.v1.Record.Ledger"></a>

### Record.Ledger
分类帐项目


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `path` | [cosmos.crypto.hd.v1.BIP44Params](#cosmos.crypto.hd.v1.BIP44Params) |  |  |






<a name="cosmos.crypto.keyring.v1.Record.Local"></a>

### Record.Local
Item 是存储在密钥环后端的密钥环项目。
本地项目


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `priv_key` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `priv_key_type` | [string](#string) |  |  |






<a name="cosmos.crypto.keyring.v1.Record.Multi"></a>

### Record.Multi
多项目






<a name="cosmos.crypto.keyring.v1.Record.Offline"></a>

### Record.Offline
线下项目





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/multisig/keys.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/multisig/keys.proto



<a name="cosmos.crypto.multisig.LegacyAminoPubKey"></a>

### LegacyAminoPubKey
LegacyAminoPubKey 指定了一个公钥类型，它嵌套了多个公钥和一个阈值，它使用了传统的氨基地址规则。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `threshold` | [uint32](#uint32) |  |  |
| `public_keys` | [google.protobuf.Any](#google.protobuf.Any) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/multisig/v1beta1/multisig.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/multisig/v1beta1/multisig.proto



<a name="cosmos.crypto.multisig.v1beta1.CompactBitArray"></a>

### CompactBitArray
CompactBitArray 是空间高效位数组的实现。
这用于确保编码数据在 proto 编码后占用最少的空间。
这不是线程安全的，也不适用于并发使用。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `extra_bits_stored` | [uint32](#uint32) |  |  |
| `elems` | [bytes](#bytes) |  |  |






<a name="cosmos.crypto.multisig.v1beta1.MultiSignature"></a>

### MultiSignature
MultiSignature 包装来自 multisig.LegacyAminoPubKey 的签名。
请参阅 cosmos.tx.v1betata1.ModeInfo.Multi 了解如何指定哪些签名者签名以及使用哪些模式。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `signatures` | [bytes](#bytes) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/secp256k1/keys.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/secp256k1/keys.proto



<a name="cosmos.crypto.secp256k1.PrivKey"></a>

### PrivKey
PrivKey 定义了一个 secp256k1 私钥。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |






<a name="cosmos.crypto.secp256k1.PubKey"></a>

### PubKey
PubKey 定义了一个 secp256k1 公钥 Key 是公钥的压缩形式。 如果 y 坐标是与 x 坐标关联的两个字节中字典序最大的，则第一个字节取决于 0x02 字节。 否则第一个字节是 0x03。 此前缀后跟 x 坐标。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/crypto/secp256r1/keys.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/crypto/secp256r1/keys.proto
Since: cosmos-sdk 0.43


<a name="cosmos.crypto.secp256r1.PrivKey"></a>

### PrivKey
PrivKey 定义了一个 secp256r1 ECDSA 私钥。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `secret` | [bytes](#bytes) |  | secret number serialized using big-endian encoding |






<a name="cosmos.crypto.secp256r1.PubKey"></a>

### PubKey
PubKey 定义了一个 secp256r1 ECDSA 公钥。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `key` | [bytes](#bytes) |  | Point on secp256r1 curve in a compressed representation as specified in section 4.3.6 of ANSI X9.62: https://webstore.ansi.org/standards/ascx9/ansix9621998 |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/distribution/v1beta1/distribution.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/distribution/v1beta1/distribution.proto



<a name="cosmos.distribution.v1beta1.CommunityPoolSpendProposal"></a>

### CommunityPoolSpendProposal
CommunityPoolSpendProposal 详细说明了使用社区资金的建议，以及建议花费的代币数量以及接收者帐户。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |
| `recipient` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.distribution.v1beta1.CommunityPoolSpendProposalWithDeposit"></a>

### CommunityPoolSpendProposalWithDeposit
CommunityPoolSpendProposalWithDeposit 定义了一个带有存款的 CommunityPoolSpendProposal


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |
| `recipient` | [string](#string) |  |  |
| `amount` | [string](#string) |  |  |
| `deposit` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.DelegationDelegatorReward"></a>

### DelegationDelegatorReward
DelegationDelegatorReward 代表的属性
代表的委托奖励。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  |  |
| `reward` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |






<a name="cosmos.distribution.v1beta1.DelegatorStartingInfo"></a>

### DelegatorStartingInfo
DelegatorStartingInfo 表示委托人奖励期的开始信息。 它跟踪前一个验证器周期、委托的质押令牌数量和创建高度(稍后检查是否发生了任何斜线)。 注意:即使验证器被削减到整个 staking 代币，验证器中的委托人可能会剩下少于一个完整的代币，因此使用 sdk.Dec。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `previous_period` | [uint64](#uint64) |  |  |
| `stake` | [string](#string) |  |  |
| `height` | [uint64](#uint64) |  |  |






<a name="cosmos.distribution.v1beta1.FeePool"></a>

### FeePool
FeePool 是用于分发的全球费用池。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `community_pool` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |






<a name="cosmos.distribution.v1beta1.Params"></a>

### Params
参数定义分发模块的参数集。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `community_tax` | [string](#string) |  |  |
| `base_proposer_reward` | [string](#string) |  |  |
| `bonus_proposer_reward` | [string](#string) |  |  |
| `withdraw_addr_enabled` | [bool](#bool) |  |  |






<a name="cosmos.distribution.v1beta1.ValidatorAccumulatedCommission"></a>

### ValidatorAccumulatedCommission
ValidatorAccumulatedCommission 代表作为运行计数器保存的验证器的累积佣金，可以随时提取。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `commission` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |






<a name="cosmos.distribution.v1beta1.ValidatorCurrentRewards"></a>

### ValidatorCurrentRewards
ValidatorCurrentRewards 代表验证者的当前奖励和当前周期，作为运行计数器保持并在每个区块递增，只要验证者的代币保持不变。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rewards` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |
| `period` | [uint64](#uint64) |  |  |






<a name="cosmos.distribution.v1beta1.ValidatorHistoricalRewards"></a>

### ValidatorHistoricalRewards
ValidatorHistoricalRewards 代表验证者的历史奖励。 高度在存储键中是隐含的。 根据规范，累积奖励率是从第零期到此期奖励/代币的总和。 引用计数指示可能需要在任何时候引用此历史条目的对象数。
ReferenceCount = 结束相关期间的未完成授权的数量(并且可能需要读取该记录)
  + 结束相关期间的斜线数(并且可能需要读取该记录)
  + 第零时期每个验证器一个，在初始化时设置


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `cumulative_reward_ratio` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |
| `reference_count` | [uint32](#uint32) |  |  |






<a name="cosmos.distribution.v1beta1.ValidatorOutstandingRewards"></a>

### ValidatorOutstandingRewards
ValidatorOutstandingRewards 代表一个验证者的优秀(未撤回)奖励，跟踪成本低，允许简单的健全性检查。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rewards` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated |  |






<a name="cosmos.distribution.v1beta1.ValidatorSlashEvent"></a>

### ValidatorSlashEvent
ValidatorSlashEvent 表示验证器斜线事件。
高度在存储键中是隐含的。
这是计算适当数量的抵押代币所必需的
对于在出现斜线后撤回的代表团。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_period` | [uint64](#uint64) |  |  |
| `fraction` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.ValidatorSlashEvents"></a>

### ValidatorSlashEvents
ValidatorSlashEvents 是 ValidatorSlashEvent 消息的集合。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_slash_events` | [ValidatorSlashEvent](#cosmos.distribution.v1beta1.ValidatorSlashEvent) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/distribution/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/distribution/v1beta1/genesis.proto



<a name="cosmos.distribution.v1beta1.DelegatorStartingInfoRecord"></a>

### DelegatorStartingInfoRecord
DelegatorStartingInfoRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address is the address of the delegator. |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `starting_info` | [DelegatorStartingInfo](#cosmos.distribution.v1beta1.DelegatorStartingInfo) |  | starting_info defines the starting info of a delegator. |






<a name="cosmos.distribution.v1beta1.DelegatorWithdrawInfo"></a>

### DelegatorWithdrawInfo
DelegatorWithdrawInfo 是分配奖励默认提取到的地址，该结构仅在创世时用于提供默认提取地址。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address is the address of the delegator. |
| `withdraw_address` | [string](#string) |  | withdraw_address is the address to withdraw the delegation rewards to. |






<a name="cosmos.distribution.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义分发模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.distribution.v1beta1.Params) |  | params defines all the paramaters of the module. |
| `fee_pool` | [FeePool](#cosmos.distribution.v1beta1.FeePool) |  | fee_pool defines the fee pool at genesis. |
| `delegator_withdraw_infos` | [DelegatorWithdrawInfo](#cosmos.distribution.v1beta1.DelegatorWithdrawInfo) | repeated | fee_pool defines the delegator withdraw infos at genesis. |
| `previous_proposer` | [string](#string) |  | fee_pool defines the previous proposer at genesis. |
| `outstanding_rewards` | [ValidatorOutstandingRewardsRecord](#cosmos.distribution.v1beta1.ValidatorOutstandingRewardsRecord) | repeated | fee_pool defines the outstanding rewards of all validators at genesis. |
| `validator_accumulated_commissions` | [ValidatorAccumulatedCommissionRecord](#cosmos.distribution.v1beta1.ValidatorAccumulatedCommissionRecord) | repeated | fee_pool defines the accumulated commisions of all validators at genesis. |
| `validator_historical_rewards` | [ValidatorHistoricalRewardsRecord](#cosmos.distribution.v1beta1.ValidatorHistoricalRewardsRecord) | repeated | fee_pool defines the historical rewards of all validators at genesis. |
| `validator_current_rewards` | [ValidatorCurrentRewardsRecord](#cosmos.distribution.v1beta1.ValidatorCurrentRewardsRecord) | repeated | fee_pool defines the current rewards of all validators at genesis. |
| `delegator_starting_infos` | [DelegatorStartingInfoRecord](#cosmos.distribution.v1beta1.DelegatorStartingInfoRecord) | repeated | fee_pool defines the delegator starting infos at genesis. |
| `validator_slash_events` | [ValidatorSlashEventRecord](#cosmos.distribution.v1beta1.ValidatorSlashEventRecord) | repeated | fee_pool defines the validator slash events at genesis. |






<a name="cosmos.distribution.v1beta1.ValidatorAccumulatedCommissionRecord"></a>

### ValidatorAccumulatedCommissionRecord
ValidatorAccumulatedCommissionRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `accumulated` | [ValidatorAccumulatedCommission](#cosmos.distribution.v1beta1.ValidatorAccumulatedCommission) |  | accumulated is the accumulated commission of a validator. |






<a name="cosmos.distribution.v1beta1.ValidatorCurrentRewardsRecord"></a>

### ValidatorCurrentRewardsRecord
ValidatorCurrentRewardsRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `rewards` | [ValidatorCurrentRewards](#cosmos.distribution.v1beta1.ValidatorCurrentRewards) |  | rewards defines the current rewards of a validator. |






<a name="cosmos.distribution.v1beta1.ValidatorHistoricalRewardsRecord"></a>

### ValidatorHistoricalRewardsRecord
ValidatorHistoricalRewardsRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `period` | [uint64](#uint64) |  | period defines the period the historical rewards apply to. |
| `rewards` | [ValidatorHistoricalRewards](#cosmos.distribution.v1beta1.ValidatorHistoricalRewards) |  | rewards defines the historical rewards of a validator. |






<a name="cosmos.distribution.v1beta1.ValidatorOutstandingRewardsRecord"></a>

### ValidatorOutstandingRewardsRecord
ValidatorOutstandingRewardsRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `outstanding_rewards` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated | outstanding_rewards represents the oustanding rewards of a validator. |






<a name="cosmos.distribution.v1beta1.ValidatorSlashEventRecord"></a>

### ValidatorSlashEventRecord
ValidatorSlashEventRecord 用于通过 genesis json 导入/导出。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address is the address of the validator. |
| `height` | [uint64](#uint64) |  | height defines the block height at which the slash event occured. |
| `period` | [uint64](#uint64) |  | period is the period of the slash event. |
| `validator_slash_event` | [ValidatorSlashEvent](#cosmos.distribution.v1beta1.ValidatorSlashEvent) |  | validator_slash_event describes the slash event. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/distribution/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/distribution/v1beta1/query.proto



<a name="cosmos.distribution.v1beta1.QueryCommunityPoolRequest"></a>

### QueryCommunityPoolRequest
QueryCommunityPoolRequest 是 Query/CommunityPool RPC 方法的请求类型。






<a name="cosmos.distribution.v1beta1.QueryCommunityPoolResponse"></a>

### QueryCommunityPoolResponse
QueryCommunityPoolResponse 是 Query/CommunityPool RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pool` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated | pool defines community pool's coins. |






<a name="cosmos.distribution.v1beta1.QueryDelegationRewardsRequest"></a>

### QueryDelegationRewardsRequest
QueryDelegationRewardsRequest 是 Query/DelegationRewards RPC 方法的请求类型。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address defines the delegator address to query for. |
| `validator_address` | [string](#string) |  | validator_address defines the validator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryDelegationRewardsResponse"></a>

### QueryDelegationRewardsResponse
QueryDelegationRewardsResponse 是 Query/DelegationRewards RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rewards` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated | rewards defines the rewards accrued by a delegation. |






<a name="cosmos.distribution.v1beta1.QueryDelegationTotalRewardsRequest"></a>

### QueryDelegationTotalRewardsRequest
QueryDelegationTotalRewardsRequest 是请求类型
Query/DelegationTotalRewards RPC 方法。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address defines the delegator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryDelegationTotalRewardsResponse"></a>

### QueryDelegationTotalRewardsResponse
QueryDelegationTotalRewardsResponse 是响应类型
Query/DelegationTotalRewards RPC 方法。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rewards` | [DelegationDelegatorReward](#cosmos.distribution.v1beta1.DelegationDelegatorReward) | repeated | rewards defines all the rewards accrued by a delegator. |
| `total` | [cosmos.base.v1beta1.DecCoin](#cosmos.base.v1beta1.DecCoin) | repeated | total defines the sum of all the rewards. |






<a name="cosmos.distribution.v1beta1.QueryDelegatorValidatorsRequest"></a>

### QueryDelegatorValidatorsRequest
QueryDelegatorValidatorsRequest 是 Query/DelegatorValidators RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address defines the delegator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryDelegatorValidatorsResponse"></a>

### QueryDelegatorValidatorsResponse
QueryDelegatorValidatorsResponse 是 Query/DelegatorValidators RPC 方法的响应类型。

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validators` | [string](#string) | repeated | validators defines the validators a delegator is delegating for. |






<a name="cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressRequest"></a>

### QueryDelegatorWithdrawAddressRequest
QueryDelegatorWithdrawAddressRequest 是 Query/DelegatorWithdrawAddress RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address defines the delegator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressResponse"></a>

### QueryDelegatorWithdrawAddressResponse
QueryDelegatorWithdrawAddressResponse 是 Query/DelegatorWithdrawAddress RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `withdraw_address` | [string](#string) |  | withdraw_address defines the delegator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。






<a name="cosmos.distribution.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.distribution.v1beta1.Params) |  | params defines the parameters of the module. |






<a name="cosmos.distribution.v1beta1.QueryValidatorCommissionRequest"></a>

### QueryValidatorCommissionRequest
QueryValidatorCommissionRequest 是 Query/ValidatorCommission RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address defines the validator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryValidatorCommissionResponse"></a>

### QueryValidatorCommissionResponse
QueryValidatorCommissionResponse 是 Query/ValidatorCommission RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `commission` | [ValidatorAccumulatedCommission](#cosmos.distribution.v1beta1.ValidatorAccumulatedCommission) |  | commission defines the commision the validator received. |






<a name="cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsRequest"></a>

### QueryValidatorOutstandingRewardsRequest
QueryValidatorCommissionResponse 是Query/ValidatorCommission RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address defines the validator address to query for. |






<a name="cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsResponse"></a>

### QueryValidatorOutstandingRewardsResponse
QueryValidatorOutstandingRewardsResponse 是 Query/ValidatorOutstandingRewards RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rewards` | [ValidatorOutstandingRewards](#cosmos.distribution.v1beta1.ValidatorOutstandingRewards) |  |  |






<a name="cosmos.distribution.v1beta1.QueryValidatorSlashesRequest"></a>

### QueryValidatorSlashesRequest
QueryValidatorSlashesRequest 是请求类型
Query/ValidatorSlashes RPC 方法


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  | validator_address defines the validator address to query for. |
| `starting_height` | [uint64](#uint64) |  | starting_height defines the optional starting height to query the slashes. |
| `ending_height` | [uint64](#uint64) |  | starting_height defines the optional ending height to query the slashes. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.distribution.v1beta1.QueryValidatorSlashesResponse"></a>

### QueryValidatorSlashesResponse
QueryValidatorSlashesResponse 是响应类型
Query/ValidatorSlashes RPC 方法。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `slashes` | [ValidatorSlashEvent](#cosmos.distribution.v1beta1.ValidatorSlashEvent) | repeated | slashes defines the slashes the validator received. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.distribution.v1beta1.Query"></a>

### Query
Query 定义了分发模块的 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Params` | [QueryParamsRequest](#cosmos.distribution.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.distribution.v1beta1.QueryParamsResponse) | Params queries params of the distribution module. | GET|/cosmos/distribution/v1beta1/params|
| `ValidatorOutstandingRewards` | [QueryValidatorOutstandingRewardsRequest](#cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsRequest) | [QueryValidatorOutstandingRewardsResponse](#cosmos.distribution.v1beta1.QueryValidatorOutstandingRewardsResponse) | ValidatorOutstandingRewards queries rewards of a validator address. | GET|/cosmos/distribution/v1beta1/validators/{validator_address}/outstanding_rewards|
| `ValidatorCommission` | [QueryValidatorCommissionRequest](#cosmos.distribution.v1beta1.QueryValidatorCommissionRequest) | [QueryValidatorCommissionResponse](#cosmos.distribution.v1beta1.QueryValidatorCommissionResponse) | ValidatorCommission queries accumulated commission for a validator. | GET|/cosmos/distribution/v1beta1/validators/{validator_address}/commission|
| `ValidatorSlashes` | [QueryValidatorSlashesRequest](#cosmos.distribution.v1beta1.QueryValidatorSlashesRequest) | [QueryValidatorSlashesResponse](#cosmos.distribution.v1beta1.QueryValidatorSlashesResponse) | ValidatorSlashes queries slash events of a validator. | GET|/cosmos/distribution/v1beta1/validators/{validator_address}/slashes|
| `DelegationRewards` | [QueryDelegationRewardsRequest](#cosmos.distribution.v1beta1.QueryDelegationRewardsRequest) | [QueryDelegationRewardsResponse](#cosmos.distribution.v1beta1.QueryDelegationRewardsResponse) | DelegationRewards queries the total rewards accrued by a delegation. | GET|/cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards/{validator_address}|
| `DelegationTotalRewards` | [QueryDelegationTotalRewardsRequest](#cosmos.distribution.v1beta1.QueryDelegationTotalRewardsRequest) | [QueryDelegationTotalRewardsResponse](#cosmos.distribution.v1beta1.QueryDelegationTotalRewardsResponse) | DelegationTotalRewards queries the total rewards accrued by a each validator. | GET|/cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards|
| `DelegatorValidators` | [QueryDelegatorValidatorsRequest](#cosmos.distribution.v1beta1.QueryDelegatorValidatorsRequest) | [QueryDelegatorValidatorsResponse](#cosmos.distribution.v1beta1.QueryDelegatorValidatorsResponse) | DelegatorValidators queries the validators of a delegator. | GET|/cosmos/distribution/v1beta1/delegators/{delegator_address}/validators|
| `DelegatorWithdrawAddress` | [QueryDelegatorWithdrawAddressRequest](#cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressRequest) | [QueryDelegatorWithdrawAddressResponse](#cosmos.distribution.v1beta1.QueryDelegatorWithdrawAddressResponse) | DelegatorWithdrawAddress queries withdraw address of a delegator. | GET|/cosmos/distribution/v1beta1/delegators/{delegator_address}/withdraw_address|
| `CommunityPool` | [QueryCommunityPoolRequest](#cosmos.distribution.v1beta1.QueryCommunityPoolRequest) | [QueryCommunityPoolResponse](#cosmos.distribution.v1beta1.QueryCommunityPoolResponse) | CommunityPool queries the community pool coins. | GET|/cosmos/distribution/v1beta1/community_pool|

 <!-- end services -->



<a name="cosmos/distribution/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/distribution/v1beta1/tx.proto



<a name="cosmos.distribution.v1beta1.MsgFundCommunityPool"></a>

### MsgFundCommunityPool
MsgFundCommunityPool 允许账户直接资助社区池。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `depositor` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.MsgFundCommunityPoolResponse"></a>

### MsgFundCommunityPoolResponse
MsgFundCommunityPoolResponse 定义了 Msg/FundCommunityPool 响应类型。






<a name="cosmos.distribution.v1beta1.MsgSetWithdrawAddress"></a>

### MsgSetWithdrawAddress
MsgSetWithdrawAddress 设置提款地址委托人(或验证人自我委托)。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `withdraw_address` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.MsgSetWithdrawAddressResponse"></a>

### MsgSetWithdrawAddressResponse
Msg tWithdrawAddress Response 定义了 Message/SetWithdrawAddress 响应类型。






<a name="cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward"></a>

### MsgWithdrawDelegatorReward
MsgWithdrawDelegatorReward 代表从单个验证器向委托人撤回委托。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_address` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.MsgWithdrawDelegatorRewardResponse"></a>

### MsgWithdrawDelegatorRewardResponse
MsgWithdrawDelegatorRewardResponse 定义了 Msg/WithdrawDelegatorReward 响应类型。






<a name="cosmos.distribution.v1beta1.MsgWithdrawValidatorCommission"></a>

### MsgWithdrawValidatorCommission
MsgWithdrawValidatorCommission 将全部佣金提取到验证者地址。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_address` | [string](#string) |  |  |






<a name="cosmos.distribution.v1beta1.MsgWithdrawValidatorCommissionResponse"></a>

### MsgWithdrawValidatorCommissionResponse
MsgWithdrawValidatorCommissionResponse 定义了 Msg/WithdrawValidatorCommission 响应类型。




 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.distribution.v1beta1.Msg"></a>

### Msg
Msg 定义了分发Msg 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `SetWithdrawAddress` | [MsgSetWithdrawAddress](#cosmos.distribution.v1beta1.MsgSetWithdrawAddress) | [MsgSetWithdrawAddressResponse](#cosmos.distribution.v1beta1.MsgSetWithdrawAddressResponse) | SetWithdrawAddress defines a method to change the withdraw address for a delegator (or validator self-delegation). | |
| `WithdrawDelegatorReward` | [MsgWithdrawDelegatorReward](#cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward) | [MsgWithdrawDelegatorRewardResponse](#cosmos.distribution.v1beta1.MsgWithdrawDelegatorRewardResponse) | WithdrawDelegatorReward defines a method to withdraw rewards of delegator from a single validator. | |
| `WithdrawValidatorCommission` | [MsgWithdrawValidatorCommission](#cosmos.distribution.v1beta1.MsgWithdrawValidatorCommission) | [MsgWithdrawValidatorCommissionResponse](#cosmos.distribution.v1beta1.MsgWithdrawValidatorCommissionResponse) | WithdrawValidatorCommission defines a method to withdraw the full commission to the validator address. | |
| `FundCommunityPool` | [MsgFundCommunityPool](#cosmos.distribution.v1beta1.MsgFundCommunityPool) | [MsgFundCommunityPoolResponse](#cosmos.distribution.v1beta1.MsgFundCommunityPoolResponse) | FundCommunityPool defines a method to allow an account to directly fund the community pool. | |

 <!-- end services -->



<a name="cosmos/evidence/v1beta1/evidence.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/evidence/v1beta1/evidence.proto



<a name="cosmos.evidence.v1beta1.Equivocation"></a>

### Equivocation
Equivocation 实现了 Evidence 接口并定义了双重签名不当行为的证据。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  |  |
| `time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `power` | [int64](#int64) |  |  |
| `consensus_address` | [string](#string) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/evidence/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/evidence/v1beta1/genesis.proto



<a name="cosmos.evidence.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了证据模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `evidence` | [google.protobuf.Any](#google.protobuf.Any) | repeated | evidence defines all the evidence at genesis. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/evidence/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/evidence/v1beta1/query.proto



<a name="cosmos.evidence.v1beta1.QueryAllEvidenceRequest"></a>

### QueryAllEvidenceRequest
QueryEvidenceRequest 是 Query/AllEvidence RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.evidence.v1beta1.QueryAllEvidenceResponse"></a>

### QueryAllEvidenceResponse
QueryAllEvidenceResponse 是 Query/AllEvidence RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `evidence` | [google.protobuf.Any](#google.protobuf.Any) | repeated | evidence returns all evidences. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.evidence.v1beta1.QueryEvidenceRequest"></a>

### QueryEvidenceRequest
QueryEvidenceRequest 是 Query/Evidence RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `evidence_hash` | [bytes](#bytes) |  | evidence_hash defines the hash of the requested evidence. |






<a name="cosmos.evidence.v1beta1.QueryEvidenceResponse"></a>

### QueryEvidenceResponse
QueryEvidenceResponse 是 Query/Evidence RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `evidence` | [google.protobuf.Any](#google.protobuf.Any) |  | evidence returns the requested evidence. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.evidence.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Evidence` | [QueryEvidenceRequest](#cosmos.evidence.v1beta1.QueryEvidenceRequest) | [QueryEvidenceResponse](#cosmos.evidence.v1beta1.QueryEvidenceResponse) | Evidence queries evidence based on evidence hash. | GET|/cosmos/evidence/v1beta1/evidence/{evidence_hash}|
| `AllEvidence` | [QueryAllEvidenceRequest](#cosmos.evidence.v1beta1.QueryAllEvidenceRequest) | [QueryAllEvidenceResponse](#cosmos.evidence.v1beta1.QueryAllEvidenceResponse) | AllEvidence queries all evidence. | GET|/cosmos/evidence/v1beta1/evidence|

 <!-- end services -->



<a name="cosmos/evidence/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/evidence/v1beta1/tx.proto



<a name="cosmos.evidence.v1beta1.MsgSubmitEvidence"></a>

### MsgSubmitEvidence
MsgSubmitEvidence 表示支持提交任意不当行为证据的消息，例如模棱两可或反事实签名。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `submitter` | [string](#string) |  |  |
| `evidence` | [google.protobuf.Any](#google.protobuf.Any) |  |  |






<a name="cosmos.evidence.v1beta1.MsgSubmitEvidenceResponse"></a>

### MsgSubmitEvidenceResponse
MsgSubmitEvidenceResponse 定义了 Msg/SubmitEvidence 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `hash` | [bytes](#bytes) |  | hash defines the hash of the evidence. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.evidence.v1beta1.Msg"></a>

### Msg
Msg 定义了证据消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `SubmitEvidence` | [MsgSubmitEvidence](#cosmos.evidence.v1beta1.MsgSubmitEvidence) | [MsgSubmitEvidenceResponse](#cosmos.evidence.v1beta1.MsgSubmitEvidenceResponse) | SubmitEvidence submits an arbitrary Evidence of misbehavior such as equivocation or counterfactual signing. | |

 <!-- end services -->



<a name="cosmos/feegrant/v1beta1/feegrant.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/feegrant/v1beta1/feegrant.proto
Since: cosmos-sdk 0.43


<a name="cosmos.feegrant.v1beta1.AllowedMsgAllowance"></a>

### AllowedMsgAllowance
AllowedMsgAllowance 仅为指定的消息类型创建允许。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `allowance` | [google.protobuf.Any](#google.protobuf.Any) |  | allowance can be any of basic and filtered fee allowance. |
| `allowed_messages` | [string](#string) | repeated | allowed_messages are the messages for which the grantee has the access. |






<a name="cosmos.feegrant.v1beta1.BasicAllowance"></a>

### BasicAllowance
BasicAllowance 通过一次性授予可选择过期的令牌来实现 Allowance。受助人最多可以使用 SpendLimit 来支付费用。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `spend_limit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | spend_limit specifies the maximum amount of tokens that can be spent by this allowance and will be updated as tokens are spent. If it is empty, there is no spend limit and any amount of coins can be spent. |
| `expiration` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | expiration specifies an optional time when this allowance expires |






<a name="cosmos.feegrant.v1beta1.Grant"></a>

### Grant
授权存储在 KVStore 中以记录具有完整上下文的授权


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  | granter is the address of the user granting an allowance of their funds. |
| `grantee` | [string](#string) |  | grantee is the address of the user being granted an allowance of another user's funds. |
| `allowance` | [google.protobuf.Any](#google.protobuf.Any) |  | allowance can be any of basic and filtered fee allowance. |






<a name="cosmos.feegrant.v1beta1.PeriodicAllowance"></a>

### PeriodicAllowance
PeriodicAllowance 扩展了 Allowance 以允许最大上限和每个时间段的限制。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `basic` | [BasicAllowance](#cosmos.feegrant.v1beta1.BasicAllowance) |  | basic specifies a struct of `BasicAllowance` |
| `period` | [google.protobuf.Duration](#google.protobuf.Duration) |  | period specifies the time duration in which period_spend_limit coins can be spent before that allowance is reset |
| `period_spend_limit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | period_spend_limit specifies the maximum number of coins that can be spent in the period |
| `period_can_spend` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | period_can_spend is the number of coins left to be spent before the period_reset time |
| `period_reset` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | period_reset is the time at which this period resets and a new one begins, it is calculated from the start time of the first transaction after the last period ended |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/feegrant/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/feegrant/v1beta1/genesis.proto
Since: cosmos-sdk 0.43


<a name="cosmos.feegrant.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 包含一组费用津贴，从商店中保留


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `allowances` | [Grant](#cosmos.feegrant.v1beta1.Grant) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/feegrant/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/feegrant/v1beta1/query.proto
Since: cosmos-sdk 0.43


<a name="cosmos.feegrant.v1beta1.QueryAllowanceRequest"></a>

### QueryAllowanceRequest
QueryAllowanceRequest 是 Query/Allowance RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  | granter is the address of the user granting an allowance of their funds. |
| `grantee` | [string](#string) |  | grantee is the address of the user being granted an allowance of another user's funds. |






<a name="cosmos.feegrant.v1beta1.QueryAllowanceResponse"></a>

### QueryAllowanceResponse
QueryAllowanceResponse 是 Query/Allowance RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `allowance` | [Grant](#cosmos.feegrant.v1beta1.Grant) |  | allowance is a allowance granted for grantee by granter. |






<a name="cosmos.feegrant.v1beta1.QueryAllowancesRequest"></a>

### QueryAllowancesRequest
QueryAllowancesRequest 是 Query/Allowances RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `grantee` | [string](#string) |  |  |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |






<a name="cosmos.feegrant.v1beta1.QueryAllowancesResponse"></a>

### QueryAllowancesResponse
QueryAllowancesResponse 是 Query/Allowances RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `allowances` | [Grant](#cosmos.feegrant.v1beta1.Grant) | repeated | allowances are allowance's granted for grantee by granter. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.feegrant.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Allowance` | [QueryAllowanceRequest](#cosmos.feegrant.v1beta1.QueryAllowanceRequest) | [QueryAllowanceResponse](#cosmos.feegrant.v1beta1.QueryAllowanceResponse) | Allowance returns fee granted to the grantee by the granter. | GET|/cosmos/feegrant/v1beta1/allowance/{granter}/{grantee}|
| `Allowances` | [QueryAllowancesRequest](#cosmos.feegrant.v1beta1.QueryAllowancesRequest) | [QueryAllowancesResponse](#cosmos.feegrant.v1beta1.QueryAllowancesResponse) | Allowances returns all the grants for address. | GET|/cosmos/feegrant/v1beta1/allowances/{grantee}|

 <!-- end services -->



<a name="cosmos/feegrant/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/feegrant/v1beta1/tx.proto
Since: cosmos-sdk 0.43


<a name="cosmos.feegrant.v1beta1.MsgGrantAllowance"></a>

### MsgGrantAllowance
MsgGrantAllowance 添加了允许受赠人从受赠人的帐户中花费最多津贴的费用。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  | granter is the address of the user granting an allowance of their funds. |
| `grantee` | [string](#string) |  | grantee is the address of the user being granted an allowance of another user's funds. |
| `allowance` | [google.protobuf.Any](#google.protobuf.Any) |  | allowance can be any of basic and filtered fee allowance. |






<a name="cosmos.feegrant.v1beta1.MsgGrantAllowanceResponse"></a>

### MsgGrantAllowanceResponse
MsgGrantAllowanceResponse 定义了 Msg/GrantAllowanceResponse 响应类型。






<a name="cosmos.feegrant.v1beta1.MsgRevokeAllowance"></a>

### MsgRevokeAllowance
MsgRevokeAllowance 删除从授予者到被授予者的任何现有津贴。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `granter` | [string](#string) |  | granter is the address of the user granting an allowance of their funds. |
| `grantee` | [string](#string) |  | grantee is the address of the user being granted an allowance of another user's funds. |






<a name="cosmos.feegrant.v1beta1.MsgRevokeAllowanceResponse"></a>

### MsgRevokeAllowanceResponse
MsgRevokeAllowanceResponse 定义了 Msg/RevokeAllowanceResponse 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.feegrant.v1beta1.Msg"></a>

### Msg
Msg 定义了feegrant msg 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `GrantAllowance` | [MsgGrantAllowance](#cosmos.feegrant.v1beta1.MsgGrantAllowance) | [MsgGrantAllowanceResponse](#cosmos.feegrant.v1beta1.MsgGrantAllowanceResponse) | GrantAllowance grants fee allowance to the grantee on the granter's account with the provided expiration time. | |
| `RevokeAllowance` | [MsgRevokeAllowance](#cosmos.feegrant.v1beta1.MsgRevokeAllowance) | [MsgRevokeAllowanceResponse](#cosmos.feegrant.v1beta1.MsgRevokeAllowanceResponse) | RevokeAllowance revokes any fee allowance of granter's account that has been granted to the grantee. | |

 <!-- end services -->



<a name="cosmos/genutil/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/genutil/v1beta1/genesis.proto



<a name="cosmos.genutil.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 JSON 中的原始创世交易。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `gen_txs` | [bytes](#bytes) | repeated | gen_txs defines the genesis transactions. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/gov/v1beta1/gov.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta1/gov.proto



<a name="cosmos.gov.v1beta1.Deposit"></a>

### Deposit
存款定义了帐户地址存入的活跃提案的金额。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `depositor` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.gov.v1beta1.DepositParams"></a>

### DepositParams
DepositParams 定义了治理提案中存款的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `min_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | Minimum deposit for a proposal to enter voting period. |
| `max_deposit_period` | [google.protobuf.Duration](#google.protobuf.Duration) |  | Maximum period for Atom holders to deposit on a proposal. Initial value: 2 months. |






<a name="cosmos.gov.v1beta1.Proposal"></a>

### Proposal
提案定义了治理提案的核心字段成员。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `content` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `status` | [ProposalStatus](#cosmos.gov.v1beta1.ProposalStatus) |  |  |
| `final_tally_result` | [TallyResult](#cosmos.gov.v1beta1.TallyResult) |  |  |
| `submit_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `deposit_end_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `total_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `voting_start_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `voting_end_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |






<a name="cosmos.gov.v1beta1.TallyParams"></a>

### TallyParams
TallyParams 定义了用于统计治理提案投票的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `quorum` | [bytes](#bytes) |  | Minimum percentage of total stake needed to vote for a result to be considered valid. |
| `threshold` | [bytes](#bytes) |  | Minimum proportion of Yes votes for proposal to pass. Default value: 0.5. |
| `veto_threshold` | [bytes](#bytes) |  | Minimum value of Veto votes to Total votes ratio for proposal to be vetoed. Default value: 1/3. |






<a name="cosmos.gov.v1beta1.TallyResult"></a>

### TallyResult
TallyResult 定义了治理提案的标准计数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `yes` | [string](#string) |  |  |
| `abstain` | [string](#string) |  |  |
| `no` | [string](#string) |  |  |
| `no_with_veto` | [string](#string) |  |  |






<a name="cosmos.gov.v1beta1.TextProposal"></a>

### TextProposal
TextProposal 定义了一个标准的文本提案，其更改需要在批准的情况下手动更新。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |






<a name="cosmos.gov.v1beta1.Vote"></a>

### Vote
投票定义了对治理提案的投票。
投票由提案 ID、投票者和投票选项组成。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `option` | [VoteOption](#cosmos.gov.v1beta1.VoteOption) |  | **Deprecated.** Deprecated: Prefer to use `options` instead. This field is set in queries if and only if `len(options) == 1` and that option has weight 1. In all other cases, this field will default to VOTE_OPTION_UNSPECIFIED. |
| `options` | [WeightedVoteOption](#cosmos.gov.v1beta1.WeightedVoteOption) | repeated | Since: cosmos-sdk 0.43 |






<a name="cosmos.gov.v1beta1.VotingParams"></a>

### VotingParams
VotingParams 定义了对治理提案进行投票的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `voting_period` | [google.protobuf.Duration](#google.protobuf.Duration) |  | Length of the voting period. |






<a name="cosmos.gov.v1beta1.WeightedVoteOption"></a>

### WeightedVoteOption
WeightedVoteOption 定义了投票分裂的投票单位。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `option` | [VoteOption](#cosmos.gov.v1beta1.VoteOption) |  |  |
| `weight` | [string](#string) |  |  |





 <!-- end messages -->


<a name="cosmos.gov.v1beta1.ProposalStatus"></a>

### ProposalStatus
ProposalStatus 枚举提案的有效状态。

| Name | Number | Description |
| ---- | ------ | ----------- |
| PROPOSAL_STATUS_UNSPECIFIED | 0 | PROPOSAL_STATUS_UNSPECIFIED defines the default propopsal status. |
| PROPOSAL_STATUS_DEPOSIT_PERIOD | 1 | PROPOSAL_STATUS_DEPOSIT_PERIOD defines a proposal status during the deposit period. |
| PROPOSAL_STATUS_VOTING_PERIOD | 2 | PROPOSAL_STATUS_VOTING_PERIOD defines a proposal status during the voting period. |
| PROPOSAL_STATUS_PASSED | 3 | PROPOSAL_STATUS_PASSED defines a proposal status of a proposal that has passed. |
| PROPOSAL_STATUS_REJECTED | 4 | PROPOSAL_STATUS_REJECTED defines a proposal status of a proposal that has been rejected. |
| PROPOSAL_STATUS_FAILED | 5 | PROPOSAL_STATUS_FAILED defines a proposal status of a proposal that has failed. |



<a name="cosmos.gov.v1beta1.VoteOption"></a>

### VoteOption
VoteOption 枚举给定治理提案的有效投票选项。

| Name | Number | Description |
| ---- | ------ | ----------- |
| VOTE_OPTION_UNSPECIFIED | 0 | VOTE_OPTION_UNSPECIFIED defines a no-op vote option. |
| VOTE_OPTION_YES | 1 | VOTE_OPTION_YES defines a yes vote option. |
| VOTE_OPTION_ABSTAIN | 2 | VOTE_OPTION_ABSTAIN defines an abstain vote option. |
| VOTE_OPTION_NO | 3 | VOTE_OPTION_NO defines a no vote option. |
| VOTE_OPTION_NO_WITH_VETO | 4 | VOTE_OPTION_NO_WITH_VETO defines a no with veto vote option. |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/gov/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta1/genesis.proto



<a name="cosmos.gov.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 gov 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `starting_proposal_id` | [uint64](#uint64) |  | starting_proposal_id is the ID of the starting proposal. |
| `deposits` | [Deposit](#cosmos.gov.v1beta1.Deposit) | repeated | deposits defines all the deposits present at genesis. |
| `votes` | [Vote](#cosmos.gov.v1beta1.Vote) | repeated | votes defines all the votes present at genesis. |
| `proposals` | [Proposal](#cosmos.gov.v1beta1.Proposal) | repeated | proposals defines all the proposals present at genesis. |
| `deposit_params` | [DepositParams](#cosmos.gov.v1beta1.DepositParams) |  | params defines all the paramaters of related to deposit. |
| `voting_params` | [VotingParams](#cosmos.gov.v1beta1.VotingParams) |  | params defines all the paramaters of related to voting. |
| `tally_params` | [TallyParams](#cosmos.gov.v1beta1.TallyParams) |  | params defines all the paramaters of related to tally. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/gov/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta1/query.proto



<a name="cosmos.gov.v1beta1.QueryDepositRequest"></a>

### QueryDepositRequest
QueryDepositRequest 是 Query/Deposit RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `depositor` | [string](#string) |  | depositor defines the deposit addresses from the proposals. |






<a name="cosmos.gov.v1beta1.QueryDepositResponse"></a>

### QueryDepositResponse
QueryDepositResponse 是 Query/Deposit RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `deposit` | [Deposit](#cosmos.gov.v1beta1.Deposit) |  | deposit defines the requested deposit. |






<a name="cosmos.gov.v1beta1.QueryDepositsRequest"></a>

### QueryDepositsRequest
QueryDepositsRequest 是 Query/Deposits RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta1.QueryDepositsResponse"></a>

### QueryDepositsResponse
QueryDepositsResponse 是 Query/Deposits RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `deposits` | [Deposit](#cosmos.gov.v1beta1.Deposit) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.gov.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params_type` | [string](#string) |  | params_type defines which parameters to query for, can be one of "voting", "tallying" or "deposit". |






<a name="cosmos.gov.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `voting_params` | [VotingParams](#cosmos.gov.v1beta1.VotingParams) |  | voting_params defines the parameters related to voting. |
| `deposit_params` | [DepositParams](#cosmos.gov.v1beta1.DepositParams) |  | deposit_params defines the parameters related to deposit. |
| `tally_params` | [TallyParams](#cosmos.gov.v1beta1.TallyParams) |  | tally_params defines the parameters related to tally. |






<a name="cosmos.gov.v1beta1.QueryProposalRequest"></a>

### QueryProposalRequest
QueryProposalRequest 是 Query/Proposal RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |






<a name="cosmos.gov.v1beta1.QueryProposalResponse"></a>

### QueryProposalResponse
QueryProposalResponse 是 Query/Proposal RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal` | [Proposal](#cosmos.gov.v1beta1.Proposal) |  |  |






<a name="cosmos.gov.v1beta1.QueryProposalsRequest"></a>

### QueryProposalsRequest
QueryProposalsRequest 是 Query/Proposals RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_status` | [ProposalStatus](#cosmos.gov.v1beta1.ProposalStatus) |  | proposal_status defines the status of the proposals. |
| `voter` | [string](#string) |  | voter defines the voter address for the proposals. |
| `depositor` | [string](#string) |  | depositor defines the deposit addresses from the proposals. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta1.QueryProposalsResponse"></a>

### QueryProposalsResponse
QueryProposalsResponse 是 Query/Proposals RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposals` | [Proposal](#cosmos.gov.v1beta1.Proposal) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.gov.v1beta1.QueryTallyResultRequest"></a>

### QueryTallyResultRequest
QueryTallyResultRequest 是 Query/Tally RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |






<a name="cosmos.gov.v1beta1.QueryTallyResultResponse"></a>

### QueryTallyResultResponse
QueryTallyResultResponse 是 Query/Tally RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tally` | [TallyResult](#cosmos.gov.v1beta1.TallyResult) |  | tally defines the requested tally. |






<a name="cosmos.gov.v1beta1.QueryVoteRequest"></a>

### QueryVoteRequest
QueryVoteRequest 是 Query/Vote RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `voter` | [string](#string) |  | voter defines the oter address for the proposals. |






<a name="cosmos.gov.v1beta1.QueryVoteResponse"></a>

### QueryVoteResponse
QueryVoteResponse 是 Query/Vote RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `vote` | [Vote](#cosmos.gov.v1beta1.Vote) |  | vote defined the queried vote. |






<a name="cosmos.gov.v1beta1.QueryVotesRequest"></a>

### QueryVotesRequest
QueryVotesRequest 是 Query/Votes RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta1.QueryVotesResponse"></a>

### QueryVotesResponse
QueryVotesResponse 是 Query/Votes RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `votes` | [Vote](#cosmos.gov.v1beta1.Vote) | repeated | votes defined the queried votes. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.gov.v1beta1.Query"></a>

### Query
Query 定义了 gov 模块的 gRPC 查询器服务

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Proposal` | [QueryProposalRequest](#cosmos.gov.v1beta1.QueryProposalRequest) | [QueryProposalResponse](#cosmos.gov.v1beta1.QueryProposalResponse) | Proposal queries proposal details based on ProposalID. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}|
| `Proposals` | [QueryProposalsRequest](#cosmos.gov.v1beta1.QueryProposalsRequest) | [QueryProposalsResponse](#cosmos.gov.v1beta1.QueryProposalsResponse) | Proposals queries all proposals based on given status. | GET|/cosmos/gov/v1beta1/proposals|
| `Vote` | [QueryVoteRequest](#cosmos.gov.v1beta1.QueryVoteRequest) | [QueryVoteResponse](#cosmos.gov.v1beta1.QueryVoteResponse) | Vote queries voted information based on proposalID, voterAddr. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}/votes/{voter}|
| `Votes` | [QueryVotesRequest](#cosmos.gov.v1beta1.QueryVotesRequest) | [QueryVotesResponse](#cosmos.gov.v1beta1.QueryVotesResponse) | Votes queries votes of a given proposal. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}/votes|
| `Params` | [QueryParamsRequest](#cosmos.gov.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.gov.v1beta1.QueryParamsResponse) | Params queries all parameters of the gov module. | GET|/cosmos/gov/v1beta1/params/{params_type}|
| `Deposit` | [QueryDepositRequest](#cosmos.gov.v1beta1.QueryDepositRequest) | [QueryDepositResponse](#cosmos.gov.v1beta1.QueryDepositResponse) | Deposit queries single deposit information based proposalID, depositAddr. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}/deposits/{depositor}|
| `Deposits` | [QueryDepositsRequest](#cosmos.gov.v1beta1.QueryDepositsRequest) | [QueryDepositsResponse](#cosmos.gov.v1beta1.QueryDepositsResponse) | Deposits queries all deposits of a single proposal. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}/deposits|
| `TallyResult` | [QueryTallyResultRequest](#cosmos.gov.v1beta1.QueryTallyResultRequest) | [QueryTallyResultResponse](#cosmos.gov.v1beta1.QueryTallyResultResponse) | TallyResult queries the tally of a proposal vote. | GET|/cosmos/gov/v1beta1/proposals/{proposal_id}/tally|

 <!-- end services -->



<a name="cosmos/gov/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta1/tx.proto



<a name="cosmos.gov.v1beta1.MsgDeposit"></a>

### MsgDeposit
MsgDeposit 定义了向现有提案提交存款的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `depositor` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.gov.v1beta1.MsgDepositResponse"></a>

### MsgDepositResponse
MsgDepositResponse 定义了 Msg/Deposit 响应类型。






<a name="cosmos.gov.v1beta1.MsgSubmitProposal"></a>

### MsgSubmitProposal
MsgSubmitProposal 定义了一个 sdk.Msg 类型，支持提交任意提案内容。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `content` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `initial_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `proposer` | [string](#string) |  |  |






<a name="cosmos.gov.v1beta1.MsgSubmitProposalResponse"></a>

### MsgSubmitProposalResponse
MsgSubmitProposalResponse 定义了 Msg/SubmitProposal 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |






<a name="cosmos.gov.v1beta1.MsgVote"></a>

### MsgVote
MsgVote 定义了用于投票的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `option` | [VoteOption](#cosmos.gov.v1beta1.VoteOption) |  |  |






<a name="cosmos.gov.v1beta1.MsgVoteResponse"></a>

### MsgVoteResponse
MsgVoteResponse 定义了 Msg/Vote 响应类型。






<a name="cosmos.gov.v1beta1.MsgVoteWeighted"></a>

### MsgVoteWeighted
MsgVoteWeighted 定义了用于投票的消息。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `options` | [WeightedVoteOption](#cosmos.gov.v1beta1.WeightedVoteOption) | repeated |  |






<a name="cosmos.gov.v1beta1.MsgVoteWeightedResponse"></a>

### MsgVoteWeightedResponse
MsgVoteWeightedResponse 定义了 Msg/VoteWeighted 响应类型。

Since: cosmos-sdk 0.43





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.gov.v1beta1.Msg"></a>

### Msg
Msg 定义了银行消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `SubmitProposal` | [MsgSubmitProposal](#cosmos.gov.v1beta1.MsgSubmitProposal) | [MsgSubmitProposalResponse](#cosmos.gov.v1beta1.MsgSubmitProposalResponse) | SubmitProposal defines a method to create new proposal given a content. | |
| `Vote` | [MsgVote](#cosmos.gov.v1beta1.MsgVote) | [MsgVoteResponse](#cosmos.gov.v1beta1.MsgVoteResponse) | Vote defines a method to add a vote on a specific proposal. | |
| `VoteWeighted` | [MsgVoteWeighted](#cosmos.gov.v1beta1.MsgVoteWeighted) | [MsgVoteWeightedResponse](#cosmos.gov.v1beta1.MsgVoteWeightedResponse) | VoteWeighted defines a method to add a weighted vote on a specific proposal.

Since: cosmos-sdk 0.43 | |
| `Deposit` | [MsgDeposit](#cosmos.gov.v1beta1.MsgDeposit) | [MsgDepositResponse](#cosmos.gov.v1beta1.MsgDepositResponse) | Deposit defines a method to add deposit on a specific proposal. | |

 <!-- end services -->



<a name="cosmos/gov/v1beta2/gov.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta2/gov.proto



<a name="cosmos.gov.v1beta2.Deposit"></a>

### Deposit
存款定义了帐户地址存入的活跃提案的金额。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `depositor` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.gov.v1beta2.DepositParams"></a>

### DepositParams
DepositParams 定义了治理提案中存款的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `min_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | Minimum deposit for a proposal to enter voting period. |
| `max_deposit_period` | [google.protobuf.Duration](#google.protobuf.Duration) |  | Maximum period for Atom holders to deposit on a proposal. Initial value: 2 months. |






<a name="cosmos.gov.v1beta2.Proposal"></a>

### Proposal
提案定义了治理提案的核心字段成员。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `messages` | [google.protobuf.Any](#google.protobuf.Any) | repeated |  |
| `status` | [ProposalStatus](#cosmos.gov.v1beta2.ProposalStatus) |  |  |
| `final_tally_result` | [TallyResult](#cosmos.gov.v1beta2.TallyResult) |  |  |
| `submit_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `deposit_end_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `total_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `voting_start_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |
| `voting_end_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |






<a name="cosmos.gov.v1beta2.TallyParams"></a>

### TallyParams
TallyParams 定义了用于统计治理提案投票的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `quorum` | [bytes](#bytes) |  | Minimum percentage of total stake needed to vote for a result to be considered valid. |
| `threshold` | [bytes](#bytes) |  | Minimum proportion of Yes votes for proposal to pass. Default value: 0.5. |
| `veto_threshold` | [bytes](#bytes) |  | Minimum value of Veto votes to Total votes ratio for proposal to be vetoed. Default value: 1/3. |






<a name="cosmos.gov.v1beta2.TallyResult"></a>

### TallyResult
TallyResult 定义了治理提案的标准计数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `yes` | [string](#string) |  |  |
| `abstain` | [string](#string) |  |  |
| `no` | [string](#string) |  |  |
| `no_with_veto` | [string](#string) |  |  |






<a name="cosmos.gov.v1beta2.Vote"></a>

### Vote
Vote 定义了对治理提案的投票。Vote 由提案 ID、投票者和投票选项组成。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `option` | [VoteOption](#cosmos.gov.v1beta2.VoteOption) |  | **Deprecated.** Deprecated: Prefer to use `options` instead. This field is set in queries if and only if `len(options) == 1` and that option has weight 1. In all other cases, this field will default to VOTE_OPTION_UNSPECIFIED. |
| `options` | [WeightedVoteOption](#cosmos.gov.v1beta2.WeightedVoteOption) | repeated |  |






<a name="cosmos.gov.v1beta2.VotingParams"></a>

### VotingParams
VotingParams 定义了对治理提案进行投票的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `voting_period` | [google.protobuf.Duration](#google.protobuf.Duration) |  | Length of the voting period. |






<a name="cosmos.gov.v1beta2.WeightedVoteOption"></a>

### WeightedVoteOption
WeightedVoteOption 定义了投票分裂的投票单位。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `option` | [VoteOption](#cosmos.gov.v1beta2.VoteOption) |  |  |
| `weight` | [string](#string) |  |  |





 <!-- end messages -->


<a name="cosmos.gov.v1beta2.ProposalStatus"></a>

### ProposalStatus
ProposalStatus 枚举提案的有效状态。

| Name | Number | Description |
| ---- | ------ | ----------- |
| PROPOSAL_STATUS_UNSPECIFIED | 0 | PROPOSAL_STATUS_UNSPECIFIED defines the default propopsal status. |
| PROPOSAL_STATUS_DEPOSIT_PERIOD | 1 | PROPOSAL_STATUS_DEPOSIT_PERIOD defines a proposal status during the deposit period. |
| PROPOSAL_STATUS_VOTING_PERIOD | 2 | PROPOSAL_STATUS_VOTING_PERIOD defines a proposal status during the voting period. |
| PROPOSAL_STATUS_PASSED | 3 | PROPOSAL_STATUS_PASSED defines a proposal status of a proposal that has passed. |
| PROPOSAL_STATUS_REJECTED | 4 | PROPOSAL_STATUS_REJECTED defines a proposal status of a proposal that has been rejected. |
| PROPOSAL_STATUS_FAILED | 5 | PROPOSAL_STATUS_FAILED defines a proposal status of a proposal that has failed. |



<a name="cosmos.gov.v1beta2.VoteOption"></a>

### VoteOption
VoteOption 枚举给定治理提案的有效投票选项。

| Name | Number | Description |
| ---- | ------ | ----------- |
| VOTE_OPTION_UNSPECIFIED | 0 | VOTE_OPTION_UNSPECIFIED defines a no-op vote option. |
| VOTE_OPTION_YES | 1 | VOTE_OPTION_YES defines a yes vote option. |
| VOTE_OPTION_ABSTAIN | 2 | VOTE_OPTION_ABSTAIN defines an abstain vote option. |
| VOTE_OPTION_NO | 3 | VOTE_OPTION_NO defines a no vote option. |
| VOTE_OPTION_NO_WITH_VETO | 4 | VOTE_OPTION_NO_WITH_VETO defines a no with veto vote option. |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/gov/v1beta2/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta2/genesis.proto



<a name="cosmos.gov.v1beta2.GenesisState"></a>

### GenesisState
GenesisState 定义了 gov 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `starting_proposal_id` | [uint64](#uint64) |  | starting_proposal_id is the ID of the starting proposal. |
| `deposits` | [Deposit](#cosmos.gov.v1beta2.Deposit) | repeated | deposits defines all the deposits present at genesis. |
| `votes` | [Vote](#cosmos.gov.v1beta2.Vote) | repeated | votes defines all the votes present at genesis. |
| `proposals` | [Proposal](#cosmos.gov.v1beta2.Proposal) | repeated | proposals defines all the proposals present at genesis. |
| `deposit_params` | [DepositParams](#cosmos.gov.v1beta2.DepositParams) |  | params defines all the paramaters of related to deposit. |
| `voting_params` | [VotingParams](#cosmos.gov.v1beta2.VotingParams) |  | params defines all the paramaters of related to voting. |
| `tally_params` | [TallyParams](#cosmos.gov.v1beta2.TallyParams) |  | params defines all the paramaters of related to tally. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/gov/v1beta2/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta2/query.proto



<a name="cosmos.gov.v1beta2.QueryDepositRequest"></a>

### QueryDepositRequest
QueryDepositRequest 是 Query/Deposit RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `depositor` | [string](#string) |  | depositor defines the deposit addresses from the proposals. |






<a name="cosmos.gov.v1beta2.QueryDepositResponse"></a>

### QueryDepositResponse
QueryDepositResponse 是 Query/Deposit RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `deposit` | [Deposit](#cosmos.gov.v1beta2.Deposit) |  | deposit defines the requested deposit. |






<a name="cosmos.gov.v1beta2.QueryDepositsRequest"></a>

### QueryDepositsRequest
QueryDepositsRequest 是 Query/Deposits RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta2.QueryDepositsResponse"></a>

### QueryDepositsResponse
QueryDepositsResponse 是 Query/Deposits RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `deposits` | [Deposit](#cosmos.gov.v1beta2.Deposit) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.gov.v1beta2.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params_type` | [string](#string) |  | params_type defines which parameters to query for, can be one of "voting", "tallying" or "deposit". |






<a name="cosmos.gov.v1beta2.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `voting_params` | [VotingParams](#cosmos.gov.v1beta2.VotingParams) |  | voting_params defines the parameters related to voting. |
| `deposit_params` | [DepositParams](#cosmos.gov.v1beta2.DepositParams) |  | deposit_params defines the parameters related to deposit. |
| `tally_params` | [TallyParams](#cosmos.gov.v1beta2.TallyParams) |  | tally_params defines the parameters related to tally. |






<a name="cosmos.gov.v1beta2.QueryProposalRequest"></a>

### QueryProposalRequest
QueryProposalRequest 是 Query/Proposal RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |






<a name="cosmos.gov.v1beta2.QueryProposalResponse"></a>

### QueryProposalResponse
QueryProposalResponse 是 Query/Proposal RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal` | [Proposal](#cosmos.gov.v1beta2.Proposal) |  |  |






<a name="cosmos.gov.v1beta2.QueryProposalsRequest"></a>

### QueryProposalsRequest
QueryProposalsRequest 是 Query/Proposals RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_status` | [ProposalStatus](#cosmos.gov.v1beta2.ProposalStatus) |  | proposal_status defines the status of the proposals. |
| `voter` | [string](#string) |  | voter defines the voter address for the proposals. |
| `depositor` | [string](#string) |  | depositor defines the deposit addresses from the proposals. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta2.QueryProposalsResponse"></a>

### QueryProposalsResponse
QueryProposalsResponse 是 Query/Proposals RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposals` | [Proposal](#cosmos.gov.v1beta2.Proposal) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.gov.v1beta2.QueryTallyResultRequest"></a>

### QueryTallyResultRequest
QueryTallyResultRequest 是 Query/Tally RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |






<a name="cosmos.gov.v1beta2.QueryTallyResultResponse"></a>

### QueryTallyResultResponse
QueryTallyResultResponse 是 Query/Tally RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tally` | [TallyResult](#cosmos.gov.v1beta2.TallyResult) |  | tally defines the requested tally. |






<a name="cosmos.gov.v1beta2.QueryVoteRequest"></a>

### QueryVoteRequest
QueryVoteRequest 是 Query/Vote RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `voter` | [string](#string) |  | voter defines the oter address for the proposals. |






<a name="cosmos.gov.v1beta2.QueryVoteResponse"></a>

### QueryVoteResponse
QueryVoteResponse 是 Query/Vote RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `vote` | [Vote](#cosmos.gov.v1beta2.Vote) |  | vote defined the queried vote. |






<a name="cosmos.gov.v1beta2.QueryVotesRequest"></a>

### QueryVotesRequest
QueryVotesRequestは、Query/VotesRPCメソッドのリクエストタイプです。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id defines the unique id of the proposal. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.gov.v1beta2.QueryVotesResponse"></a>

### QueryVotesResponse
QueryVotesResponse 是 Query/Votes RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `votes` | [Vote](#cosmos.gov.v1beta2.Vote) | repeated | votes defined the queried votes. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.gov.v1beta2.Query"></a>

### Query
Query 定义了 gov 模块的 gRPC 查询器服务

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Proposal` | [QueryProposalRequest](#cosmos.gov.v1beta2.QueryProposalRequest) | [QueryProposalResponse](#cosmos.gov.v1beta2.QueryProposalResponse) | Proposal queries proposal details based on ProposalID. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}|
| `Proposals` | [QueryProposalsRequest](#cosmos.gov.v1beta2.QueryProposalsRequest) | [QueryProposalsResponse](#cosmos.gov.v1beta2.QueryProposalsResponse) | Proposals queries all proposals based on given status. | GET|/cosmos/gov/v1beta2/proposals|
| `Vote` | [QueryVoteRequest](#cosmos.gov.v1beta2.QueryVoteRequest) | [QueryVoteResponse](#cosmos.gov.v1beta2.QueryVoteResponse) | Vote queries voted information based on proposalID, voterAddr. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}/votes/{voter}|
| `Votes` | [QueryVotesRequest](#cosmos.gov.v1beta2.QueryVotesRequest) | [QueryVotesResponse](#cosmos.gov.v1beta2.QueryVotesResponse) | Votes queries votes of a given proposal. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}/votes|
| `Params` | [QueryParamsRequest](#cosmos.gov.v1beta2.QueryParamsRequest) | [QueryParamsResponse](#cosmos.gov.v1beta2.QueryParamsResponse) | Params queries all parameters of the gov module. | GET|/cosmos/gov/v1beta2/params/{params_type}|
| `Deposit` | [QueryDepositRequest](#cosmos.gov.v1beta2.QueryDepositRequest) | [QueryDepositResponse](#cosmos.gov.v1beta2.QueryDepositResponse) | Deposit queries single deposit information based proposalID, depositAddr. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}/deposits/{depositor}|
| `Deposits` | [QueryDepositsRequest](#cosmos.gov.v1beta2.QueryDepositsRequest) | [QueryDepositsResponse](#cosmos.gov.v1beta2.QueryDepositsResponse) | Deposits queries all deposits of a single proposal. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}/deposits|
| `TallyResult` | [QueryTallyResultRequest](#cosmos.gov.v1beta2.QueryTallyResultRequest) | [QueryTallyResultResponse](#cosmos.gov.v1beta2.QueryTallyResultResponse) | TallyResult queries the tally of a proposal vote. | GET|/cosmos/gov/v1beta2/proposals/{proposal_id}/tally|

 <!-- end services -->



<a name="cosmos/gov/v1beta2/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/gov/v1beta2/tx.proto



<a name="cosmos.gov.v1beta2.MsgDeposit"></a>

### MsgDeposit
MsgDeposit 定义了向现有提案提交存款的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `depositor` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.gov.v1beta2.MsgDepositResponse"></a>

### MsgDepositResponse
MsgDepositResponse 定义了 Msg/Deposit 响应类型。






<a name="cosmos.gov.v1beta2.MsgSubmitProposal"></a>

### MsgSubmitProposal
MsgSubmitProposal 定义了一个 sdk.Msg 类型，支持提交任意提案内容。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `messages` | [google.protobuf.Any](#google.protobuf.Any) | repeated |  |
| `initial_deposit` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `proposer` | [string](#string) |  |  |






<a name="cosmos.gov.v1beta2.MsgSubmitProposalResponse"></a>

### MsgSubmitProposalResponse
MsgSubmitProposalResponse 定义了 Msg/SubmitProposal 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |






<a name="cosmos.gov.v1beta2.MsgVote"></a>

### MsgVote
MsgVote 定义了用于投票的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `option` | [VoteOption](#cosmos.gov.v1beta2.VoteOption) |  |  |






<a name="cosmos.gov.v1beta2.MsgVoteResponse"></a>

### MsgVoteResponse
MsgVoteResponse 定义了 Msg/Vote 响应类型。






<a name="cosmos.gov.v1beta2.MsgVoteWeighted"></a>

### MsgVoteWeighted
MsgVoteWeighted 定义了用于投票的消息。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  |  |
| `voter` | [string](#string) |  |  |
| `options` | [WeightedVoteOption](#cosmos.gov.v1beta2.WeightedVoteOption) | repeated |  |






<a name="cosmos.gov.v1beta2.MsgVoteWeightedResponse"></a>

### MsgVoteWeightedResponse
MsgVoteWeightedResponse 定义了 Msg/VoteWeighted 响应类型。

Since: cosmos-sdk 0.43





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.gov.v1beta2.Msg"></a>

### Msg
Msg 定义了gov Msg 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `SubmitProposal` | [MsgSubmitProposal](#cosmos.gov.v1beta2.MsgSubmitProposal) | [MsgSubmitProposalResponse](#cosmos.gov.v1beta2.MsgSubmitProposalResponse) | SubmitProposal defines a method to create new proposal given a content. | |
| `Vote` | [MsgVote](#cosmos.gov.v1beta2.MsgVote) | [MsgVoteResponse](#cosmos.gov.v1beta2.MsgVoteResponse) | Vote defines a method to add a vote on a specific proposal. | |
| `VoteWeighted` | [MsgVoteWeighted](#cosmos.gov.v1beta2.MsgVoteWeighted) | [MsgVoteWeightedResponse](#cosmos.gov.v1beta2.MsgVoteWeightedResponse) | VoteWeighted defines a method to add a weighted vote on a specific proposal.

Since: cosmos-sdk 0.43 | |
| `Deposit` | [MsgDeposit](#cosmos.gov.v1beta2.MsgDeposit) | [MsgDepositResponse](#cosmos.gov.v1beta2.MsgDepositResponse) | Deposit defines a method to add deposit on a specific proposal. | |

 <!-- end services -->



<a name="cosmos/group/v1beta1/events.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/group/v1beta1/events.proto



<a name="cosmos.group.v1beta1.EventCreateGroup"></a>

### EventCreateGroup
EventCreateGroup 是在创建组时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |






<a name="cosmos.group.v1beta1.EventCreateGroupAccount"></a>

### EventCreateGroupAccount
EventCreateGroupAccount 是在创建组帐户时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address of the group account. |






<a name="cosmos.group.v1beta1.EventCreateProposal"></a>

### EventCreateProposal
EventCreateProposal 是在创建提案时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of the proposal. |






<a name="cosmos.group.v1beta1.EventExec"></a>

### EventExec
EventExec 是在执行提案时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of the proposal. |






<a name="cosmos.group.v1beta1.EventUpdateGroup"></a>

### EventUpdateGroup
EventUpdateGroup 是更新组时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |






<a name="cosmos.group.v1beta1.EventUpdateGroupAccount"></a>

### EventUpdateGroupAccount
EventUpdateGroupAccount 是更新组帐户时发出的事件。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address of the group account. |






<a name="cosmos.group.v1beta1.EventVote"></a>

### EventVote
eventVote是在选民投票上提案时发出的活动。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of the proposal. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/group/v1beta1/types.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/group/v1beta1/types.proto



<a name="cosmos.group.v1beta1.GroupAccountInfo"></a>

### GroupAccountInfo
GroupAccountInfo 代表一个群组账户的高级链上信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the group account address. |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the group account. |
| `version` | [uint64](#uint64) |  | version is used to track changes to a group's GroupAccountInfo structure that would create a different result on a running proposal. |
| `decision_policy` | [google.protobuf.Any](#google.protobuf.Any) |  | decision_policy specifies the group account's decision policy. |






<a name="cosmos.group.v1beta1.GroupInfo"></a>

### GroupInfo
GroupInfo 代表一个组的高级链上信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `admin` | [string](#string) |  | admin is the account address of the group's admin. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the group. |
| `version` | [uint64](#uint64) |  | version is used to track changes to a group's membership structure that would break existing proposals. Whenever any members weight is changed, or any member is added or removed this version is incremented and will cause proposals based on older versions of this group to fail |
| `total_weight` | [string](#string) |  | total_weight is the sum of the group members' weights. |






<a name="cosmos.group.v1beta1.GroupMember"></a>

### GroupMember
GroupMember 表示组和成员之间的关系。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `member` | [Member](#cosmos.group.v1beta1.Member) |  | member is the member data. |






<a name="cosmos.group.v1beta1.Member"></a>

### Member
Member 代表具有帐户地址、非零权重和元数据的组成员。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the member's account address. |
| `weight` | [string](#string) |  | weight is the member's voting weight that should be greater than 0. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the member. |






<a name="cosmos.group.v1beta1.Members"></a>

### Members
成员定义了成员对象的重复切片。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `members` | [Member](#cosmos.group.v1beta1.Member) | repeated | members is the list of members. |






<a name="cosmos.group.v1beta1.Proposal"></a>

### Proposal
提案定义了一个组提案。 群组的任何成员都可以提交提案以供群组帐户决定。
一个提案由一组 `sdk.Msg` 组成，如果提案通过，将执行这些 `sdk.Msg` 以及与提案相关的一些可选元数据。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique id of the proposal. |
| `address` | [string](#string) |  | address is the group account address. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the proposal. |
| `proposers` | [string](#string) | repeated | proposers are the account addresses of the proposers. |
| `submitted_at` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | submitted_at is a timestamp specifying when a proposal was submitted. |
| `group_version` | [uint64](#uint64) |  | group_version tracks the version of the group that this proposal corresponds to. When group membership is changed, existing proposals from previous group versions will become invalid. |
| `group_account_version` | [uint64](#uint64) |  | group_account_version tracks the version of the group account that this proposal corresponds to. When a decision policy is changed, existing proposals from previous policy versions will become invalid. |
| `status` | [Proposal.Status](#cosmos.group.v1beta1.Proposal.Status) |  | Status represents the high level position in the life cycle of the proposal. Initial value is Submitted. |
| `result` | [Proposal.Result](#cosmos.group.v1beta1.Proposal.Result) |  | result is the final result based on the votes and election rule. Initial value is unfinalized. The result is persisted so that clients can always rely on this state and not have to replicate the logic. |
| `vote_state` | [Tally](#cosmos.group.v1beta1.Tally) |  | vote_state contains the sums of all weighted votes for this proposal. |
| `timeout` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | timeout is the timestamp of the block where the proposal execution times out. Header times of the votes and execution messages must be before this end time to be included in the election. After the timeout timestamp the proposal can not be executed anymore and should be considered pending delete. |
| `executor_result` | [Proposal.ExecutorResult](#cosmos.group.v1beta1.Proposal.ExecutorResult) |  | executor_result is the final result based on the votes and election rule. Initial value is NotRun. |
| `msgs` | [google.protobuf.Any](#google.protobuf.Any) | repeated | msgs is a list of Msgs that will be executed if the proposal passes. |






<a name="cosmos.group.v1beta1.Tally"></a>

### Tally
Tally 代表加权投票的总和。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `yes_count` | [string](#string) |  | yes_count is the weighted sum of yes votes. |
| `no_count` | [string](#string) |  | no_count is the weighted sum of no votes. |
| `abstain_count` | [string](#string) |  | abstain_count is the weighted sum of abstainers |
| `veto_count` | [string](#string) |  | veto_count is the weighted sum of vetoes. |






<a name="cosmos.group.v1beta1.ThresholdDecisionPolicy"></a>

### ThresholdDecisionPolicy
ThresholdDecisionPolicy 实现了 DecisionPolicy 接口


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `threshold` | [string](#string) |  | threshold is the minimum weighted sum of yes votes that must be met or exceeded for a proposal to succeed. |
| `timeout` | [google.protobuf.Duration](#google.protobuf.Duration) |  | timeout is the duration from submission of a proposal to the end of voting period Within this times votes and exec messages can be submitted. |






<a name="cosmos.group.v1beta1.Vote"></a>

### Vote
Vote 代表对提案的投票。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal is the unique ID of the proposal. |
| `voter` | [string](#string) |  | voter is the account address of the voter. |
| `choice` | [Choice](#cosmos.group.v1beta1.Choice) |  | choice is the voter's choice on the proposal. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the vote. |
| `submitted_at` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | submitted_at is the timestamp when the vote was submitted. |





 <!-- end messages -->


<a name="cosmos.group.v1beta1.Choice"></a>

### Choice
选择定义了可用的投票选择类型。

| Name | Number | Description |
| ---- | ------ | ----------- |
| CHOICE_UNSPECIFIED | 0 | CHOICE_UNSPECIFIED defines a no-op voting choice. |
| CHOICE_NO | 1 | CHOICE_NO defines a no voting choice. |
| CHOICE_YES | 2 | CHOICE_YES defines a yes voting choice. |
| CHOICE_ABSTAIN | 3 | CHOICE_ABSTAIN defines an abstaining voting choice. |
| CHOICE_VETO | 4 | CHOICE_VETO defines a voting choice with veto. |



<a name="cosmos.group.v1beta1.Proposal.ExecutorResult"></a>

### Proposal.ExecutorResult
ExecutorResult 定义了提案执行器结果的类型。

| Name | Number | Description |
| ---- | ------ | ----------- |
| EXECUTOR_RESULT_UNSPECIFIED | 0 | An empty value is not allowed. |
| EXECUTOR_RESULT_NOT_RUN | 1 | We have not yet run the executor. |
| EXECUTOR_RESULT_SUCCESS | 2 | The executor was successful and proposed action updated state. |
| EXECUTOR_RESULT_FAILURE | 3 | The executor returned an error and proposed action didn't update state. |



<a name="cosmos.group.v1beta1.Proposal.Result"></a>

### Proposal.Result
结果定义了提案结果的类型。

| Name | Number | Description |
| ---- | ------ | ----------- |
| RESULT_UNSPECIFIED | 0 | An empty value is invalid and not allowed |
| RESULT_UNFINALIZED | 1 | Until a final tally has happened the status is unfinalized |
| RESULT_ACCEPTED | 2 | Final result of the tally |
| RESULT_REJECTED | 3 | Final result of the tally |



<a name="cosmos.group.v1beta1.Proposal.Status"></a>

### Proposal.Status
状态定义提案状态。

| Name | Number | Description |
| ---- | ------ | ----------- |
| STATUS_UNSPECIFIED | 0 | An empty value is invalid and not allowed. |
| STATUS_SUBMITTED | 1 | Initial status of a proposal when persisted. |
| STATUS_CLOSED | 2 | Final status of a proposal when the final tally was executed. |
| STATUS_ABORTED | 3 | Final status of a proposal when the group was modified before the final tally. |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/group/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/group/v1beta1/query.proto



<a name="cosmos.group.v1beta1.QueryGroupAccountInfoRequest"></a>

### QueryGroupAccountInfoRequest
QueryGroupAccountInfoRequest 是 Query/GroupAccountInfo 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the account address of the group account. |






<a name="cosmos.group.v1beta1.QueryGroupAccountInfoResponse"></a>

### QueryGroupAccountInfoResponse
QueryGroupAccountInfoResponse 是 Query/GroupAccountInfo 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `info` | [GroupAccountInfo](#cosmos.group.v1beta1.GroupAccountInfo) |  | info is the GroupAccountInfo for the group account. |






<a name="cosmos.group.v1beta1.QueryGroupAccountsByAdminRequest"></a>

### QueryGroupAccountsByAdminRequest
QueryGroupAccountsByAdminRequest 是 Query/GroupAccountsByAdmin 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the admin address of the group account. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryGroupAccountsByAdminResponse"></a>

### QueryGroupAccountsByAdminResponse
QueryGroupAccountsByAdminResponse 是 Query/GroupAccountsByAdmin 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_accounts` | [GroupAccountInfo](#cosmos.group.v1beta1.GroupAccountInfo) | repeated | group_accounts are the group accounts info with provided admin. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryGroupAccountsByGroupRequest"></a>

### QueryGroupAccountsByGroupRequest
QueryGroupAccountsByGroupRequest 是 Query/GroupAccountsByGroup 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group account's group. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryGroupAccountsByGroupResponse"></a>

### QueryGroupAccountsByGroupResponse
QueryGroupAccountsByGroupResponse 是 Query/GroupAccountsByGroup 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_accounts` | [GroupAccountInfo](#cosmos.group.v1beta1.GroupAccountInfo) | repeated | group_accounts are the group accounts info associated with the provided group. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryGroupInfoRequest"></a>

### QueryGroupInfoRequest
QueryGroupInfoRequest 是 Query/GroupInfo 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |






<a name="cosmos.group.v1beta1.QueryGroupInfoResponse"></a>

### QueryGroupInfoResponse
QueryGroupInfoResponse 是 Query/GroupInfo 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `info` | [GroupInfo](#cosmos.group.v1beta1.GroupInfo) |  | info is the GroupInfo for the group. |






<a name="cosmos.group.v1beta1.QueryGroupMembersRequest"></a>

### QueryGroupMembersRequest
QueryGroupMembersRequest 是 Query/GroupMembers 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryGroupMembersResponse"></a>

### QueryGroupMembersResponse
QueryGroupMembersResponse 是 Query/GroupMembersResponse 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `members` | [GroupMember](#cosmos.group.v1beta1.GroupMember) | repeated | members are the members of the group with given group_id. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryGroupsByAdminRequest"></a>

### QueryGroupsByAdminRequest
QueryGroupsByAdminRequest 是 Query/GroupsByAdmin 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of a group's admin. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryGroupsByAdminResponse"></a>

### QueryGroupsByAdminResponse
QueryGroupsByAdminResponse 是 Query/GroupsByAdminResponse 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `groups` | [GroupInfo](#cosmos.group.v1beta1.GroupInfo) | repeated | groups are the groups info with the provided admin. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryProposalRequest"></a>

### QueryProposalRequest
QueryProposalRequest 是查询/建议请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of a proposal. |






<a name="cosmos.group.v1beta1.QueryProposalResponse"></a>

### QueryProposalResponse
QueryProposalResponse 是查询/建议响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal` | [Proposal](#cosmos.group.v1beta1.Proposal) |  | proposal is the proposal info. |






<a name="cosmos.group.v1beta1.QueryProposalsByGroupAccountRequest"></a>

### QueryProposalsByGroupAccountRequest
QueryProposalsByGroupAccountRequest 是 Query/ProposalByGroupAccount 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the group account address related to proposals. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryProposalsByGroupAccountResponse"></a>

### QueryProposalsByGroupAccountResponse
QueryProposalsByGroupAccountResponse 是 Query/ProposalByGroupAccount 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposals` | [Proposal](#cosmos.group.v1beta1.Proposal) | repeated | proposals are the proposals with given group account. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryVoteByProposalVoterRequest"></a>

### QueryVoteByProposalVoterRequest
QueryVoteByProposalVoterRequest 是 Query/VoteByProposalVoter 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of a proposal. |
| `voter` | [string](#string) |  | voter is a proposal voter account address. |






<a name="cosmos.group.v1beta1.QueryVoteByProposalVoterResponse"></a>

### QueryVoteByProposalVoterResponse
QueryVoteByProposalVoterResponse 是 Query/VoteByProposalVoter 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `vote` | [Vote](#cosmos.group.v1beta1.Vote) |  | vote is the vote with given proposal_id and voter. |






<a name="cosmos.group.v1beta1.QueryVotesByProposalRequest"></a>

### QueryVotesByProposalRequest
QueryVotesByProposalRequest 是 Query/VotesByProposal 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal_id is the unique ID of a proposal. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryVotesByProposalResponse"></a>

### QueryVotesByProposalResponse
QueryVotesByProposalResponse 是 Query/VotesByProposal 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `votes` | [Vote](#cosmos.group.v1beta1.Vote) | repeated | votes are the list of votes for given proposal_id. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.group.v1beta1.QueryVotesByVoterRequest"></a>

### QueryVotesByVoterRequest
QueryVotesByVoterRequest 是 Query/VotesByVoter 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `voter` | [string](#string) |  | voter is a proposal voter account address. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.group.v1beta1.QueryVotesByVoterResponse"></a>

### QueryVotesByVoterResponse
QueryVotesByVoterResponse 是 Query/VotesByVoter 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `votes` | [Vote](#cosmos.group.v1beta1.Vote) | repeated | votes are the list of votes by given voter. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.group.v1beta1.Query"></a>

### Query
Query是 cosmos.group.v1beta1 查询服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `GroupInfo` | [QueryGroupInfoRequest](#cosmos.group.v1beta1.QueryGroupInfoRequest) | [QueryGroupInfoResponse](#cosmos.group.v1beta1.QueryGroupInfoResponse) | GroupInfo queries group info based on group id. | |
| `GroupAccountInfo` | [QueryGroupAccountInfoRequest](#cosmos.group.v1beta1.QueryGroupAccountInfoRequest) | [QueryGroupAccountInfoResponse](#cosmos.group.v1beta1.QueryGroupAccountInfoResponse) | GroupAccountInfo queries group account info based on group account address. | |
| `GroupMembers` | [QueryGroupMembersRequest](#cosmos.group.v1beta1.QueryGroupMembersRequest) | [QueryGroupMembersResponse](#cosmos.group.v1beta1.QueryGroupMembersResponse) | GroupMembers queries members of a group | |
| `GroupsByAdmin` | [QueryGroupsByAdminRequest](#cosmos.group.v1beta1.QueryGroupsByAdminRequest) | [QueryGroupsByAdminResponse](#cosmos.group.v1beta1.QueryGroupsByAdminResponse) | GroupsByAdmin queries groups by admin address. | |
| `GroupAccountsByGroup` | [QueryGroupAccountsByGroupRequest](#cosmos.group.v1beta1.QueryGroupAccountsByGroupRequest) | [QueryGroupAccountsByGroupResponse](#cosmos.group.v1beta1.QueryGroupAccountsByGroupResponse) | GroupAccountsByGroup queries group accounts by group id. | |
| `GroupAccountsByAdmin` | [QueryGroupAccountsByAdminRequest](#cosmos.group.v1beta1.QueryGroupAccountsByAdminRequest) | [QueryGroupAccountsByAdminResponse](#cosmos.group.v1beta1.QueryGroupAccountsByAdminResponse) | GroupsByAdmin queries group accounts by admin address. | |
| `Proposal` | [QueryProposalRequest](#cosmos.group.v1beta1.QueryProposalRequest) | [QueryProposalResponse](#cosmos.group.v1beta1.QueryProposalResponse) | Proposal queries a proposal based on proposal id. | |
| `ProposalsByGroupAccount` | [QueryProposalsByGroupAccountRequest](#cosmos.group.v1beta1.QueryProposalsByGroupAccountRequest) | [QueryProposalsByGroupAccountResponse](#cosmos.group.v1beta1.QueryProposalsByGroupAccountResponse) | ProposalsByGroupAccount queries proposals based on group account address. | |
| `VoteByProposalVoter` | [QueryVoteByProposalVoterRequest](#cosmos.group.v1beta1.QueryVoteByProposalVoterRequest) | [QueryVoteByProposalVoterResponse](#cosmos.group.v1beta1.QueryVoteByProposalVoterResponse) | VoteByProposalVoter queries a vote by proposal id and voter. | |
| `VotesByProposal` | [QueryVotesByProposalRequest](#cosmos.group.v1beta1.QueryVotesByProposalRequest) | [QueryVotesByProposalResponse](#cosmos.group.v1beta1.QueryVotesByProposalResponse) | VotesByProposal queries a vote by proposal. | |
| `VotesByVoter` | [QueryVotesByVoterRequest](#cosmos.group.v1beta1.QueryVotesByVoterRequest) | [QueryVotesByVoterResponse](#cosmos.group.v1beta1.QueryVotesByVoterResponse) | VotesByVoter queries a vote by voter. | |

 <!-- end services -->



<a name="cosmos/group/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/group/v1beta1/tx.proto



<a name="cosmos.group.v1beta1.MsgCreateGroup"></a>

### MsgCreateGroup
MsgCreateGroup 是 Msg/CreateGroup 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `members` | [Member](#cosmos.group.v1beta1.Member) | repeated | members defines the group members. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the group. |






<a name="cosmos.group.v1beta1.MsgCreateGroupAccount"></a>

### MsgCreateGroupAccount
MsgCreateGroupAccount 是 Msg/CreateGroupAccount 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the group account. |
| `decision_policy` | [google.protobuf.Any](#google.protobuf.Any) |  | decision_policy specifies the group account's decision policy. |






<a name="cosmos.group.v1beta1.MsgCreateGroupAccountResponse"></a>

### MsgCreateGroupAccountResponse
MsgCreateGroupAccountResponse 是 Msg/CreateGroupAccount 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the account address of the newly created group account. |






<a name="cosmos.group.v1beta1.MsgCreateGroupResponse"></a>

### MsgCreateGroupResponse
MsgCreateGroupResponse 是 Msg/CreateGroup 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the newly created group. |






<a name="cosmos.group.v1beta1.MsgCreateProposal"></a>

### MsgCreateProposal
MsgCreateProposal 是 Msg/CreateProposal 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the group account address. |
| `proposers` | [string](#string) | repeated | proposers are the account addresses of the proposers. Proposers signatures will be counted as yes votes. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the proposal. |
| `msgs` | [google.protobuf.Any](#google.protobuf.Any) | repeated | msgs is a list of Msgs that will be executed if the proposal passes. |
| `exec` | [Exec](#cosmos.group.v1beta1.Exec) |  | exec defines the mode of execution of the proposal, whether it should be executed immediately on creation or not. If so, proposers signatures are considered as Yes votes. |






<a name="cosmos.group.v1beta1.MsgCreateProposalResponse"></a>

### MsgCreateProposalResponse
MsgCreateProposalResponse 是 Msg/CreateProposal 响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal is the unique ID of the proposal. |






<a name="cosmos.group.v1beta1.MsgExec"></a>

### MsgExec
MsgExec 是 Msg/Exec 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal is the unique ID of the proposal. |
| `signer` | [string](#string) |  | signer is the account address used to execute the proposal. |






<a name="cosmos.group.v1beta1.MsgExecResponse"></a>

### MsgExecResponse
Msg Exec Response 是Message/Exec 请求类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountAdmin"></a>

### MsgUpdateGroupAccountAdmin
MsgUpdateGroupAccountAdmin 是 Msg/UpdateGroupAccountAdmin 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `address` | [string](#string) |  | address is the group account address. |
| `new_admin` | [string](#string) |  | new_admin is the new group account admin. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountAdminResponse"></a>

### MsgUpdateGroupAccountAdminResponse
MsgUpdateGroupAccountAdminResponse 是 Msg/UpdateGroupAccountAdmin 响应类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicy"></a>

### MsgUpdateGroupAccountDecisionPolicy
MsgUpdateGroupAccountDecisionPolicy 是 Msg/UpdateGroupAccountDecisionPolicy 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `address` | [string](#string) |  | address is the group account address. |
| `decision_policy` | [google.protobuf.Any](#google.protobuf.Any) |  | decision_policy is the updated group account decision policy. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicyResponse"></a>

### MsgUpdateGroupAccountDecisionPolicyResponse
MsgUpdateGroupAccountDecisionPolicyResponse 是 Msg/UpdateGroupAccountDecisionPolicy 响应类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountMetadata"></a>

### MsgUpdateGroupAccountMetadata
MsgUpdateGroupAccountMetadata 是 Msg/UpdateGroupAccountMetadata 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `address` | [string](#string) |  | address is the group account address. |
| `metadata` | [bytes](#bytes) |  | metadata is the updated group account metadata. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupAccountMetadataResponse"></a>

### MsgUpdateGroupAccountMetadataResponse
MsgUpdateGroupAccountMetadataResponse 是 Msg/UpdateGroupAccountMetadata 响应类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupAdmin"></a>

### MsgUpdateGroupAdmin
MsgUpdateGroupAdmin 是 Msg/UpdateGroupAdmin 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the current account address of the group admin. |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `new_admin` | [string](#string) |  | new_admin is the group new admin account address. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupAdminResponse"></a>

### MsgUpdateGroupAdminResponse
MsgUpdateGroupAdminResponse 是 Msg/UpdateGroupAdmin 响应类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupMembers"></a>

### MsgUpdateGroupMembers
消息更新组成员是消息/更新组成员请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `member_updates` | [Member](#cosmos.group.v1beta1.Member) | repeated | member_updates is the list of members to update, set weight to 0 to remove a member. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupMembersResponse"></a>

### MsgUpdateGroupMembersResponse
MsgUpdateGroupMembersResponse 是 Msg/UpdateGroupMembers 响应类型。






<a name="cosmos.group.v1beta1.MsgUpdateGroupMetadata"></a>

### MsgUpdateGroupMetadata
MsgUpdateGroupMetadata 是 Msg/UpdateGroupMetadata 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `admin` | [string](#string) |  | admin is the account address of the group admin. |
| `group_id` | [uint64](#uint64) |  | group_id is the unique ID of the group. |
| `metadata` | [bytes](#bytes) |  | metadata is the updated group's metadata. |






<a name="cosmos.group.v1beta1.MsgUpdateGroupMetadataResponse"></a>

### MsgUpdateGroupMetadataResponse
MsgUpdateGroupMetadataResponse 是 Msg/UpdateGroupMetadata 响应类型。






<a name="cosmos.group.v1beta1.MsgVote"></a>

### MsgVote
MsgVote 是 Msg/Vote 请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `proposal_id` | [uint64](#uint64) |  | proposal is the unique ID of the proposal. |
| `voter` | [string](#string) |  | voter is the voter account address. |
| `choice` | [Choice](#cosmos.group.v1beta1.Choice) |  | choice is the voter's choice on the proposal. |
| `metadata` | [bytes](#bytes) |  | metadata is any arbitrary metadata to attached to the vote. |
| `exec` | [Exec](#cosmos.group.v1beta1.Exec) |  | exec defines whether the proposal should be executed immediately after voting or not. |






<a name="cosmos.group.v1beta1.MsgVoteResponse"></a>

### MsgVoteResponse
MsgVoteResponse 是 Msg/Vote 响应类型。





 <!-- end messages -->


<a name="cosmos.group.v1beta1.Exec"></a>

### Exec
Exec 定义了提案在创建或新投票时的执行模式。

| Name | Number | Description |
| ---- | ------ | ----------- |
| EXEC_UNSPECIFIED | 0 | An empty value means that there should be a separate MsgExec request for the proposal to execute. |
| EXEC_TRY | 1 | Try to execute the proposal immediately. If the proposal is not allowed per the DecisionPolicy, the proposal will still be open and could be executed at a later point. |


 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.group.v1beta1.Msg"></a>

### Msg
消息是 cosmos.group.v1beta1 消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `CreateGroup` | [MsgCreateGroup](#cosmos.group.v1beta1.MsgCreateGroup) | [MsgCreateGroupResponse](#cosmos.group.v1beta1.MsgCreateGroupResponse) | CreateGroup creates a new group with an admin account address, a list of members and some optional metadata. | |
| `UpdateGroupMembers` | [MsgUpdateGroupMembers](#cosmos.group.v1beta1.MsgUpdateGroupMembers) | [MsgUpdateGroupMembersResponse](#cosmos.group.v1beta1.MsgUpdateGroupMembersResponse) | UpdateGroupMembers updates the group members with given group id and admin address. | |
| `UpdateGroupAdmin` | [MsgUpdateGroupAdmin](#cosmos.group.v1beta1.MsgUpdateGroupAdmin) | [MsgUpdateGroupAdminResponse](#cosmos.group.v1beta1.MsgUpdateGroupAdminResponse) | UpdateGroupAdmin updates the group admin with given group id and previous admin address. | |
| `UpdateGroupMetadata` | [MsgUpdateGroupMetadata](#cosmos.group.v1beta1.MsgUpdateGroupMetadata) | [MsgUpdateGroupMetadataResponse](#cosmos.group.v1beta1.MsgUpdateGroupMetadataResponse) | UpdateGroupMetadata updates the group metadata with given group id and admin address. | |
| `CreateGroupAccount` | [MsgCreateGroupAccount](#cosmos.group.v1beta1.MsgCreateGroupAccount) | [MsgCreateGroupAccountResponse](#cosmos.group.v1beta1.MsgCreateGroupAccountResponse) | CreateGroupAccount creates a new group account using given DecisionPolicy. | |
| `UpdateGroupAccountAdmin` | [MsgUpdateGroupAccountAdmin](#cosmos.group.v1beta1.MsgUpdateGroupAccountAdmin) | [MsgUpdateGroupAccountAdminResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountAdminResponse) | UpdateGroupAccountAdmin updates a group account admin. | |
| `UpdateGroupAccountDecisionPolicy` | [MsgUpdateGroupAccountDecisionPolicy](#cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicy) | [MsgUpdateGroupAccountDecisionPolicyResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountDecisionPolicyResponse) | UpdateGroupAccountDecisionPolicy allows a group account decision policy to be updated. | |
| `UpdateGroupAccountMetadata` | [MsgUpdateGroupAccountMetadata](#cosmos.group.v1beta1.MsgUpdateGroupAccountMetadata) | [MsgUpdateGroupAccountMetadataResponse](#cosmos.group.v1beta1.MsgUpdateGroupAccountMetadataResponse) | UpdateGroupAccountMetadata updates a group account metadata. | |
| `CreateProposal` | [MsgCreateProposal](#cosmos.group.v1beta1.MsgCreateProposal) | [MsgCreateProposalResponse](#cosmos.group.v1beta1.MsgCreateProposalResponse) | CreateProposal submits a new proposal. | |
| `Vote` | [MsgVote](#cosmos.group.v1beta1.MsgVote) | [MsgVoteResponse](#cosmos.group.v1beta1.MsgVoteResponse) | Vote allows a voter to vote on a proposal. | |
| `Exec` | [MsgExec](#cosmos.group.v1beta1.MsgExec) | [MsgExecResponse](#cosmos.group.v1beta1.MsgExecResponse) | Exec executes a proposal. | |

 <!-- end services -->



<a name="cosmos/mint/v1beta1/mint.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/mint/v1beta1/mint.proto



<a name="cosmos.mint.v1beta1.Minter"></a>

### Minter
Minter 代表铸币状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `inflation` | [string](#string) |  | current annual inflation rate |
| `annual_provisions` | [string](#string) |  | current annual expected provisions |






<a name="cosmos.mint.v1beta1.Params"></a>

### Params
Params 保存 mint 模块的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `mint_denom` | [string](#string) |  | type of coin to mint |
| `inflation_rate_change` | [string](#string) |  | maximum annual change in inflation rate |
| `inflation_max` | [string](#string) |  | maximum inflation rate |
| `inflation_min` | [string](#string) |  | minimum inflation rate |
| `goal_bonded` | [string](#string) |  | goal of percent bonded atoms |
| `blocks_per_year` | [uint64](#uint64) |  | expected blocks per year |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/mint/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/mint/v1beta1/genesis.proto



<a name="cosmos.mint.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 mint 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `minter` | [Minter](#cosmos.mint.v1beta1.Minter) |  | minter is a space for holding current inflation information. |
| `params` | [Params](#cosmos.mint.v1beta1.Params) |  | params defines all the paramaters of the module. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/mint/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/mint/v1beta1/query.proto



<a name="cosmos.mint.v1beta1.QueryAnnualProvisionsRequest"></a>

### QueryAnnualProvisionsRequest
QueryAnnualProvisionsRequest 是 Query/AnnualProvisions RPC 方法的请求类型。






<a name="cosmos.mint.v1beta1.QueryAnnualProvisionsResponse"></a>

### QueryAnnualProvisionsResponse
QueryAnnualProvisionsResponse 是 Query/AnnualProvisions RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `annual_provisions` | [bytes](#bytes) |  | annual_provisions is the current minting annual provisions value. |






<a name="cosmos.mint.v1beta1.QueryInflationRequest"></a>

### QueryInflationRequest
QueryInflationRequest 是 Query/Inflation RPC 方法的请求类型。






<a name="cosmos.mint.v1beta1.QueryInflationResponse"></a>

### QueryInflationResponse
QueryInflationResponse 是 Query/Inflation RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `inflation` | [bytes](#bytes) |  | inflation is the current minting inflation value. |






<a name="cosmos.mint.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。






<a name="cosmos.mint.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.mint.v1beta1.Params) |  | params defines the parameters of the module. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.mint.v1beta1.Query"></a>

### Query
Query 提供定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Params` | [QueryParamsRequest](#cosmos.mint.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.mint.v1beta1.QueryParamsResponse) | Params returns the total set of minting parameters. | GET|/cosmos/mint/v1beta1/params|
| `Inflation` | [QueryInflationRequest](#cosmos.mint.v1beta1.QueryInflationRequest) | [QueryInflationResponse](#cosmos.mint.v1beta1.QueryInflationResponse) | Inflation returns the current minting inflation value. | GET|/cosmos/mint/v1beta1/inflation|
| `AnnualProvisions` | [QueryAnnualProvisionsRequest](#cosmos.mint.v1beta1.QueryAnnualProvisionsRequest) | [QueryAnnualProvisionsResponse](#cosmos.mint.v1beta1.QueryAnnualProvisionsResponse) | AnnualProvisions current minting annual provisions value. | GET|/cosmos/mint/v1beta1/annual_provisions|

 <!-- end services -->



<a name="cosmos/nft/v1beta1/event.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/nft/v1beta1/event.proto



<a name="cosmos.nft.v1beta1.EventBurn"></a>

### EventBurn
EventBurn 在 Burn 上发出


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `id` | [string](#string) |  |  |
| `owner` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.EventMint"></a>

### EventMint
EventMint 在 Mint 上发出


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `id` | [string](#string) |  |  |
| `owner` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.EventSend"></a>

### EventSend
EventSend 在 Msg/Send 上发出


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `id` | [string](#string) |  |  |
| `sender` | [string](#string) |  |  |
| `receiver` | [string](#string) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/nft/v1beta1/nft.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/nft/v1beta1/nft.proto



<a name="cosmos.nft.v1beta1.Class"></a>

### Class
Class 定义了 nft 类型的类。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `id` | [string](#string) |  | id defines the unique identifier of the NFT classification, similar to the contract address of ERC721 |
| `name` | [string](#string) |  | name defines the human-readable name of the NFT classification. Optional |
| `symbol` | [string](#string) |  | symbol is an abbreviated name for nft classification. Optional |
| `description` | [string](#string) |  | description is a brief description of nft classification. Optional |
| `uri` | [string](#string) |  | uri for the class metadata stored off chain. It can define schema for Class and NFT `Data` attributes. Optional |
| `uri_hash` | [string](#string) |  | uri_hash is a hash of the document pointed by uri. Optional |
| `data` | [google.protobuf.Any](#google.protobuf.Any) |  | data is the app specific metadata of the NFT class. Optional |






<a name="cosmos.nft.v1beta1.NFT"></a>

### NFT
NFT 定义了 NFT。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  | class_id associated with the NFT, similar to the contract address of ERC721 |
| `id` | [string](#string) |  | id is a unique identifier of the NFT |
| `uri` | [string](#string) |  | uri for the NFT metadata stored off chain |
| `uri_hash` | [string](#string) |  | uri_hash is a hash of the document pointed by uri |
| `data` | [google.protobuf.Any](#google.protobuf.Any) |  | data is an app specific data of the NFT. Optional |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/nft/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/nft/v1beta1/genesis.proto



<a name="cosmos.nft.v1beta1.Entry"></a>

### Entry
条目定义一个人拥有的所有 nft


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `owner` | [string](#string) |  | owner is the owner address of the following nft |
| `nfts` | [NFT](#cosmos.nft.v1beta1.NFT) | repeated | nfts is a group of nfts of the same owner |






<a name="cosmos.nft.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 nft 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `classes` | [Class](#cosmos.nft.v1beta1.Class) | repeated | class defines the class of the nft type. |
| `entries` | [Entry](#cosmos.nft.v1beta1.Entry) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/nft/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/nft/v1beta1/query.proto



<a name="cosmos.nft.v1beta1.QueryBalanceRequest"></a>

### QueryBalanceRequest
QueryBalanceRequest 是 Query/Balance RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `owner` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QueryBalanceResponse"></a>

### QueryBalanceResponse
QueryBalanceResponse 是 Query/Balance RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [uint64](#uint64) |  |  |






<a name="cosmos.nft.v1beta1.QueryClassRequest"></a>

### QueryClassRequest
QueryClassRequest 是 Query/Class RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QueryClassResponse"></a>

### QueryClassResponse
QueryClassResponse 是 Query/Class RPC 方法的响应类型

| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class` | [Class](#cosmos.nft.v1beta1.Class) |  |  |






<a name="cosmos.nft.v1beta1.QueryClassesRequest"></a>

### QueryClassesRequest
QueryClassesRequest 是 Query/Classes RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.nft.v1beta1.QueryClassesResponse"></a>

### QueryClassesResponse
QueryClassesResponse 是 Query/Classes RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `classes` | [Class](#cosmos.nft.v1beta1.Class) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  |  |






<a name="cosmos.nft.v1beta1.QueryNFTRequest"></a>

### QueryNFTRequest
QueryNFTRequest 是 Query/NFT RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `id` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QueryNFTResponse"></a>

### QueryNFTResponse
QueryNFTResponse 是 Query/NFT RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `nft` | [NFT](#cosmos.nft.v1beta1.NFT) |  |  |






<a name="cosmos.nft.v1beta1.QueryNFTsOfClassRequest"></a>

### QueryNFTsOfClassRequest
QueryNFTsOfClassRequest 是 Query/NFTsOfClass RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `owner` | [string](#string) |  |  |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  |  |






<a name="cosmos.nft.v1beta1.QueryNFTsOfClassResponse"></a>

### QueryNFTsOfClassResponse
QueryNFTsOfClassResponse 是 Query/NFTsOfClass 和 Query/NFTsOfClassByOwner RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `nfts` | [NFT](#cosmos.nft.v1beta1.NFT) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  |  |






<a name="cosmos.nft.v1beta1.QueryOwnerRequest"></a>

### QueryOwnerRequest
QueryOwnerRequest 是 Query/Owner RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |
| `id` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QueryOwnerResponse"></a>

### QueryOwnerResponse
QueryOwnerResponse 是 Query/Owner RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `owner` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QuerySupplyRequest"></a>

### QuerySupplyRequest
QuerySupplyRequest 是 Query/Supply RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  |  |






<a name="cosmos.nft.v1beta1.QuerySupplyResponse"></a>

### QuerySupplyResponse
QuerySupplyResponse 是 Query/Supply RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [uint64](#uint64) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.nft.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Balance` | [QueryBalanceRequest](#cosmos.nft.v1beta1.QueryBalanceRequest) | [QueryBalanceResponse](#cosmos.nft.v1beta1.QueryBalanceResponse) | Balance queries the number of NFTs of a given class owned by the owner, same as balanceOf in ERC721 | GET|/cosmos/nft/v1beta1/balance/{owner}/{class_id}|
| `Owner` | [QueryOwnerRequest](#cosmos.nft.v1beta1.QueryOwnerRequest) | [QueryOwnerResponse](#cosmos.nft.v1beta1.QueryOwnerResponse) | Owner queries the owner of the NFT based on its class and id, same as ownerOf in ERC721 | GET|/cosmos/nft/v1beta1/owner/{class_id}/{id}|
| `Supply` | [QuerySupplyRequest](#cosmos.nft.v1beta1.QuerySupplyRequest) | [QuerySupplyResponse](#cosmos.nft.v1beta1.QuerySupplyResponse) | Supply queries the number of NFTs from the given class, same as totalSupply of ERC721. | GET|/cosmos/nft/v1beta1/supply/{class_id}|
| `NFTsOfClass` | [QueryNFTsOfClassRequest](#cosmos.nft.v1beta1.QueryNFTsOfClassRequest) | [QueryNFTsOfClassResponse](#cosmos.nft.v1beta1.QueryNFTsOfClassResponse) | NFTsOfClass queries all NFTs of a given class or optional owner, similar to tokenByIndex in ERC721Enumerable | GET|/cosmos/nft/v1beta1/nfts/{class_id}|
| `NFT` | [QueryNFTRequest](#cosmos.nft.v1beta1.QueryNFTRequest) | [QueryNFTResponse](#cosmos.nft.v1beta1.QueryNFTResponse) | NFT queries an NFT based on its class and id. | GET|/cosmos/nft/v1beta1/nfts/{class_id}/{id}|
| `Class` | [QueryClassRequest](#cosmos.nft.v1beta1.QueryClassRequest) | [QueryClassResponse](#cosmos.nft.v1beta1.QueryClassResponse) | Class queries an NFT class based on its id | GET|/cosmos/nft/v1beta1/classes/{class_id}|
| `Classes` | [QueryClassesRequest](#cosmos.nft.v1beta1.QueryClassesRequest) | [QueryClassesResponse](#cosmos.nft.v1beta1.QueryClassesResponse) | Classes queries all NFT classes | GET|/cosmos/nft/v1beta1/classes|

 <!-- end services -->



<a name="cosmos/nft/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/nft/v1beta1/tx.proto



<a name="cosmos.nft.v1beta1.MsgSend"></a>

### MsgSend
MsgSend 表示从一个帐户向另一个帐户发送 nft 的消息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `class_id` | [string](#string) |  | class_id defines the unique identifier of the nft classification, similar to the contract address of ERC721 |
| `id` | [string](#string) |  | id defines the unique identification of nft |
| `sender` | [string](#string) |  | sender is the address of the owner of nft |
| `receiver` | [string](#string) |  | receiver is the receiver address of nft |






<a name="cosmos.nft.v1beta1.MsgSendResponse"></a>

### MsgSendResponse
MsgSendResponse 定义了 Msg/Send 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.nft.v1beta1.Msg"></a>

### Msg
Msg 定义了 nft Msg 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Send` | [MsgSend](#cosmos.nft.v1beta1.MsgSend) | [MsgSendResponse](#cosmos.nft.v1beta1.MsgSendResponse) | Send defines a method to send a nft from one account to another account. | |

 <!-- end services -->



<a name="cosmos/params/v1beta1/params.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/params/v1beta1/params.proto



<a name="cosmos.params.v1beta1.ParamChange"></a>

### ParamChange
ParamChange 定义了一个单独的参数更改，用于 ParameterChangeProposal。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `subspace` | [string](#string) |  |  |
| `key` | [string](#string) |  |  |
| `value` | [string](#string) |  |  |






<a name="cosmos.params.v1beta1.ParameterChangeProposal"></a>

### ParameterChangeProposal
ParameterChangeProposal 定义了更改一个或多个参数的提议。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |
| `changes` | [ParamChange](#cosmos.params.v1beta1.ParamChange) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/params/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/params/v1beta1/query.proto



<a name="cosmos.params.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `subspace` | [string](#string) |  | subspace defines the module to query the parameter for. |
| `key` | [string](#string) |  | key defines the key of the parameter in the subspace. |






<a name="cosmos.params.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `param` | [ParamChange](#cosmos.params.v1beta1.ParamChange) |  | param defines the queried parameter. |






<a name="cosmos.params.v1beta1.QuerySubspacesRequest"></a>

### QuerySubspacesRequest
QuerySubspacesRequest 定义了一个请求类型，用于查询所有已注册的
子空间和子空间的所有键。






<a name="cosmos.params.v1beta1.QuerySubspacesResponse"></a>

### QuerySubspacesResponse
QuerySubspacesResponse 定义了所有查询的响应类型
注册的子空间和子空间的所有键。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `subspaces` | [Subspace](#cosmos.params.v1beta1.Subspace) | repeated |  |






<a name="cosmos.params.v1beta1.Subspace"></a>

### Subspace
子空间定义了一个参数子空间名称和所有存在的键
子空间。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `subspace` | [string](#string) |  |  |
| `keys` | [string](#string) | repeated |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.params.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Params` | [QueryParamsRequest](#cosmos.params.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.params.v1beta1.QueryParamsResponse) | Params queries a specific parameter of a module, given its subspace and key. | GET|/cosmos/params/v1beta1/params|
| `Subspaces` | [QuerySubspacesRequest](#cosmos.params.v1beta1.QuerySubspacesRequest) | [QuerySubspacesResponse](#cosmos.params.v1beta1.QuerySubspacesResponse) | Subspaces queries for all registered subspaces and all keys for a subspace. | GET|/cosmos/params/v1beta1/subspaces|

 <!-- end services -->



<a name="cosmos/slashing/v1beta1/slashing.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/slashing/v1beta1/slashing.proto



<a name="cosmos.slashing.v1beta1.Params"></a>

### Params
Params 表示 slashing 模块使用的参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `signed_blocks_window` | [int64](#int64) |  |  |
| `min_signed_per_window` | [bytes](#bytes) |  |  |
| `downtime_jail_duration` | [google.protobuf.Duration](#google.protobuf.Duration) |  |  |
| `slash_fraction_double_sign` | [bytes](#bytes) |  |  |
| `slash_fraction_downtime` | [bytes](#bytes) |  |  |






<a name="cosmos.slashing.v1beta1.ValidatorSigningInfo"></a>

### ValidatorSigningInfo
ValidatorSigningInfo 定义了验证者的签名信息，用于监控他们的活跃度活动。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  |  |
| `start_height` | [int64](#int64) |  | Height at which validator was first a candidate OR was unjailed |
| `index_offset` | [int64](#int64) |  | Index which is incremented each time the validator was a bonded in a block and may have signed a precommit or not. This in conjunction with the `SignedBlocksWindow` param determines the index in the `MissedBlocksBitArray`. |
| `jailed_until` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | Timestamp until which the validator is jailed due to liveness downtime. |
| `tombstoned` | [bool](#bool) |  | Whether or not a validator has been tombstoned (killed out of validator set). It is set once the validator commits an equivocation or for any other configured misbehiavor. |
| `missed_blocks_counter` | [int64](#int64) |  | A counter kept to avoid unnecessary array reads. Note that `Sum(MissedBlocksBitArray)` always equals `MissedBlocksCounter`. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/slashing/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/slashing/v1beta1/genesis.proto



<a name="cosmos.slashing.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 slashing 模块的创世状态。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.slashing.v1beta1.Params) |  | params defines all the paramaters of related to deposit. |
| `signing_infos` | [SigningInfo](#cosmos.slashing.v1beta1.SigningInfo) | repeated | signing_infos represents a map between validator addresses and their signing infos. |
| `missed_blocks` | [ValidatorMissedBlocks](#cosmos.slashing.v1beta1.ValidatorMissedBlocks) | repeated | missed_blocks represents a map between validator addresses and their missed blocks. |






<a name="cosmos.slashing.v1beta1.MissedBlock"></a>

### MissedBlock
MissedBlock 包含高度和未命中状态为布尔值。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `index` | [int64](#int64) |  | index is the height at which the block was missed. |
| `missed` | [bool](#bool) |  | missed is the missed status. |






<a name="cosmos.slashing.v1beta1.SigningInfo"></a>

### SigningInfo
SigningInfo 存储对应地址的验证者签名信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the validator address. |
| `validator_signing_info` | [ValidatorSigningInfo](#cosmos.slashing.v1beta1.ValidatorSigningInfo) |  | validator_signing_info represents the signing info of this validator. |






<a name="cosmos.slashing.v1beta1.ValidatorMissedBlocks"></a>

### ValidatorMissedBlocks
ValidatorMissedBlocks 包含对应的遗漏块数组
地址。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the validator address. |
| `missed_blocks` | [MissedBlock](#cosmos.slashing.v1beta1.MissedBlock) | repeated | missed_blocks is an array of missed blocks by the validator. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/slashing/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/slashing/v1beta1/query.proto



<a name="cosmos.slashing.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型






<a name="cosmos.slashing.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.slashing.v1beta1.Params) |  |  |






<a name="cosmos.slashing.v1beta1.QuerySigningInfoRequest"></a>

### QuerySigningInfoRequest
QuerySigningInfoRequest 是 Query/SigningInfo RPC 方法的请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `cons_address` | [string](#string) |  | cons_address is the address to query signing info of |






<a name="cosmos.slashing.v1beta1.QuerySigningInfoResponse"></a>

### QuerySigningInfoResponse
QuerySigningInfoResponse 是 Query/SigningInfo RPC 的响应类型
方法


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `val_signing_info` | [ValidatorSigningInfo](#cosmos.slashing.v1beta1.ValidatorSigningInfo) |  | val_signing_info is the signing info of requested val cons address |






<a name="cosmos.slashing.v1beta1.QuerySigningInfosRequest"></a>

### QuerySigningInfosRequest
QuerySigningInfosRequest 是 Query/SigningInfos RPC 的请求类型
方法


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  |  |






<a name="cosmos.slashing.v1beta1.QuerySigningInfosResponse"></a>

### QuerySigningInfosResponse
QuerySigningInfosResponse 是 Query/SigningInfos RPC 的响应类型
方法


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `info` | [ValidatorSigningInfo](#cosmos.slashing.v1beta1.ValidatorSigningInfo) | repeated | info is the signing info of all validators |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.slashing.v1beta1.Query"></a>

### Query
Query 提供定义了 gRPC 查询器服务

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Params` | [QueryParamsRequest](#cosmos.slashing.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.slashing.v1beta1.QueryParamsResponse) | Params queries the parameters of slashing module | GET|/cosmos/slashing/v1beta1/params|
| `SigningInfo` | [QuerySigningInfoRequest](#cosmos.slashing.v1beta1.QuerySigningInfoRequest) | [QuerySigningInfoResponse](#cosmos.slashing.v1beta1.QuerySigningInfoResponse) | SigningInfo queries the signing info of given cons address | GET|/cosmos/slashing/v1beta1/signing_infos/{cons_address}|
| `SigningInfos` | [QuerySigningInfosRequest](#cosmos.slashing.v1beta1.QuerySigningInfosRequest) | [QuerySigningInfosResponse](#cosmos.slashing.v1beta1.QuerySigningInfosResponse) | SigningInfos queries signing info of all validators | GET|/cosmos/slashing/v1beta1/signing_infos|

 <!-- end services -->



<a name="cosmos/slashing/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/slashing/v1beta1/tx.proto



<a name="cosmos.slashing.v1beta1.MsgUnjail"></a>

### MsgUnjail
MsgUnjail 定义了 Msg/Unjail 请求类型


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_addr` | [string](#string) |  |  |






<a name="cosmos.slashing.v1beta1.MsgUnjailResponse"></a>

### MsgUnjailResponse
MsgUnjail Response 定义了 Message/Unjail 响应类型





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.slashing.v1beta1.Msg"></a>

### Msg
Msg 定义了slashing Msg 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Unjail` | [MsgUnjail](#cosmos.slashing.v1beta1.MsgUnjail) | [MsgUnjailResponse](#cosmos.slashing.v1beta1.MsgUnjailResponse) | Unjail defines a method for unjailing a jailed validator, thus returning them into the bonded validator set, so they can begin receiving provisions and rewards again. | |

 <!-- end services -->



<a name="cosmos/staking/v1beta1/authz.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/staking/v1beta1/authz.proto



<a name="cosmos.staking.v1beta1.StakeAuthorization"></a>

### StakeAuthorization
StakeAuthorization 定义了委托/取消委托/重新委托的授权。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `max_tokens` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  | max_tokens specifies the maximum amount of tokens can be delegate to a validator. If it is empty, there is no spend limit and any amount of coins can be delegated. |
| `allow_list` | [StakeAuthorization.Validators](#cosmos.staking.v1beta1.StakeAuthorization.Validators) |  | allow_list specifies list of validator addresses to whom grantee can delegate tokens on behalf of granter's account. |
| `deny_list` | [StakeAuthorization.Validators](#cosmos.staking.v1beta1.StakeAuthorization.Validators) |  | deny_list specifies list of validator addresses to whom grantee can not delegate tokens. |
| `authorization_type` | [AuthorizationType](#cosmos.staking.v1beta1.AuthorizationType) |  | authorization_type defines one of AuthorizationType. |






<a name="cosmos.staking.v1beta1.StakeAuthorization.Validators"></a>

### StakeAuthorization.Validators
验证器定义验证器地址列表。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) | repeated |  |





 <!-- end messages -->


<a name="cosmos.staking.v1beta1.AuthorizationType"></a>

### AuthorizationType
AuthorizationType 定义了 Staking 模块授权类型的类型

Since: cosmos-sdk 0.43

| Name | Number | Description |
| ---- | ------ | ----------- |
| AUTHORIZATION_TYPE_UNSPECIFIED | 0 | AUTHORIZATION_TYPE_UNSPECIFIED specifies an unknown authorization type |
| AUTHORIZATION_TYPE_DELEGATE | 1 | AUTHORIZATION_TYPE_DELEGATE defines an authorization type for Msg/Delegate |
| AUTHORIZATION_TYPE_UNDELEGATE | 2 | AUTHORIZATION_TYPE_UNDELEGATE defines an authorization type for Msg/Undelegate |
| AUTHORIZATION_TYPE_REDELEGATE | 3 | AUTHORIZATION_TYPE_REDELEGATE defines an authorization type for Msg/BeginRedelegate |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/staking/v1beta1/staking.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/staking/v1beta1/staking.proto



<a name="cosmos.staking.v1beta1.Commission"></a>

### Commission
佣金定义给定验证器的佣金参数。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `commission_rates` | [CommissionRates](#cosmos.staking.v1beta1.CommissionRates) |  | commission_rates defines the initial commission rates to be used for creating a validator. |
| `update_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | update_time is the last time the commission rate was changed. |






<a name="cosmos.staking.v1beta1.CommissionRates"></a>

### CommissionRates
CommissionRates 定义了用于创建的初始佣金率
一个验证器。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `rate` | [string](#string) |  | rate is the commission rate charged to delegators, as a fraction. |
| `max_rate` | [string](#string) |  | max_rate defines the maximum commission rate which validator can ever charge, as a fraction. |
| `max_change_rate` | [string](#string) |  | max_change_rate defines the maximum daily increase of the validator commission, as a fraction. |






<a name="cosmos.staking.v1beta1.DVPair"></a>

### DVPair
DVPair 是一个结构体，它只有一个委托者-验证者对，没有其他数据。
它旨在用作可编组指针。 例如，DVPair 可用于构造从状态获取 UnbondingDelegation 的密钥。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_address` | [string](#string) |  |  |






<a name="cosmos.staking.v1beta1.DVPairs"></a>

### DVPairs
DVPairs 定义了一组 DVPair 对象。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pairs` | [DVPair](#cosmos.staking.v1beta1.DVPair) | repeated |  |






<a name="cosmos.staking.v1beta1.DVVTriplet"></a>

### DVVTriplet
DVVTriplet 是只有一个委托人-验证人-验证人三元组的结构
没有其他数据。 它旨在用作可编组指针。 为了
例如，一个 DVVTriplet 可用于构造获得
从州重新授权。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_src_address` | [string](#string) |  |  |
| `validator_dst_address` | [string](#string) |  |  |






<a name="cosmos.staking.v1beta1.DVVTriplets"></a>

### DVVTriplets
DVV Triplets 定义了一组 DVV Triplet 对象。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `triplets` | [DVVTriplet](#cosmos.staking.v1beta1.DVVTriplet) | repeated |  |






<a name="cosmos.staking.v1beta1.Delegation"></a>

### Delegation
委托代表与账户持有的代币的债券。 它由一名委托人拥有，并与一名验证人的投票权相关联。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address is the bech32-encoded address of the delegator. |
| `validator_address` | [string](#string) |  | validator_address is the bech32-encoded address of the validator. |
| `shares` | [string](#string) |  | shares define the delegation shares received. |






<a name="cosmos.staking.v1beta1.DelegationResponse"></a>

### DelegationResponse
委托响应等价于委托，除了它包含一个
除了股票之外的余额更适合客户响应。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegation` | [Delegation](#cosmos.staking.v1beta1.Delegation) |  |  |
| `balance` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  |  |






<a name="cosmos.staking.v1beta1.Description"></a>

### Description
描述定义验证器描述。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `moniker` | [string](#string) |  | moniker defines a human-readable name for the validator. |
| `identity` | [string](#string) |  | identity defines an optional identity signature (ex. UPort or Keybase). |
| `website` | [string](#string) |  | website defines an optional website link. |
| `security_contact` | [string](#string) |  | security_contact defines an optional email for security contact. |
| `details` | [string](#string) |  | details define other optional details. |






<a name="cosmos.staking.v1beta1.HistoricalInfo"></a>

### HistoricalInfo
HistoricalInfo 包含给定块的标头和验证器信息。
它被存储为 staking 模块状态的一部分，它最持久地保持 `n`
最近的历史信息 
(`n` 由堆叠模块的 `historical entries` 参数设置 ).


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `header` | [tendermint.types.Header](#tendermint.types.Header) |  |  |
| `valset` | [Validator](#cosmos.staking.v1beta1.Validator) | repeated |  |






<a name="cosmos.staking.v1beta1.Params"></a>

### Params
params 定义了 staking 模块的参数 .


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `unbonding_time` | [google.protobuf.Duration](#google.protobuf.Duration) |  | unbonding_time is the time duration of unbonding. |
| `max_validators` | [uint32](#uint32) |  | max_validators is the maximum number of validators. |
| `max_entries` | [uint32](#uint32) |  | max_entries is the max entries for either unbonding delegation or redelegation (per pair/trio). |
| `historical_entries` | [uint32](#uint32) |  | historical_entries is the number of historical entries to persist. |
| `bond_denom` | [string](#string) |  | bond_denom defines the bondable coin denomination. |
| `min_commission_rate` | [string](#string) |  | min_commission_rate is the chain-wide minimum commission rate that a validator can charge their delegators |






<a name="cosmos.staking.v1beta1.Pool"></a>

### Pool
池用于跟踪债券的保税和非保税代币供应
面值


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `not_bonded_tokens` | [string](#string) |  |  |
| `bonded_tokens` | [string](#string) |  |  |






<a name="cosmos.staking.v1beta1.Redelegation"></a>

### Redelegation
重新委托包含特定委托人的重新委托债券的列表
从特定的源验证器到特定的目标验证器。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address is the bech32-encoded address of the delegator. |
| `validator_src_address` | [string](#string) |  | validator_src_address is the validator redelegation source operator address. |
| `validator_dst_address` | [string](#string) |  | validator_dst_address is the validator redelegation destination operator address. |
| `entries` | [RedelegationEntry](#cosmos.staking.v1beta1.RedelegationEntry) | repeated | entries are the redelegation entries.

redelegation entries |






<a name="cosmos.staking.v1beta1.RedelegationEntry"></a>

### RedelegationEntry
RedelegationEntry 定义了一个具有相关元数据的重新授权对象。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `creation_height` | [int64](#int64) |  | creation_height defines the height which the redelegation took place. |
| `completion_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | completion_time defines the unix time for redelegation completion. |
| `initial_balance` | [string](#string) |  | initial_balance defines the initial balance when redelegation started. |
| `shares_dst` | [string](#string) |  | shares_dst is the amount of destination-validator shares created by redelegation. |






<a name="cosmos.staking.v1beta1.RedelegationEntryResponse"></a>

### RedelegationEntryResponse
RedelegationEntryResponse 等价于 RedelegationEntry，除了它包含更适合客户端响应的份额之外的余额。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `redelegation_entry` | [RedelegationEntry](#cosmos.staking.v1beta1.RedelegationEntry) |  |  |
| `balance` | [string](#string) |  |  |






<a name="cosmos.staking.v1beta1.RedelegationResponse"></a>

### RedelegationResponse
RedelelegationResponse 等价于 Redelegation，不同之处在于它的条目除了更适合客户端响应的份额之外还包含余额。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `redelegation` | [Redelegation](#cosmos.staking.v1beta1.Redelegation) |  |  |
| `entries` | [RedelegationEntryResponse](#cosmos.staking.v1beta1.RedelegationEntryResponse) | repeated |  |






<a name="cosmos.staking.v1beta1.UnbondingDelegation"></a>

### UnbondingDelegation
UnbondingDelegation 将单个委托人的所有解除绑定债券存储在一个按时间排序的列表中。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  | delegator_address is the bech32-encoded address of the delegator. |
| `validator_address` | [string](#string) |  | validator_address is the bech32-encoded address of the validator. |
| `entries` | [UnbondingDelegationEntry](#cosmos.staking.v1beta1.UnbondingDelegationEntry) | repeated | entries are the unbonding delegation entries.

unbonding delegation entries |






<a name="cosmos.staking.v1beta1.UnbondingDelegationEntry"></a>

### UnbondingDelegationEntry
UnbondingDelegationEntry 定义了一个具有相关元数据的解除绑定对象。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `creation_height` | [int64](#int64) |  | creation_height is the height which the unbonding took place. |
| `completion_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | completion_time is the unix time for unbonding completion. |
| `initial_balance` | [string](#string) |  | initial_balance defines the tokens initially scheduled to receive at completion. |
| `balance` | [string](#string) |  | balance defines the tokens to receive at completion. |






<a name="cosmos.staking.v1beta1.ValAddresses"></a>

### ValAddresses
ValAddresses 定义了一组重复的验证器地址。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `addresses` | [string](#string) | repeated |  |






<a name="cosmos.staking.v1beta1.Validator"></a>

### Validator
Validator 定义了一个validator，连同Validator 的债券份额的总量和它们对硬币的汇率。 削减会导致汇率下降，从而可以正确计算未来的取消委托，而无需迭代委托人。 当代币被委托给这个验证者时，验证者将获得一个委托，其债券份额的数量基于委托代币的数量除以当前汇率。 投票权的计算方法为保税股份总数乘以汇率。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `operator_address` | [string](#string) |  | operator_address defines the address of the validator's operator; bech encoded in JSON. |
| `consensus_pubkey` | [google.protobuf.Any](#google.protobuf.Any) |  | consensus_pubkey is the consensus public key of the validator, as a Protobuf Any. |
| `jailed` | [bool](#bool) |  | jailed defined whether the validator has been jailed from bonded status or not. |
| `status` | [BondStatus](#cosmos.staking.v1beta1.BondStatus) |  | status is the validator status (bonded/unbonding/unbonded). |
| `tokens` | [string](#string) |  | tokens define the delegated tokens (incl. self-delegation). |
| `delegator_shares` | [string](#string) |  | delegator_shares defines total shares issued to a validator's delegators. |
| `description` | [Description](#cosmos.staking.v1beta1.Description) |  | description defines the description terms for the validator. |
| `unbonding_height` | [int64](#int64) |  | unbonding_height defines, if unbonding, the height at which this validator has begun unbonding. |
| `unbonding_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | unbonding_time defines, if unbonding, the min time for the validator to complete unbonding. |
| `commission` | [Commission](#cosmos.staking.v1beta1.Commission) |  | commission defines the commission parameters. |
| `min_self_delegation` | [string](#string) |  | min_self_delegation is the validator's self declared minimum self delegation. |





 <!-- end messages -->


<a name="cosmos.staking.v1beta1.BondStatus"></a>

### BondStatus
BondStatus 是验证者的状态。 

| Name | Number | Description |
| ---- | ------ | ----------- |
| BOND_STATUS_UNSPECIFIED | 0 | UNSPECIFIED defines an invalid validator status. |
| BOND_STATUS_UNBONDED | 1 | UNBONDED defines a validator that is not bonded. |
| BOND_STATUS_UNBONDING | 2 | UNBONDING defines a validator that is unbonding. |
| BOND_STATUS_BONDED | 3 | BONDED defines a validator that is bonded. |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/staking/v1beta1/genesis.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/staking/v1beta1/genesis.proto



<a name="cosmos.staking.v1beta1.GenesisState"></a>

### GenesisState
GenesisState 定义了 staking 模块的创世状态。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.staking.v1beta1.Params) |  | params defines all the paramaters of related to deposit. |
| `last_total_power` | [bytes](#bytes) |  | last_total_power tracks the total amounts of bonded tokens recorded during the previous end block. |
| `last_validator_powers` | [LastValidatorPower](#cosmos.staking.v1beta1.LastValidatorPower) | repeated | last_validator_powers is a special index that provides a historical list of the last-block's bonded validators. |
| `validators` | [Validator](#cosmos.staking.v1beta1.Validator) | repeated | delegations defines the validator set at genesis. |
| `delegations` | [Delegation](#cosmos.staking.v1beta1.Delegation) | repeated | delegations defines the delegations active at genesis. |
| `unbonding_delegations` | [UnbondingDelegation](#cosmos.staking.v1beta1.UnbondingDelegation) | repeated | unbonding_delegations defines the unbonding delegations active at genesis. |
| `redelegations` | [Redelegation](#cosmos.staking.v1beta1.Redelegation) | repeated | redelegations defines the redelegations active at genesis. |
| `exported` | [bool](#bool) |  |  |






<a name="cosmos.staking.v1beta1.LastValidatorPower"></a>

### LastValidatorPower
验证器集更新逻辑所需的 LastValidatorPower。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the address of the validator. |
| `power` | [int64](#int64) |  | power defines the power of the validator. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/staking/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/staking/v1beta1/query.proto



<a name="cosmos.staking.v1beta1.QueryDelegationRequest"></a>

### QueryDelegationRequest
QueryDelegationRequest 是 Query/Delegation RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |






<a name="cosmos.staking.v1beta1.QueryDelegationResponse"></a>

### QueryDelegationResponse
QueryDelegationResponse 是 Query/Delegation RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegation_response` | [DelegationResponse](#cosmos.staking.v1beta1.DelegationResponse) |  | delegation_responses defines the delegation info of a delegation. |






<a name="cosmos.staking.v1beta1.QueryDelegatorDelegationsRequest"></a>

### QueryDelegatorDelegationsRequest
QueryDelegatorDelegationsRequest 是 Query/DelegatorDelegations RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryDelegatorDelegationsResponse"></a>

### QueryDelegatorDelegationsResponse
QueryDelegatorDelegationsResponse 是 Query/DelegatorDelegations RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegation_responses` | [DelegationResponse](#cosmos.staking.v1beta1.DelegationResponse) | repeated | delegation_responses defines all the delegations' info of a delegator. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsRequest"></a>

### QueryDelegatorUnbondingDelegationsRequest
QueryDelegatorUnbondingDelegationsRequest 是 Query/DelegatorUnbondingDelegations RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsResponse"></a>

### QueryDelegatorUnbondingDelegationsResponse
QueryUnbondingDelegatorDelegationsResponse 是 Query/UnbondingDelegatorDelegations RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `unbonding_responses` | [UnbondingDelegation](#cosmos.staking.v1beta1.UnbondingDelegation) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryDelegatorValidatorRequest"></a>

### QueryDelegatorValidatorRequest
QueryDelegatorValidatorRequest 是 Query/DelegatorValidator RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |






<a name="cosmos.staking.v1beta1.QueryDelegatorValidatorResponse"></a>

### QueryDelegatorValidatorResponse
QueryDelegatorValidatorResponse Query/DelegatorValidator RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator` | [Validator](#cosmos.staking.v1beta1.Validator) |  | validator defines the the validator info. |






<a name="cosmos.staking.v1beta1.QueryDelegatorValidatorsRequest"></a>

### QueryDelegatorValidatorsRequest
QueryDelegatorValidatorsRequest 是 Query/DelegatorValidators RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryDelegatorValidatorsResponse"></a>

### QueryDelegatorValidatorsResponse
QueryDelegatorValidatorsResponse 是 Query/DelegatorValidators RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validators` | [Validator](#cosmos.staking.v1beta1.Validator) | repeated | validators defines the the validators' info of a delegator. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryHistoricalInfoRequest"></a>

### QueryHistoricalInfoRequest
QueryHistoricalInfoRequest 是 Query/HistoricalInfo RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  | height defines at which height to query the historical info. |






<a name="cosmos.staking.v1beta1.QueryHistoricalInfoResponse"></a>

### QueryHistoricalInfoResponse
QueryHistoricalInfoResponse 是 Query/HistoricalInfo RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `hist` | [HistoricalInfo](#cosmos.staking.v1beta1.HistoricalInfo) |  | hist defines the historical info at the given height. |






<a name="cosmos.staking.v1beta1.QueryParamsRequest"></a>

### QueryParamsRequest
QueryParamsRequest 是 Query/Params RPC 方法的请求类型。 






<a name="cosmos.staking.v1beta1.QueryParamsResponse"></a>

### QueryParamsResponse
QueryParamsResponse 是 Query/Params RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `params` | [Params](#cosmos.staking.v1beta1.Params) |  | params holds all the parameters of this module. |






<a name="cosmos.staking.v1beta1.QueryPoolRequest"></a>

### QueryPoolRequest
QueryPoolRequest 是 Query/Pool RPC 方法的请求类型。 






<a name="cosmos.staking.v1beta1.QueryPoolResponse"></a>

### QueryPoolResponse
QueryPoolResponse 是 Query/Pool RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `pool` | [Pool](#cosmos.staking.v1beta1.Pool) |  | pool defines the pool info. |






<a name="cosmos.staking.v1beta1.QueryRedelegationsRequest"></a>

### QueryRedelegationsRequest
QueryRedelegationsRequest 是 Query/Redelegations RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `src_validator_addr` | [string](#string) |  | src_validator_addr defines the validator address to redelegate from. |
| `dst_validator_addr` | [string](#string) |  | dst_validator_addr defines the validator address to redelegate to. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryRedelegationsResponse"></a>

### QueryRedelegationsResponse
QueryRedelegationsResponse 是 Query/Redelegations RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `redelegation_responses` | [RedelegationResponse](#cosmos.staking.v1beta1.RedelegationResponse) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryUnbondingDelegationRequest"></a>

### QueryUnbondingDelegationRequest
QueryUnbondingDelegationRequest 是 Query/UnbondingDelegation RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_addr` | [string](#string) |  | delegator_addr defines the delegator address to query for. |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |






<a name="cosmos.staking.v1beta1.QueryUnbondingDelegationResponse"></a>

### QueryUnbondingDelegationResponse
QueryDelegationResponse 是 Query/UnbondingDelegation RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `unbond` | [UnbondingDelegation](#cosmos.staking.v1beta1.UnbondingDelegation) |  | unbond defines the unbonding information of a delegation. |






<a name="cosmos.staking.v1beta1.QueryValidatorDelegationsRequest"></a>

### QueryValidatorDelegationsRequest
QueryValidatorDelegationsRequest 是 Query/ValidatorDelegations RPC 方法的请求类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryValidatorDelegationsResponse"></a>

### QueryValidatorDelegationsResponse
QueryValidatorDelegationsResponse 是 Query/ValidatorDelegations RPC 方法的响应类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegation_responses` | [DelegationResponse](#cosmos.staking.v1beta1.DelegationResponse) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryValidatorRequest"></a>

### QueryValidatorRequest
QueryValidatorRequest 是 Query/Validator RPC 方法的响应类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |






<a name="cosmos.staking.v1beta1.QueryValidatorResponse"></a>

### QueryValidatorResponse
QueryValidatorResponse 是 Query/Validator RPC 方法的响应类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator` | [Validator](#cosmos.staking.v1beta1.Validator) |  | validator defines the the validator info. |






<a name="cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsRequest"></a>

### QueryValidatorUnbondingDelegationsRequest
QueryValidatorUnbondingDelegationsRequest 是 Query/ValidatorUnbondingDelegations RPC 方法的必需类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validator_addr` | [string](#string) |  | validator_addr defines the validator address to query for. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsResponse"></a>

### QueryValidatorUnbondingDelegationsResponse
QueryValidatorUnbondingDelegationsResponse 是 Query/ValidatorUnbondingDelegations RPC 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `unbonding_responses` | [UnbondingDelegation](#cosmos.staking.v1beta1.UnbondingDelegation) | repeated |  |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |






<a name="cosmos.staking.v1beta1.QueryValidatorsRequest"></a>

### QueryValidatorsRequest
QueryValidatorsRequest 是 Query/Validators RPC 方法的请求类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `status` | [string](#string) |  | status enables to query for validators matching a given status. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an optional pagination for the request. |






<a name="cosmos.staking.v1beta1.QueryValidatorsResponse"></a>

### QueryValidatorsResponse
QueryValidatorsResponse 是 Query/Validators RPC 方法的响应类型 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `validators` | [Validator](#cosmos.staking.v1beta1.Validator) | repeated | validators contains all the queried validators. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines the pagination in the response. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.staking.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 查询器服务。 

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Validators` | [QueryValidatorsRequest](#cosmos.staking.v1beta1.QueryValidatorsRequest) | [QueryValidatorsResponse](#cosmos.staking.v1beta1.QueryValidatorsResponse) | Validators queries all validators that match the given status. | GET|/cosmos/staking/v1beta1/validators|
| `Validator` | [QueryValidatorRequest](#cosmos.staking.v1beta1.QueryValidatorRequest) | [QueryValidatorResponse](#cosmos.staking.v1beta1.QueryValidatorResponse) | Validator queries validator info for given validator address. | GET|/cosmos/staking/v1beta1/validators/{validator_addr}|
| `ValidatorDelegations` | [QueryValidatorDelegationsRequest](#cosmos.staking.v1beta1.QueryValidatorDelegationsRequest) | [QueryValidatorDelegationsResponse](#cosmos.staking.v1beta1.QueryValidatorDelegationsResponse) | ValidatorDelegations queries delegate info for given validator. | GET|/cosmos/staking/v1beta1/validators/{validator_addr}/delegations|
| `ValidatorUnbondingDelegations` | [QueryValidatorUnbondingDelegationsRequest](#cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsRequest) | [QueryValidatorUnbondingDelegationsResponse](#cosmos.staking.v1beta1.QueryValidatorUnbondingDelegationsResponse) | ValidatorUnbondingDelegations queries unbonding delegations of a validator. | GET|/cosmos/staking/v1beta1/validators/{validator_addr}/unbonding_delegations|
| `Delegation` | [QueryDelegationRequest](#cosmos.staking.v1beta1.QueryDelegationRequest) | [QueryDelegationResponse](#cosmos.staking.v1beta1.QueryDelegationResponse) | Delegation queries delegate info for given validator delegator pair. | GET|/cosmos/staking/v1beta1/validators/{validator_addr}/delegations/{delegator_addr}|
| `UnbondingDelegation` | [QueryUnbondingDelegationRequest](#cosmos.staking.v1beta1.QueryUnbondingDelegationRequest) | [QueryUnbondingDelegationResponse](#cosmos.staking.v1beta1.QueryUnbondingDelegationResponse) | UnbondingDelegation queries unbonding info for given validator delegator pair. | GET|/cosmos/staking/v1beta1/validators/{validator_addr}/delegations/{delegator_addr}/unbonding_delegation|
| `DelegatorDelegations` | [QueryDelegatorDelegationsRequest](#cosmos.staking.v1beta1.QueryDelegatorDelegationsRequest) | [QueryDelegatorDelegationsResponse](#cosmos.staking.v1beta1.QueryDelegatorDelegationsResponse) | DelegatorDelegations queries all delegations of a given delegator address. | GET|/cosmos/staking/v1beta1/delegations/{delegator_addr}|
| `DelegatorUnbondingDelegations` | [QueryDelegatorUnbondingDelegationsRequest](#cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsRequest) | [QueryDelegatorUnbondingDelegationsResponse](#cosmos.staking.v1beta1.QueryDelegatorUnbondingDelegationsResponse) | DelegatorUnbondingDelegations queries all unbonding delegations of a given delegator address. | GET|/cosmos/staking/v1beta1/delegators/{delegator_addr}/unbonding_delegations|
| `Redelegations` | [QueryRedelegationsRequest](#cosmos.staking.v1beta1.QueryRedelegationsRequest) | [QueryRedelegationsResponse](#cosmos.staking.v1beta1.QueryRedelegationsResponse) | Redelegations queries redelegations of given address. | GET|/cosmos/staking/v1beta1/delegators/{delegator_addr}/redelegations|
| `DelegatorValidators` | [QueryDelegatorValidatorsRequest](#cosmos.staking.v1beta1.QueryDelegatorValidatorsRequest) | [QueryDelegatorValidatorsResponse](#cosmos.staking.v1beta1.QueryDelegatorValidatorsResponse) | DelegatorValidators queries all validators info for given delegator address. | GET|/cosmos/staking/v1beta1/delegators/{delegator_addr}/validators|
| `DelegatorValidator` | [QueryDelegatorValidatorRequest](#cosmos.staking.v1beta1.QueryDelegatorValidatorRequest) | [QueryDelegatorValidatorResponse](#cosmos.staking.v1beta1.QueryDelegatorValidatorResponse) | DelegatorValidator queries validator info for given delegator validator pair. | GET|/cosmos/staking/v1beta1/delegators/{delegator_addr}/validators/{validator_addr}|
| `HistoricalInfo` | [QueryHistoricalInfoRequest](#cosmos.staking.v1beta1.QueryHistoricalInfoRequest) | [QueryHistoricalInfoResponse](#cosmos.staking.v1beta1.QueryHistoricalInfoResponse) | HistoricalInfo queries the historical info for given height. | GET|/cosmos/staking/v1beta1/historical_info/{height}|
| `Pool` | [QueryPoolRequest](#cosmos.staking.v1beta1.QueryPoolRequest) | [QueryPoolResponse](#cosmos.staking.v1beta1.QueryPoolResponse) | Pool queries the pool info. | GET|/cosmos/staking/v1beta1/pool|
| `Params` | [QueryParamsRequest](#cosmos.staking.v1beta1.QueryParamsRequest) | [QueryParamsResponse](#cosmos.staking.v1beta1.QueryParamsResponse) | Parameters queries the staking parameters. | GET|/cosmos/staking/v1beta1/params|

 <!-- end services -->



<a name="cosmos/staking/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/staking/v1beta1/tx.proto



<a name="cosmos.staking.v1beta1.MsgBeginRedelegate"></a>

### MsgBeginRedelegate
MsgBeginRedelegate 定义了一条 SDK 消息，用于将代币从委托人和源验证器重新委托到目标验证器。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_src_address` | [string](#string) |  |  |
| `validator_dst_address` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  |  |






<a name="cosmos.staking.v1beta1.MsgBeginRedelegateResponse"></a>

### MsgBeginRedelegateResponse
MsgBeginRedelegateResponse 定义了 Msg/BeginRedelegate 响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `completion_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |






<a name="cosmos.staking.v1beta1.MsgCreateValidator"></a>

### MsgCreateValidator
MsgCreateValidator 定义了用于创建新验证器的 SDK 消息。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `description` | [Description](#cosmos.staking.v1beta1.Description) |  |  |
| `commission` | [CommissionRates](#cosmos.staking.v1beta1.CommissionRates) |  |  |
| `min_self_delegation` | [string](#string) |  |  |
| `delegator_address` | [string](#string) |  |  |
| `validator_address` | [string](#string) |  |  |
| `pubkey` | [google.protobuf.Any](#google.protobuf.Any) |  |  |
| `value` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  |  |






<a name="cosmos.staking.v1beta1.MsgCreateValidatorResponse"></a>

### MsgCreateValidatorResponse
MsgCreateValidatorResponse 定义了 Msg/CreateValidator 响应类型。 






<a name="cosmos.staking.v1beta1.MsgDelegate"></a>

### MsgDelegate
MsgDelegate 定义了一条 SDK 消息，用于执行从委托人到验证人的代币委托。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_address` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  |  |






<a name="cosmos.staking.v1beta1.MsgDelegateResponse"></a>

### MsgDelegateResponse
Msg Delegate Response 定义了Message/Delegate 响应类型。 






<a name="cosmos.staking.v1beta1.MsgEditValidator"></a>

### MsgEditValidator
MsgEditValidator 定义了用于编辑现有验证器的 SDK 消息。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `description` | [Description](#cosmos.staking.v1beta1.Description) |  |  |
| `validator_address` | [string](#string) |  |  |
| `commission_rate` | [string](#string) |  | We pass a reference to the new commission rate and min self delegation as it's not mandatory to update. If not updated, the deserialized rate will be zero with no way to distinguish if an update was intended. REF: #2373 |
| `min_self_delegation` | [string](#string) |  |  |






<a name="cosmos.staking.v1beta1.MsgEditValidatorResponse"></a>

### MsgEditValidatorResponse
MsgEditValidatorResponse 定义了 Msg/EditValidator 响应类型。 






<a name="cosmos.staking.v1beta1.MsgUndelegate"></a>

### MsgUndelegate
MsgUndelegate 定义了一条 SDK 消息，用于执行来自委托和验证器的取消委托。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `delegator_address` | [string](#string) |  |  |
| `validator_address` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) |  |  |






<a name="cosmos.staking.v1beta1.MsgUndelegateResponse"></a>

### MsgUndelegateResponse
MsgUndelegateResponse 定义了 Msg/Undelegate 响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `completion_time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.staking.v1beta1.Msg"></a>

### Msg
Msg 定义了质押 Msg 服务。 

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `CreateValidator` | [MsgCreateValidator](#cosmos.staking.v1beta1.MsgCreateValidator) | [MsgCreateValidatorResponse](#cosmos.staking.v1beta1.MsgCreateValidatorResponse) | CreateValidator defines a method for creating a new validator. | |
| `EditValidator` | [MsgEditValidator](#cosmos.staking.v1beta1.MsgEditValidator) | [MsgEditValidatorResponse](#cosmos.staking.v1beta1.MsgEditValidatorResponse) | EditValidator defines a method for editing an existing validator. | |
| `Delegate` | [MsgDelegate](#cosmos.staking.v1beta1.MsgDelegate) | [MsgDelegateResponse](#cosmos.staking.v1beta1.MsgDelegateResponse) | Delegate defines a method for performing a delegation of coins from a delegator to a validator. | |
| `BeginRedelegate` | [MsgBeginRedelegate](#cosmos.staking.v1beta1.MsgBeginRedelegate) | [MsgBeginRedelegateResponse](#cosmos.staking.v1beta1.MsgBeginRedelegateResponse) | BeginRedelegate defines a method for performing a redelegation of coins from a delegator and source validator to a destination validator. | |
| `Undelegate` | [MsgUndelegate](#cosmos.staking.v1beta1.MsgUndelegate) | [MsgUndelegateResponse](#cosmos.staking.v1beta1.MsgUndelegateResponse) | Undelegate defines a method for performing an undelegation from a delegate and a validator. | |

 <!-- end services -->



<a name="cosmos/tx/signing/v1beta1/signing.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/tx/signing/v1beta1/signing.proto



<a name="cosmos.tx.signing.v1beta1.SignatureDescriptor"></a>

### SignatureDescriptor
SignatureDescriptor 是一种方便的类型，它表示签名的完整数据，包括签名者的公钥、签名模式和签名本身。 它主要用于协调客户端之间的签名。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `public_key` | [google.protobuf.Any](#google.protobuf.Any) |  | public_key is the public key of the signer |
| `data` | [SignatureDescriptor.Data](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data) |  |  |
| `sequence` | [uint64](#uint64) |  | sequence is the sequence of the account, which describes the number of committed transactions signed by a given address. It is used to prevent replay attacks. |






<a name="cosmos.tx.signing.v1beta1.SignatureDescriptor.Data"></a>

### SignatureDescriptor.Data
数据代表签名数据 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `single` | [SignatureDescriptor.Data.Single](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Single) |  | single represents a single signer |
| `multi` | [SignatureDescriptor.Data.Multi](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Multi) |  | multi represents a multisig signer |






<a name="cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Multi"></a>

### SignatureDescriptor.Data.Multi
Multi 是多重签名公钥的签名数据 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `bitarray` | [cosmos.crypto.multisig.v1beta1.CompactBitArray](#cosmos.crypto.multisig.v1beta1.CompactBitArray) |  | bitarray specifies which keys within the multisig are signing |
| `signatures` | [SignatureDescriptor.Data](#cosmos.tx.signing.v1beta1.SignatureDescriptor.Data) | repeated | signatures is the signatures of the multi-signature |






<a name="cosmos.tx.signing.v1beta1.SignatureDescriptor.Data.Single"></a>

### SignatureDescriptor.Data.Single
Single 是单个签名者的签名数据


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `mode` | [SignMode](#cosmos.tx.signing.v1beta1.SignMode) |  | mode is the signing mode of the single signer |
| `signature` | [bytes](#bytes) |  | signature is the raw signature bytes |






<a name="cosmos.tx.signing.v1beta1.SignatureDescriptors"></a>

### SignatureDescriptors
SignatureDescriptors 包装多个 SignatureDescriptor。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `signatures` | [SignatureDescriptor](#cosmos.tx.signing.v1beta1.SignatureDescriptor) | repeated | signatures are the signature descriptors |





 <!-- end messages -->


<a name="cosmos.tx.signing.v1beta1.SignMode"></a>

### SignMode
SignMode 代表一种具有自身安全保证的签名模式。

| Name | Number | Description |
| ---- | ------ | ----------- |
| SIGN_MODE_UNSPECIFIED | 0 | SIGN_MODE_UNSPECIFIED specifies an unknown signing mode and will be rejected. |
| SIGN_MODE_DIRECT | 1 | SIGN_MODE_DIRECT specifies a signing mode which uses SignDoc and is verified with raw bytes from Tx. |
| SIGN_MODE_TEXTUAL | 2 | SIGN_MODE_TEXTUAL is a future signing mode that will verify some human-readable textual representation on top of the binary representation from SIGN_MODE_DIRECT. It is currently not supported. |
| SIGN_MODE_DIRECT_AUX | 3 | SIGN_MODE_DIRECT_AUX specifies a signing mode which uses SignDocDirectAux. As opposed to SIGN_MODE_DIRECT, this sign mode does not require signers signing over other signers' `signer_info`. It also allows for adding Tips in transactions. |
| SIGN_MODE_LEGACY_AMINO_JSON | 127 | SIGN_MODE_LEGACY_AMINO_JSON is a backwards compatibility mode which uses Amino JSON and will be removed in the future. |


 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/tx/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/tx/v1beta1/tx.proto



<a name="cosmos.tx.v1beta1.AuthInfo"></a>

### AuthInfo
AuthInfo 描述了用于签署交易的费用和签名者模式。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `signer_infos` | [SignerInfo](#cosmos.tx.v1beta1.SignerInfo) | repeated | signer_infos defines the signing modes for the required signers. The number and order of elements must match the required signers from TxBody's messages. The first element is the primary signer and the one which pays the fee. |
| `fee` | [Fee](#cosmos.tx.v1beta1.Fee) |  | Fee is the fee and gas limit for the transaction. The first signer is the primary signer and the one which pays the fee. The fee can be calculated based on the cost of evaluating the body and doing signature verification of the signers. This can be estimated via simulation. |
| `tip` | [Tip](#cosmos.tx.v1beta1.Tip) |  | Tip is the optional tip used for meta-transactions.

Since: cosmos-sdk 0.45 |






<a name="cosmos.tx.v1beta1.AuxSignerData"></a>

### AuxSignerData
AuxSignerData 是辅助签名者(例如倾销者)构建并发送给费用支付者(谁将构建和广播实际 tx)的中间格式。 AuxSignerData 本身不是有效的 tx，如果直接按原样发送，将被节点拒绝。

Since: cosmos-sdk 0.45


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `address` | [string](#string) |  | address is the bech32-encoded address of the auxiliary signer. If using AuxSignerData across different chains, the bech32 prefix of the target chain (where the final transaction is broadcasted) should be used. |
| `sign_doc` | [SignDocDirectAux](#cosmos.tx.v1beta1.SignDocDirectAux) |  | sign_doc is the SIGN_MOD_DIRECT_AUX sign doc that the auxiliary signer signs. Note: we use the same sign doc even if we're signing with LEGACY_AMINO_JSON. |
| `mode` | [cosmos.tx.signing.v1beta1.SignMode](#cosmos.tx.signing.v1beta1.SignMode) |  | mode is the signing mode of the single signer |
| `sig` | [bytes](#bytes) |  | sig is the signature of the sign doc. |






<a name="cosmos.tx.v1beta1.Fee"></a>

### Fee
费用包括以费用支付的硬币数量和交易使用的最大天然气量。 该比率产生一个有效的“gasprice”，它必须高于某个最小值才能被接受到内存池中。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | amount is the amount of coins to be paid as a fee |
| `gas_limit` | [uint64](#uint64) |  | gas_limit is the maximum gas that can be used in transaction processing before an out of gas error occurs |
| `payer` | [string](#string) |  | if unset, the first signer is responsible for paying the fees. If set, the specified account must pay the fees. the payer must be a tx signer (and thus have signed this field in AuthInfo). setting this field does *not* change the ordering of required signers for the transaction. |
| `granter` | [string](#string) |  | if set, the fee payer (either the first signer or the value of the payer field) requests that a fee grant be used to pay fees instead of the fee payer's own balance. If an appropriate fee grant does not exist or the chain does not support fee grants, this will fail |






<a name="cosmos.tx.v1beta1.ModeInfo"></a>

### ModeInfo
ModeInfo 描述了单个或嵌套多重签名者的签名模式。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `single` | [ModeInfo.Single](#cosmos.tx.v1beta1.ModeInfo.Single) |  | single represents a single signer |
| `multi` | [ModeInfo.Multi](#cosmos.tx.v1beta1.ModeInfo.Multi) |  | multi represents a nested multisig signer |






<a name="cosmos.tx.v1beta1.ModeInfo.Multi"></a>

### ModeInfo.Multi
Multi 是 multisig 公钥的模式信息


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `bitarray` | [cosmos.crypto.multisig.v1beta1.CompactBitArray](#cosmos.crypto.multisig.v1beta1.CompactBitArray) |  | bitarray specifies which keys within the multisig are signing |
| `mode_infos` | [ModeInfo](#cosmos.tx.v1beta1.ModeInfo) | repeated | mode_infos is the corresponding modes of the signers of the multisig which could include nested multisig public keys |






<a name="cosmos.tx.v1beta1.ModeInfo.Single"></a>

### ModeInfo.Single
Single 是单个签名者的模式信息。 它被构造为一条消息，以允许将来使用其他字段，例如 SIGN_MODE_TEXTUAL 的语言环境


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `mode` | [cosmos.tx.signing.v1beta1.SignMode](#cosmos.tx.signing.v1beta1.SignMode) |  | mode is the signing mode of the single signer |






<a name="cosmos.tx.v1beta1.SignDoc"></a>

### SignDoc
SignDoc 是用于为 SIGN_MODE_DIRECT 生成符号字节的类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `body_bytes` | [bytes](#bytes) |  | body_bytes is protobuf serialization of a TxBody that matches the representation in TxRaw. |
| `auth_info_bytes` | [bytes](#bytes) |  | auth_info_bytes is a protobuf serialization of an AuthInfo that matches the representation in TxRaw. |
| `chain_id` | [string](#string) |  | chain_id is the unique identifier of the chain this transaction targets. It prevents signed transactions from being used on another chain by an attacker |
| `account_number` | [uint64](#uint64) |  | account_number is the account number of the account in state |






<a name="cosmos.tx.v1beta1.SignDocDirectAux"></a>

### SignDocDirectAux
SignDocDirectAux 是用于为 SIGN_MODE_DIRECT_AUX 生成符号字节的类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `body_bytes` | [bytes](#bytes) |  | body_bytes is protobuf serialization of a TxBody that matches the representation in TxRaw. |
| `public_key` | [google.protobuf.Any](#google.protobuf.Any) |  | public_key is the public key of the signing account. |
| `chain_id` | [string](#string) |  | chain_id is the identifier of the chain this transaction targets. It prevents signed transactions from being used on another chain by an attacker. |
| `account_number` | [uint64](#uint64) |  | account_number is the account number of the account in state. |
| `sequence` | [uint64](#uint64) |  | sequence is the sequence number of the signing account. |
| `tip` | [Tip](#cosmos.tx.v1beta1.Tip) |  | Tip is the optional tip used for meta-transactions. It should be left empty if the signer is not the tipper for this transaction. |






<a name="cosmos.tx.v1beta1.SignerInfo"></a>

### SignerInfo
SignerInfo 描述了单个顶级签名者的公钥和签名模式。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `public_key` | [google.protobuf.Any](#google.protobuf.Any) |  | public_key is the public key of the signer. It is optional for accounts that already exist in state. If unset, the verifier can use the required \ signer address for this position and lookup the public key. |
| `mode_info` | [ModeInfo](#cosmos.tx.v1beta1.ModeInfo) |  | mode_info describes the signing mode of the signer and is a nested structure to support nested multisig pubkey's |
| `sequence` | [uint64](#uint64) |  | sequence is the sequence of the account, which describes the number of committed transactions signed by a given address. It is used to prevent replay attacks. |






<a name="cosmos.tx.v1beta1.Tip"></a>

### Tip
Tip 是用于元交易的提示。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated | amount is the amount of the tip |
| `tipper` | [string](#string) |  | tipper is the address of the account paying for the tip |






<a name="cosmos.tx.v1beta1.Tx"></a>

### Tx
Tx 是用于广播交易的标准类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `body` | [TxBody](#cosmos.tx.v1beta1.TxBody) |  | body is the processable content of the transaction |
| `auth_info` | [AuthInfo](#cosmos.tx.v1beta1.AuthInfo) |  | auth_info is the authorization related content of the transaction, specifically signers, signer modes and fee |
| `signatures` | [bytes](#bytes) | repeated | signatures is a list of signatures that matches the length and order of AuthInfo's signer_infos to allow connecting signature meta information like public key and signing mode by position. |






<a name="cosmos.tx.v1beta1.TxBody"></a>

### TxBody
TxBody 是所有签名者都签署的交易主体。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `messages` | [google.protobuf.Any](#google.protobuf.Any) | repeated | messages is a list of messages to be executed. The required signers of those messages define the number and order of elements in AuthInfo's signer_infos and Tx's signatures. Each required signer address is added to the list only the first time it occurs. By convention, the first required signer (usually from the first message) is referred to as the primary signer and pays the fee for the whole transaction. |
| `memo` | [string](#string) |  | memo is any arbitrary note/comment to be added to the transaction. WARNING: in clients, any publicly exposed text should not be called memo, but should be called `note` instead (see https://github.com/cosmos/cosmos-sdk/issues/9122). |
| `timeout_height` | [uint64](#uint64) |  | timeout is the block height after which this transaction will not be processed by the chain |
| `extension_options` | [google.protobuf.Any](#google.protobuf.Any) | repeated | extension_options are arbitrary options that can be added by chains when the default options are not sufficient. If any of these are present and can't be handled, the transaction will be rejected |
| `non_critical_extension_options` | [google.protobuf.Any](#google.protobuf.Any) | repeated | extension_options are arbitrary options that can be added by chains when the default options are not sufficient. If any of these are present and can't be handled, they will be ignored |






<a name="cosmos.tx.v1beta1.TxRaw"></a>

### TxRaw
TxRaw 是 Tx 的一个变体，它固定签名者的 body 和 auth_info 的精确二进制表示。 这用于签名、广播和验证。 二进制 `serialize(tx: TxRaw)`存储在 Tendermint 中，哈希 `sha256(serialize(tx: TxRaw))`存储在 Tendermint 中，哈希 "txhash",通常用作交易 ID。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `body_bytes` | [bytes](#bytes) |  | body_bytes is a protobuf serialization of a TxBody that matches the representation in SignDoc. |
| `auth_info_bytes` | [bytes](#bytes) |  | auth_info_bytes is a protobuf serialization of an AuthInfo that matches the representation in SignDoc. |
| `signatures` | [bytes](#bytes) | repeated | signatures is a list of signatures that matches the length and order of AuthInfo's signer_infos to allow connecting signature meta information like public key and signing mode by position. |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/tx/v1beta1/service.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/tx/v1beta1/service.proto



<a name="cosmos.tx.v1beta1.BroadcastTxRequest"></a>

### BroadcastTxRequest
BroadcastTxRequest 是 Service.BroadcastTxRequest RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tx_bytes` | [bytes](#bytes) |  | tx_bytes is the raw transaction. |
| `mode` | [BroadcastMode](#cosmos.tx.v1beta1.BroadcastMode) |  |  |






<a name="cosmos.tx.v1beta1.BroadcastTxResponse"></a>

### BroadcastTxResponse
BroadcastTxResponse 是 Service.BroadcastTx 方法的响应类型。 


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tx_response` | [cosmos.base.abci.v1beta1.TxResponse](#cosmos.base.abci.v1beta1.TxResponse) |  | tx_response is the queried TxResponses. |






<a name="cosmos.tx.v1beta1.GetTxRequest"></a>

### GetTxRequest
GetTxRequest 是 Service.GetTx RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `hash` | [string](#string) |  | hash is the tx hash to query, encoded as a hex string. |






<a name="cosmos.tx.v1beta1.GetTxResponse"></a>

### GetTxResponse
GetTxResponse 是 Service.GetTx 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tx` | [Tx](#cosmos.tx.v1beta1.Tx) |  | tx is the queried transaction. |
| `tx_response` | [cosmos.base.abci.v1beta1.TxResponse](#cosmos.base.abci.v1beta1.TxResponse) |  | tx_response is the queried TxResponses. |






<a name="cosmos.tx.v1beta1.GetTxsEventRequest"></a>

### GetTxsEventRequest
GetTxsEventRequest 是 Service.TxsByEvents RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `events` | [string](#string) | repeated | events is the list of transaction event type. |
| `pagination` | [cosmos.base.query.v1beta1.PageRequest](#cosmos.base.query.v1beta1.PageRequest) |  | pagination defines an pagination for the request. |
| `order_by` | [OrderBy](#cosmos.tx.v1beta1.OrderBy) |  |  |






<a name="cosmos.tx.v1beta1.GetTxsEventResponse"></a>

### GetTxsEventResponse
GetTxsEventResponse 是 Service.TxsByEvents RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `txs` | [Tx](#cosmos.tx.v1beta1.Tx) | repeated | txs is the list of queried transactions. |
| `tx_responses` | [cosmos.base.abci.v1beta1.TxResponse](#cosmos.base.abci.v1beta1.TxResponse) | repeated | tx_responses is the list of queried TxResponses. |
| `pagination` | [cosmos.base.query.v1beta1.PageResponse](#cosmos.base.query.v1beta1.PageResponse) |  | pagination defines an pagination for the response. |






<a name="cosmos.tx.v1beta1.SimulateRequest"></a>

### SimulateRequest
SimulateRequest 是 Service.Simulate RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `tx` | [Tx](#cosmos.tx.v1beta1.Tx) |  | **Deprecated.** tx is the transaction to simulate. Deprecated. Send raw tx bytes instead. |
| `tx_bytes` | [bytes](#bytes) |  | tx_bytes is the raw transaction.

Since: cosmos-sdk 0.43 |






<a name="cosmos.tx.v1beta1.SimulateResponse"></a>

### SimulateResponse
SimulateResponse 是 Service.SimulateRPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `gas_info` | [cosmos.base.abci.v1beta1.GasInfo](#cosmos.base.abci.v1beta1.GasInfo) |  | gas_info is the information about gas used in the simulation. |
| `result` | [cosmos.base.abci.v1beta1.Result](#cosmos.base.abci.v1beta1.Result) |  | result is the result of the simulation. |





 <!-- end messages -->


<a name="cosmos.tx.v1beta1.BroadcastMode"></a>

### BroadcastMode
BroadcastMode 指定 TxService.Broadcast RPC 方法的广播模式。

| Name | Number | Description |
| ---- | ------ | ----------- |
| BROADCAST_MODE_UNSPECIFIED | 0 | zero-value for mode ordering |
| BROADCAST_MODE_BLOCK | 1 | BROADCAST_MODE_BLOCK defines a tx broadcasting mode where the client waits for the tx to be committed in a block. |
| BROADCAST_MODE_SYNC | 2 | BROADCAST_MODE_SYNC defines a tx broadcasting mode where the client waits for a CheckTx execution response only. |
| BROADCAST_MODE_ASYNC | 3 | BROADCAST_MODE_ASYNC defines a tx broadcasting mode where the client returns immediately. |



<a name="cosmos.tx.v1beta1.OrderBy"></a>

### OrderBy
OrderBy 定义排序顺序

| Name | Number | Description |
| ---- | ------ | ----------- |
| ORDER_BY_UNSPECIFIED | 0 | ORDER_BY_UNSPECIFIED specifies an unknown sorting order. OrderBy defaults to ASC in this case. |
| ORDER_BY_ASC | 1 | ORDER_BY_ASC defines ascending order |
| ORDER_BY_DESC | 2 | ORDER_BY_DESC defines descending order |


 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.tx.v1beta1.Service"></a>

### Service
Service 定义了一个用于与事务交互的 gRPC 服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `Simulate` | [SimulateRequest](#cosmos.tx.v1beta1.SimulateRequest) | [SimulateResponse](#cosmos.tx.v1beta1.SimulateResponse) | Simulate simulates executing a transaction for estimating gas usage. | POST|/cosmos/tx/v1beta1/simulate|
| `GetTx` | [GetTxRequest](#cosmos.tx.v1beta1.GetTxRequest) | [GetTxResponse](#cosmos.tx.v1beta1.GetTxResponse) | GetTx fetches a tx by hash. | GET|/cosmos/tx/v1beta1/txs/{hash}|
| `BroadcastTx` | [BroadcastTxRequest](#cosmos.tx.v1beta1.BroadcastTxRequest) | [BroadcastTxResponse](#cosmos.tx.v1beta1.BroadcastTxResponse) | BroadcastTx broadcast transaction. | POST|/cosmos/tx/v1beta1/txs|
| `GetTxsEvent` | [GetTxsEventRequest](#cosmos.tx.v1beta1.GetTxsEventRequest) | [GetTxsEventResponse](#cosmos.tx.v1beta1.GetTxsEventResponse) | GetTxsEvent fetches txs by event. | GET|/cosmos/tx/v1beta1/txs|

 <!-- end services -->



<a name="cosmos/upgrade/v1beta1/upgrade.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/upgrade/v1beta1/upgrade.proto



<a name="cosmos.upgrade.v1beta1.CancelSoftwareUpgradeProposal"></a>

### CancelSoftwareUpgradeProposal
CancelSoftwareUpgradeProposal 是一种用于取消软件升级的 gov 内容类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |






<a name="cosmos.upgrade.v1beta1.ModuleVersion"></a>

### ModuleVersion
ModuleVersion 指定一个模块及其共识版本。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | name of the app module |
| `version` | [uint64](#uint64) |  | consensus version of the app module |






<a name="cosmos.upgrade.v1beta1.Plan"></a>

### Plan
计划指定有关计划升级及其发生时间的信息。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | Sets the name for the upgrade. This name will be used by the upgraded version of the software to apply any special "on-upgrade" commands during the first BeginBlock method after the upgrade is applied. It is also used to detect whether a software version can handle a given upgrade. If no upgrade handler with this name has been set in the software, it will be assumed that the software is out-of-date when the upgrade Time or Height is reached and the software will exit. |
| `time` | [google.protobuf.Timestamp](#google.protobuf.Timestamp) |  | **Deprecated.** Deprecated: Time based upgrades have been deprecated. Time based upgrade logic has been removed from the SDK. If this field is not empty, an error will be thrown. |
| `height` | [int64](#int64) |  | The height at which the upgrade must be performed. Only used if Time is not set. |
| `info` | [string](#string) |  | Any application specific upgrade info to be included on-chain such as a git commit that validators could automatically upgrade to |
| `upgraded_client_state` | [google.protobuf.Any](#google.protobuf.Any) |  | **Deprecated.** Deprecated: UpgradedClientState field has been deprecated. IBC upgrade logic has been moved to the IBC module in the sub module 02-client. If this field is not empty, an error will be thrown. |






<a name="cosmos.upgrade.v1beta1.SoftwareUpgradeProposal"></a>

### SoftwareUpgradeProposal
SoftwareUpgradeProposal 是用于启动软件升级的 gov 内容类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `title` | [string](#string) |  |  |
| `description` | [string](#string) |  |  |
| `plan` | [Plan](#cosmos.upgrade.v1beta1.Plan) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/upgrade/v1beta1/query.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/upgrade/v1beta1/query.proto



<a name="cosmos.upgrade.v1beta1.QueryAppliedPlanRequest"></a>

### QueryAppliedPlanRequest
QueryCurrentPlanRequest 是 Query/AppliedPlan RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `name` | [string](#string) |  | name is the name of the applied plan to query for. |






<a name="cosmos.upgrade.v1beta1.QueryAppliedPlanResponse"></a>

### QueryAppliedPlanResponse
QueryAppliedPlanResponse 是 Query/AppliedPlan RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `height` | [int64](#int64) |  | height is the block height at which the plan was applied. |






<a name="cosmos.upgrade.v1beta1.QueryCurrentPlanRequest"></a>

### QueryCurrentPlanRequest
QueryCurrentPlanRequest 是 Query/CurrentPlan RPC 方法的请求类型。






<a name="cosmos.upgrade.v1beta1.QueryCurrentPlanResponse"></a>

### QueryCurrentPlanResponse
QueryCurrentPlanResponse 是 Query/CurrentPlan RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `plan` | [Plan](#cosmos.upgrade.v1beta1.Plan) |  | plan is the current upgrade plan. |






<a name="cosmos.upgrade.v1beta1.QueryModuleVersionsRequest"></a>

### QueryModuleVersionsRequest
QueryModuleVersionsRequest 是 Query/ModuleVersions RPC 方法的请求类型。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `module_name` | [string](#string) |  | module_name is a field to query a specific module consensus version from state. Leaving this empty will fetch the full list of module versions from state |






<a name="cosmos.upgrade.v1beta1.QueryModuleVersionsResponse"></a>

### QueryModuleVersionsResponse
QueryModuleVersionsResponse 是 Query/ModuleVersions RPC 方法的响应类型。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `module_versions` | [ModuleVersion](#cosmos.upgrade.v1beta1.ModuleVersion) | repeated | module_versions is a list of module names with their consensus versions. |






<a name="cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateRequest"></a>

### QueryUpgradedConsensusStateRequest
QueryUpgradedConsensusStateRequest 是 Query/UpgradedConsensusState RPC 方法的请求类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `last_height` | [int64](#int64) |  | last height of the current chain must be sent in request as this is the height under which next consensus state is stored |






<a name="cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateResponse"></a>

### QueryUpgradedConsensusStateResponse
QueryUpgradedConsensusStateResponse 是 Query/UpgradedConsensusState RPC 方法的响应类型。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `upgraded_consensus_state` | [bytes](#bytes) |  | Since: cosmos-sdk 0.43 |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.upgrade.v1beta1.Query"></a>

### Query
Query 定义了 gRPC 升级查询器服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `CurrentPlan` | [QueryCurrentPlanRequest](#cosmos.upgrade.v1beta1.QueryCurrentPlanRequest) | [QueryCurrentPlanResponse](#cosmos.upgrade.v1beta1.QueryCurrentPlanResponse) | CurrentPlan queries the current upgrade plan. | GET|/cosmos/upgrade/v1beta1/current_plan|
| `AppliedPlan` | [QueryAppliedPlanRequest](#cosmos.upgrade.v1beta1.QueryAppliedPlanRequest) | [QueryAppliedPlanResponse](#cosmos.upgrade.v1beta1.QueryAppliedPlanResponse) | AppliedPlan queries a previously applied upgrade plan by its name. | GET|/cosmos/upgrade/v1beta1/applied_plan/{name}|
| `UpgradedConsensusState` | [QueryUpgradedConsensusStateRequest](#cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateRequest) | [QueryUpgradedConsensusStateResponse](#cosmos.upgrade.v1beta1.QueryUpgradedConsensusStateResponse) | UpgradedConsensusState queries the consensus state that will serve as a trusted kernel for the next version of this chain. It will only be stored at the last height of this chain. UpgradedConsensusState RPC not supported with legacy querier This rpc is deprecated now that IBC has its own replacement (https://github.com/cosmos/ibc-go/blob/2c880a22e9f9cc75f62b527ca94aa75ce1106001/proto/ibc/core/client/v1/query.proto#L54) | GET|/cosmos/upgrade/v1beta1/upgraded_consensus_state/{last_height}|
| `ModuleVersions` | [QueryModuleVersionsRequest](#cosmos.upgrade.v1beta1.QueryModuleVersionsRequest) | [QueryModuleVersionsResponse](#cosmos.upgrade.v1beta1.QueryModuleVersionsResponse) | ModuleVersions queries the list of module versions from state.

Since: cosmos-sdk 0.43 | GET|/cosmos/upgrade/v1beta1/module_versions|

 <!-- end services -->



<a name="cosmos/vesting/v1beta1/vesting.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/vesting/v1beta1/vesting.proto



<a name="cosmos.vesting.v1beta1.BaseVestingAccount"></a>

### BaseVestingAccount
BaseVestingAccount 实现了 VestingAccount 接口。 它包含任何归属账户实施所需的所有必要字段。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_account` | [cosmos.auth.v1beta1.BaseAccount](#cosmos.auth.v1beta1.BaseAccount) |  |  |
| `original_vesting` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `delegated_free` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `delegated_vesting` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `end_time` | [int64](#int64) |  |  |






<a name="cosmos.vesting.v1beta1.ContinuousVestingAccount"></a>

### ContinuousVestingAccount
ContinuousVestingAccount 实现了 VestingAccount 接口。 它通过随时间线性解锁硬币来持续归属。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_vesting_account` | [BaseVestingAccount](#cosmos.vesting.v1beta1.BaseVestingAccount) |  |  |
| `start_time` | [int64](#int64) |  |  |






<a name="cosmos.vesting.v1beta1.DelayedVestingAccount"></a>

### DelayedVestingAccount
DelayedVestingAccount 实现了 VestingAccount 接口。 它在特定时间后授予所有代币，但不是在此之前。 换句话说，它会将它们锁定到指定的时间。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_vesting_account` | [BaseVestingAccount](#cosmos.vesting.v1beta1.BaseVestingAccount) |  |  |






<a name="cosmos.vesting.v1beta1.Period"></a>

### Period
期间定义了将授予的时间长度和硬币数量。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `length` | [int64](#int64) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |






<a name="cosmos.vesting.v1beta1.PeriodicVestingAccount"></a>

### PeriodicVestingAccount
PeriodicVestingAccount 实现了 VestingAccount 接口。 它通过在每个指定时期解锁硬币来定期归属。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_vesting_account` | [BaseVestingAccount](#cosmos.vesting.v1beta1.BaseVestingAccount) |  |  |
| `start_time` | [int64](#int64) |  |  |
| `vesting_periods` | [Period](#cosmos.vesting.v1beta1.Period) | repeated |  |






<a name="cosmos.vesting.v1beta1.PermanentLockedAccount"></a>

### PermanentLockedAccount
PermanentLockedAccount 实现了 VestingAccount 接口。 它永远不会释放硬币，无限期地锁定它们。 即使在锁定时，此帐户中的硬币仍可用于委托和治理投票。

Since: cosmos-sdk 0.43


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `base_vesting_account` | [BaseVestingAccount](#cosmos.vesting.v1beta1.BaseVestingAccount) |  |  |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



<a name="cosmos/vesting/v1beta1/tx.proto"></a>
<p align="right"><a href="#top">Top</a></p>

## cosmos/vesting/v1beta1/tx.proto



<a name="cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccount"></a>

### MsgCreatePeriodicVestingAccount
MsgCreateVestingAccount 定义了一条消息，用于创建归属账户。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `from_address` | [string](#string) |  |  |
| `to_address` | [string](#string) |  |  |
| `start_time` | [int64](#int64) |  |  |
| `vesting_periods` | [Period](#cosmos.vesting.v1beta1.Period) | repeated |  |






<a name="cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccountResponse"></a>

### MsgCreatePeriodicVestingAccountResponse
MsgCreateVestingAccountResponse 定义了 Msg/CreatePeriodicVestingAccount 响应类型。






<a name="cosmos.vesting.v1beta1.MsgCreateVestingAccount"></a>

### MsgCreateVestingAccount
MsgCreateVestingAccount 定义了一条消息，用于创建归属账户。


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| `from_address` | [string](#string) |  |  |
| `to_address` | [string](#string) |  |  |
| `amount` | [cosmos.base.v1beta1.Coin](#cosmos.base.v1beta1.Coin) | repeated |  |
| `end_time` | [int64](#int64) |  |  |
| `delayed` | [bool](#bool) |  |  |






<a name="cosmos.vesting.v1beta1.MsgCreateVestingAccountResponse"></a>

### MsgCreateVestingAccountResponse
MsgCreateVestingAccountResponse 定义了 Msg/CreateVestingAccount 响应类型。





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->


<a name="cosmos.vesting.v1beta1.Msg"></a>

### Msg
Msg 定义了银行消息服务。

| Method Name | Request Type | Response Type | Description | HTTP Verb | Endpoint |
| ----------- | ------------ | ------------- | ------------| ------- | -------- |
| `CreateVestingAccount` | [MsgCreateVestingAccount](#cosmos.vesting.v1beta1.MsgCreateVestingAccount) | [MsgCreateVestingAccountResponse](#cosmos.vesting.v1beta1.MsgCreateVestingAccountResponse) | CreateVestingAccount defines a method that enables creating a vesting account. | |
| `CreatePeriodicVestingAccount` | [MsgCreatePeriodicVestingAccount](#cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccount) | [MsgCreatePeriodicVestingAccountResponse](#cosmos.vesting.v1beta1.MsgCreatePeriodicVestingAccountResponse) | CreatePeriodicVestingAccount defines a method that enables creating a periodic vesting account. | |

 <!-- end services -->



## Scalar Value Types

| .proto Type | Notes | C++ | Java | Python | Go | C# | PHP | Ruby |
| ----------- | ----- | --- | ---- | ------ | -- | -- | --- | ---- |
| <a name="double"/> double |  | double | double | float | float64 | double | float | Float |
| <a name="float"/> float |  | float | float | float | float32 | float | float | Float |
| <a name="int32"/> int32 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="int64"/> int64 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="uint32"/> uint32 | Uses variable-length encoding. | uint32 | int | int/long | uint32 | uint | integer | Bignum or Fixnum (as required) |
| <a name="uint64"/> uint64 | Uses variable-length encoding. | uint64 | long | int/long | uint64 | ulong | integer/string | Bignum or Fixnum (as required) |
| <a name="sint32"/> sint32 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="sint64"/> sint64 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="fixed32"/> fixed32 | Always four bytes. More efficient than uint32 if values are often greater than 2^28. | uint32 | int | int | uint32 | uint | integer | Bignum or Fixnum (as required) |
| <a name="fixed64"/> fixed64 | Always eight bytes. More efficient than uint64 if values are often greater than 2^56. | uint64 | long | int/long | uint64 | ulong | integer/string | Bignum |
| <a name="sfixed32"/> sfixed32 | Always four bytes. | int32 | int | int | int32 | int | integer | Bignum or Fixnum (as required) |
| <a name="sfixed64"/> sfixed64 | Always eight bytes. | int64 | long | int/long | int64 | long | integer/string | Bignum |
| <a name="bool"/> bool |  | bool | boolean | boolean | bool | bool | boolean | TrueClass/FalseClass |
| <a name="string"/> string | A string must always contain UTF-8 encoded or 7-bit ASCII text. | string | String | str/unicode | string | string | string | String (UTF-8) |
| <a name="bytes"/> bytes | May contain any arbitrary sequence of bytes. | string | ByteString | str | []byte | ByteString | string | String (ASCII-8BIT) |


