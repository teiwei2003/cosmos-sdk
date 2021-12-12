<!--
order: 2
-->

# REST 端点迁移

迁移到 gRPC-Gateway REST 端点。旧版 REST 端点在 v0.40 中被标记为已弃用，并将在 v0.45 中删除。 {概要}

::: 警告
由于安全漏洞，在 v0.44 中提前删除了两个旧版 REST 端点(`POST /txs` 和 `POST /txs/encode`)。
:::

## 传统 REST 端点

Cosmos SDK 版本 v0.39 和更早版本使用名为“gorilla/mux”的包注册了 REST 端点。这些 REST 端点在 v0.40 中被标记为已弃用，此后被称为旧版 REST 端点。传统 REST 端点将在 v0.45 中正式删除。

## gRPC 网关 REST 端点

随着 v0.40 中的 Protocol Buffers 迁移，Cosmos SDK 已被设置为利用大量 gRPC 工具和解决方案。 v0.40 引入了使用 [grpc-gateway](https://grpc-ecosystem.github.io/grpc) 从 [gRPC `Query` 服务](../building-modules/query-services.md) 生成的新 REST 端点-网关/)。这些新的 REST 端点称为 gRPC-Gateway REST 端点。

## 迁移到新的 REST 端点 

| Legacy REST Endpoint                                                            | Description                                                         | New gRPC-gateway REST Endpoint                                                                        |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `GET /txs/{hash}`                                                               | Query tx by hash                                                    | `GET /cosmos/tx/v1beta1/txs/{hash}`                                                                   |
| `GET /txs`                                                                      | Query tx by events                                                  | `GET /cosmos/tx/v1beta1/txs`                                                                          |
| `POST /txs`                                                                     | Broadcast tx                                                        | `POST /cosmos/tx/v1beta1/txs`                                                                         |
| `POST /txs/encode`                                                              | Encodes an Amino JSON tx to an Amino binary tx                      | N/A, use Protobuf directly                                                                            |
| `POST /txs/decode`                                                              | Decodes an Amino binary tx into an Amino JSON tx                    | N/A, use Protobuf directly                                                                            |
| `POST /bank/*`                                                                  | Create unsigned `Msg`s for bank tx                                  | N/A, use Protobuf directly                                                                            |
| `GET /bank/balances/{address}`                                                  | Get the balance of an address                                       | `GET /cosmos/bank/v1beta1/balances/{address}`                                                 |
| `GET /bank/total`                                                               | Get the total supply of all coins                                   | `GET /cosmos/bank/v1beta1/supply`                                                                     |
| `GET /bank/total/{denom}`                                                       | Get the total supply of one coin                                    | `GET /cosmos/bank/v1beta1/supply/{denom}`                                                             |
| `POST /distribution/delegators/{delegatorAddr}/rewards`                         | Withdraw all delegator rewards                                      | N/A, use Protobuf directly                                                                            |
| `POST /distribution/*`                                                          | Create unsigned `Msg`s for distribution                             | N/A, use Protobuf directly                                                                            |
| `GET /distribution/delegators/{delegatorAddr}/rewards`                          | Get the total rewards balance from all delegations                  | `GET /cosmos/distribution/v1beta1/v1beta1/delegators/{delegator_address}/rewards`                     |
| `GET /distribution/delegators/{delegatorAddr}/rewards/{validatorAddr}`          | Query a delegation reward                                           | `GET /cosmos/distribution/v1beta1/delegators/{delegatorAddr}/rewards/{validatorAddr}`                 |
| `GET /distribution/delegators/{delegatorAddr}/withdraw_address`                 | Get the rewards withdrawal address                                  | `GET /cosmos/distribution/v1beta1/delegators/{delegatorAddr}/withdraw_address`                        |
| `GET /distribution/validators/{validatorAddr}`                                  | Validator distribution information                                  | N/A                                         |
| `GET /distribution/validators/{validatorAddr}/rewards`                          | Commission and outstanding rewards of a single a validator      |  `GET /cosmos/distribution/v1beta1/validators/{validatorAddr}/commission` <br> `GET /cosmos/distribution/v1beta1/validators/{validatorAddr}/outstanding_rewards`                               |
| `GET /distribution/validators/{validatorAddr}/outstanding_rewards`              | Outstanding rewards of a single validator                           | `GET /cosmos/distribution/v1beta1/validators/{validatorAddr}/outstanding_rewards`                     |
| `GET /distribution/parameters`                                                  | Get the current distribution parameter values                       | `GET /cosmos/distribution/v1beta1/params`                                                             |
| `GET /distribution/community_pool`                                              | Get the amount held in the community pool                           | `GET /cosmos/distribution/v1beta1/community_pool`                                                     |
| `GET /evidence/{evidence-hash}`                                                 | Get evidence by hash                                                | `GET /cosmos/evidence/v1beta1/evidence/{evidence_hash}`                                               |
| `GET /evidence`                                                                 | Get all evidence                                                    | `GET /cosmos/evidence/v1beta1/evidence`                                                               |
| `POST /gov/*`                                                                   | Create unsigned `Msg`s for gov                                      | N/A, use Protobuf directly                                                                            |
| `GET /gov/parameters/{type}`                                                    | Get government parameters                                           | `GET /cosmos/gov/v1beta1/params/{type}`                                                               |
| `GET /gov/proposals`                                                            | Get all proposals                                                   | `GET /cosmos/gov/v1beta1/proposals`                                                                   |
| `GET /gov/proposals/{proposal-id}`                                              | Get proposal by id                                                  | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}`                                                     |
| `GET /gov/proposals/{proposal-id}/proposer`                                     | Get proposer of a proposal                                          | N/A, use Query tx by events endpoint               |
| `GET /gov/proposals/{proposal-id}/deposits`                                     | Get deposits of a proposal                                          | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}/deposits`                                            |
| `GET /gov/proposals/{proposal-id}/deposits/{depositor}`                         | Get depositor a of deposit                                          | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}/deposits/{depositor}`                                |
| `GET /gov/proposals/{proposal-id}/tally`                                        | Get tally of a proposal                                             | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}/tally`                                               |
| `GET /gov/proposals/{proposal-id}/votes`                                        | Get votes of a proposal                                             | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}/votes`                                               |
| `GET /gov/proposals/{proposal-id}/votes/{vote}`                                 | Get a particular vote                                               | `GET /cosmos/gov/v1beta1/proposals/{proposal-id}/votes/{vote}`                                        |
| `GET /minting/parameters`                                                       | Get parameters for minting                                          | `GET /cosmos/minting/v1beta1/params`                                                                  |
| `GET /minting/inflation`                                                        | Get minting inflation                                               | `GET /cosmos/minting/v1beta1/inflation`                                                               |
| `GET /minting/annual-provisions`                                                | Get minting annual provisions                                       | `GET /cosmos/minting/v1beta1/annual_provisions`                                                       |
| `POST /slashing/*`                                                              | Create unsigned `Msg`s for slashing                                 | N/A, use Protobuf directly                                                                            |
| `GET /slashing/validators/{validatorPubKey}/signing_info`                       | Get validator signing info                                          | `GET /cosmos/slashing/v1beta1/signing_infos/{cons_address}` (Use consensus address instead of pubkey) |
| `GET /slashing/signing_infos`                                                   | Get all signing infos                                               | `GET /cosmos/slashing/v1beta1/signing_infos`                                                          |
| `GET /slashing/parameters`                                                      | Get slashing parameters                                             | `GET /cosmos/slashing/v1beta1/params`                                                                 |
| `POST /staking/*`                                                               | Create unsigned `Msg`s for staking                                  | N/A, use Protobuf directly                                                                            |
| `GET /staking/delegators/{delegatorAddr}/delegations`                           | Get all delegations from a delegator                                | `GET /cosmos/staking/v1beta1/delegations/{delegatorAddr}`                                  |
| `GET /staking/delegators/{delegatorAddr}/unbonding_delegations`                 | Get all unbonding delegations from a delegator                      | `GET /cosmos/staking/v1beta1/delegators/{delegatorAddr}/unbonding_delegations`                        |
| `GET /staking/delegators/{delegatorAddr}/txs`                                   | Get all staking txs (i.e msgs) from a delegator                     | Removed                                                                                               |
| `GET /staking/delegators/{delegatorAddr}/validators`                            | Query all validators that a delegator is bonded to                  | `GET /cosmos/staking/v1beta1/delegators/{delegatorAddr}/validators`                                   |
| `GET /staking/delegators/{delegatorAddr}/validators/{validatorAddr}`            | Query a validator that a delegator is bonded to                     | `GET /cosmos/staking/v1beta1/delegators/{delegatorAddr}/validators/{validatorAddr}`                   |
| `GET /staking/delegators/{delegatorAddr}/delegations/{validatorAddr}`           | Query a delegation between a delegator and a validator              | `GET /cosmos/staking/v1beta1/validators/{validatorAddr}/delegations/{delegatorAddr}`                  |
| `GET /staking/delegators/{delegatorAddr}/unbonding_delegations/{validatorAddr}` | Query all unbonding delegations between a delegator and a validator | `GET /cosmos/staking/v1beta1/delegators/{delegatorAddr}/unbonding_delegations/{validatorAddr}`        |
| `GET /staking/redelegations`                                                    | Query redelegations                                                 | `GET /cosmos/staking/v1beta1/v1beta1/delegators/{delegator_addr}/redelegations`                       |
| `GET /staking/validators`                                                       | Get all validators                                                  | `GET /cosmos/staking/v1beta1/validators`                                                              |
| `GET /staking/validators/{validatorAddr}`                                       | Get a single validator info                                         | `GET /cosmos/staking/v1beta1/validators/{validatorAddr}`                                              |
| `GET /staking/validators/{validatorAddr}/delegations`                           | Get all delegations to a validator                                  | `GET /cosmos/staking/v1beta1/validators/{validatorAddr}/delegations`                                  |
| `GET /staking/validators/{validatorAddr}/unbonding_delegations`                 | Get all unbonding delegations from a validator                      | `GET /cosmos/staking/v1beta1/validators/{validatorAddr}/unbonding_delegations`                        |
| `GET /staking/historical_info/{height}`                                         | Get HistoricalInfo at a given height                                | `GET /cosmos/staking/v1beta1/historical_info/{height}`                                                |
| `GET /staking/pool`                                                             | Get the current state of the staking pool                           | `GET /cosmos/staking/v1beta1/pool`                                                                    |
| `GET /staking/parameters`                                                       | Get the current staking parameter values                            | `GET /cosmos/staking/v1beta1/params`                                                                  |
| `POST /upgrade/*`                                                               | Create unsigned `Msg`s for upgrade                                  | N/A, use Protobuf directly                                                                            |
| `GET /upgrade/current`                                                          | Get the current plan                                                | `GET /cosmos/upgrade/v1beta1/current_plan`                                                            |
| `GET /upgrade/applied_plan/{name}`                                              | Get a previously applied plan                                       | `GET /cosmos/upgrade/v1beta1/applied/{name}`                                                          |

## 迁移到 gRPC

Cosmos SDK 并没有像上面描述的那样访问 REST 端点，而是公开了一个 gRPC 服务器。 任何客户端都可以使用 gRPC 而不是 REST 与节点进行交互。 可以在 [此处](../core/grpc_rest.md) 中找到与节点通信的不同方式的概述，可以在 [此处](../run-node /txs.md#programmatically-with-go)。