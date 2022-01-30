# クライアント

## CLI

ユーザーは、CLIを使用して`gov`モジュールを照会および操作できます。

### Query

`query`コマンドを使用すると、ユーザーは`gov`状態を照会できます。

```bash
simd query gov --help
```

#### deposit

`deposit`コマンドを使用すると、ユーザーは特定の預金者からの特定の提案について預金を照会できます。 

```bash
simd query gov deposit [proposal-id] [depositer-addr] [flags]
```

例:

```bash
simd query gov deposit 1 cosmos1..
```

出力例:

```bash
amount:
- amount: "100"
  denom: stake
depositor: cosmos1..
proposal_id: "1"
```

#### deposits

`deposits`コマンドを使用すると、ユーザーは特定のプロポーザルのすべてのデポジットを照会できます。

```bash
simd query gov deposits [proposal-id] [flags]
```

例:

```bash
simd query gov deposits 1
```

出力例:

```bash
deposits:
- amount:
  - amount: "100"
    denom: stake
  depositor: cosmos1..
  proposal_id: "1"
pagination:
  next_key: null
  total: "0"
```

#### param

`param`コマンドを使用すると、ユーザーは` gov`モジュールの特定のパラメーターを照会できます。

```bash
simd query gov param [param-type] [flags]
```

例:

```bash
simd query gov param voting
```

出力例:

```bash
voting_period: "172800000000000"
```

#### params

`params`コマンドを使用すると、ユーザーは` gov`モジュールのすべてのパラメーターを照会できます。

```bash
simd query gov params [flags]
```

例:

```bash
simd query gov params
```

出力例:

```bash
deposit_params:
  max_deposit_period: "172800000000000"
  min_deposit:
  - amount: "10000000"
    denom: stake
tally_params:
  quorum: "0.334000000000000000"
  threshold: "0.500000000000000000"
  veto_threshold: "0.334000000000000000"
voting_params:
  voting_period: "172800000000000"
```

#### proposal

`proposal`コマンドを使用すると、ユーザーは特定の提案を照会できます。

```bash
simd query gov proposal [proposal-id] [flags]
```

例:

```bash
simd query gov proposal 1
```

出力例:

```bash
messages: [
  {
    '@type':/cosmos.bank.v1beta1.MsgSend
    from_address: "cosmos1..",
    to_address: "cosmos1..",
    amount: "100atom"
  }
],
deposit_end_time: "2021-09-17T23:36:18.254995423Z"
final_tally_result:
  abstain: "0"
  "no": "0"
  no_with_veto: "0"
  "yes": "0"
proposal_id: "1"
status: PROPOSAL_STATUS_DEPOSIT_PERIOD
submit_time: "2021-09-15T23:36:18.254995423Z"
total_deposit:
- amount: "100"
  denom: stake
voting_end_time: "0001-01-01T00:00:00Z"
voting_start_time: "0001-01-01T00:00:00Z"
```

#### proposals

`proposals`コマンドを使用すると、ユーザーはオプションのフィルターを使用してすべての提案を照会できます。

```bash
simd query gov proposals [flags]
```

例:

```bash
simd query gov proposals
```

出力例:

```bash
pagination:
  next_key: null
  total: "1"
proposals:
- content:
    '@type':/cosmos.gov.v1beta1.TextProposal
    description: testing, testing, 1, 2, 3
    title: Test Proposal
  deposit_end_time: "2021-09-17T23:36:18.254995423Z"
  final_tally_result:
    abstain: "0"
    "no": "0"
    no_with_veto: "0"
    "yes": "0"
  proposal_id: "1"
  status: PROPOSAL_STATUS_DEPOSIT_PERIOD
  submit_time: "2021-09-15T23:36:18.254995423Z"
  total_deposit:
  - amount: "100"
    denom: stake
  voting_end_time: "0001-01-01T00:00:00Z"
  voting_start_time: "0001-01-01T00:00:00Z"
```

#### proposer

`proposer`コマンドを使用すると、ユーザーは特定の提案について提案者に問い合わせることができます。

```bash
simd query gov proposer [proposal-id] [flags]
```

例:

```bash
simd query gov proposer 1
```

出力例:

```bash
proposal_id: "1"
proposer: cosmos1r0tllwu5c9dtgwg3wr28lpvf76hg85f5zmh9l2
```

#### tally

`tally`コマンドを使用すると、ユーザーは特定の提案投票の集計を照会できます。

```bash
simd query gov tally [proposal-id] [flags]
```

例:

```bash
simd query gov tally 1
```

出力例:

```bash
abstain: "0"
"no": "0"
no_with_veto: "0"
"yes": "1"
```

#### vote

`vote`コマンドを使用すると、ユーザーは特定の提案に対する投票を照会できます。

```bash
simd query gov vote [proposal-id] [voter-addr] [flags]
```

例:

```bash
simd query gov vote 1 cosmos1..
```

出力例:

```bash
option: VOTE_OPTION_YES
options:
- option: VOTE_OPTION_YES
  weight: "1.000000000000000000"
proposal_id: "1"
voter: cosmos1..
```

#### votes

`votes`コマンドを使用すると、ユーザーは特定の提案に対するすべての投票を照会できます。

```bash
simd query gov votes [proposal-id] [flags]
```

例:

```bash
simd query gov votes 1
```

出力例:

```bash
pagination:
  next_key: null
  total: "0"
votes:
- option: VOTE_OPTION_YES
  options:
  - option: VOTE_OPTION_YES
    weight: "1.000000000000000000"
  proposal_id: "1"
  voter: cosmos1r0tllwu5c9dtgwg3wr28lpvf76hg85f5zmh9l2
```

### Transactions

`tx`コマンドを使用すると、ユーザーは` gov`モジュールを操作できます。

```bash
simd tx gov --help
```

#### deposit

`deposit`コマンドを使用すると、ユーザーは特定のプロポーザルのトークンをデポジットできます。

```bash
simd tx gov deposit [proposal-id] [deposit] [flags]
```

例:

```bash
simd tx gov deposit 1 10000000stake --from cosmos1..
```

#### submit-proposal

`submit-proposal`コマンドを使用すると、ユーザーはガバナンス提案を送信し、オプションで初期デポジットを含めることができます。

```bash
simd tx gov submit-proposal [command] [flags]
```

例:

```bash
simd tx gov submit-proposal --title="Test Proposal" --description="testing, testing, 1, 2, 3" --type="Text" --deposit="10000000stake" --from cosmos1..
```

例 (`cancel-software-upgrade`):

```bash
simd tx gov submit-proposal cancel-software-upgrade --title="Test Proposal" --description="testing, testing, 1, 2, 3" --deposit="10000000stake" --from cosmos1..
```

例 (`community-pool-spend`):

```bash
simd tx gov submit-proposal community-pool-spend proposal.json --from cosmos1..
```

```json
{
  "title": "Test Proposal",
  "description": "testing, testing, 1, 2, 3",
  "recipient": "cosmos1..",
  "amount": "10000000stake",
  "deposit": "10000000stake"
}
```

例 (`param-change`):

```bash
simd tx gov submit-proposal param-change proposal.json --from cosmos1..
```

```json
{
  "title": "Test Proposal",
  "description": "testing, testing, 1, 2, 3",
  "changes": [
    {
      "subspace": "staking",
      "key": "MaxValidators",
      "value": 100
    }
  ],
  "deposit": "10000000stake"
}
```

例 (`software-upgrade`):

```bash
simd tx gov submit-proposal software-upgrade v2 --title="Test Proposal" --description="testing, testing, 1, 2, 3" --upgrade-height 1000000 --from cosmos1..
```

#### vote

`vote`コマンドを使用すると、ユーザーは特定のガバナンス提案に投票を送信できます。

```bash
simd tx gov vote [command] [flags]
```

例:

```bash
simd tx gov vote 1 yes --from cosmos1..
```

#### weighted-vote

`weighted-vote`コマンドを使用すると、ユーザーは特定のガバナンス提案に対して加重投票を送信できます。

```bash
simd tx gov weighted-vote [proposal-id] [weighted-options]
```

例:

```bash
simd tx gov weighted-vote 1 yes=0.5,no=0.5 --from cosmos1
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `gov`モジュールにクエリを実行できます。

### Proposal

`Proposal`エンドポイントを使用すると、ユーザーは特定のプロポーザルを照会できます。

```bash
cosmos.gov.v1beta1.Query/Proposal
```

例:

```bash
grpcurl -plaintext \
    -d '{"proposal_id":"1"}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Proposal
```

出力例:

```bash
{
  "proposal": {
    "proposalId": "1",
    "content": {"@type":"/cosmos.gov.v1beta1.TextProposal","description":"testing, testing, 1, 2, 3","title":"Test Proposal"},
    "status": "PROPOSAL_STATUS_VOTING_PERIOD",
    "finalTallyResult": {
      "yes": "0",
      "abstain": "0",
      "no": "0",
      "noWithVeto": "0"
    },
    "submitTime": "2021-09-16T19:40:08.712440474Z",
    "depositEndTime": "2021-09-18T19:40:08.712440474Z",
    "totalDeposit": [
      {
        "denom": "stake",
        "amount": "10000000"
      }
    ],
    "votingStartTime": "2021-09-16T19:40:08.712440474Z",
    "votingEndTime": "2021-09-18T19:40:08.712440474Z"
  }
}
```

### Proposals

`Proposals`エンドポイントを使用すると、ユーザーはオプションのフィルターを使用してすべての提案を照会できます。

```bash
cosmos.gov.v1beta1.Query/Proposals
```

例:

```bash
grpcurl -plaintext \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Proposals
```

出力例:

```bash
{
  "proposals": [
    {
      "proposalId": "1",
      "content": {"@type":"/cosmos.gov.v1beta1.TextProposal","description":"testing, testing, 1, 2, 3","title":"Test Proposal"},
      "status": "PROPOSAL_STATUS_VOTING_PERIOD",
      "finalTallyResult": {
        "yes": "0",
        "abstain": "0",
        "no": "0",
        "noWithVeto": "0"
      },
      "submitTime": "2021-09-16T19:40:08.712440474Z",
      "depositEndTime": "2021-09-18T19:40:08.712440474Z",
      "totalDeposit": [
        {
          "denom": "stake",
          "amount": "10000000"
        }
      ],
      "votingStartTime": "2021-09-16T19:40:08.712440474Z",
      "votingEndTime": "2021-09-18T19:40:08.712440474Z"
    },
    {
      "proposalId": "2",
      "content": {"@type":"/cosmos.upgrade.v1beta1.CancelSoftwareUpgradeProposal","description":"Test Proposal","title":"testing, testing, 1, 2, 3"},
      "status": "PROPOSAL_STATUS_DEPOSIT_PERIOD",
      "finalTallyResult": {
        "yes": "0",
        "abstain": "0",
        "no": "0",
        "noWithVeto": "0"
      },
      "submitTime": "2021-09-17T18:26:57.866854713Z",
      "depositEndTime": "2021-09-19T18:26:57.866854713Z",
      "votingStartTime": "0001-01-01T00:00:00Z",
      "votingEndTime": "0001-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "total": "2"
  }
}
```

### Vote

`Vote`エンドポイントを使用すると、ユーザーは特定の提案に対する投票を照会できます。

```bash
cosmos.gov.v1beta1.Query/Vote
```

例:

```bash
grpcurl -plaintext \
    -d '{"proposal_id":"1","voter":"cosmos1.."}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Vote
```

出力例:

```bash
{
  "vote": {
    "proposalId": "1",
    "voter": "cosmos1..",
    "option": "VOTE_OPTION_YES",
    "options": [
      {
        "option": "VOTE_OPTION_YES",
        "weight": "1000000000000000000"
      }
    ]
  }
}
```

### Votes

`Votes`エンドポイントを使用すると、ユーザーは特定の提案に対するすべての投票を照会できます。

```bash
cosmos.gov.v1beta1.Query/Votes
```

例:

```bash
grpcurl -plaintext \
    -d '{"proposal_id":"1"}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Votes
```

出力例:

```bash
{
  "votes": [
    {
      "proposalId": "1",
      "voter": "cosmos1..",
      "option": "VOTE_OPTION_YES",
      "options": [
        {
          "option": "VOTE_OPTION_YES",
          "weight": "1000000000000000000"
        }
      ]
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### Params

`Params`エンドポイントを使用すると、ユーザーは` gov`モジュールのすべてのパラメーターを照会できます。

<!-- TODO: #10197 Querying governance params outputs nil values -->

```bash
cosmos.gov.v1beta1.Query/Params
```

例:

```bash
grpcurl -plaintext \
    -d '{"params_type":"voting"}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Params
```

出力例:

```bash
{
  "votingParams": {
    "votingPeriod": "172800s"
  },
  "depositParams": {
    "maxDepositPeriod": "0s"
  },
  "tallyParams": {
    "quorum": "MA==",
    "threshold": "MA==",
    "vetoThreshold": "MA=="
  }
}
```

### Deposit

`Deposit`エンドポイントを使用すると、ユーザーは特定の預金者からの特定の提案について預金を照会できます。

```bash
cosmos.gov.v1beta1.Query/Deposit
```

例:

```bash
grpcurl -plaintext \
    '{"proposal_id":"1","depositor":"cosmos1.."}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Deposit
```

出力例:

```bash
{
  "deposit": {
    "proposalId": "1",
    "depositor": "cosmos1..",
    "amount": [
      {
        "denom": "stake",
        "amount": "10000000"
      }
    ]
  }
}
```

### deposits

`Deposits`エンドポイントを使用すると、ユーザーは特定のプロポーザルのすべてのデポジットを照会できます。

```bash
cosmos.gov.v1beta1.Query/Deposits
```

例:

```bash
grpcurl -plaintext \
    -d '{"proposal_id":"1"}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/Deposits
```

出力例:

```bash
{
  "deposits": [
    {
      "proposalId": "1",
      "depositor": "cosmos1..",
      "amount": [
        {
          "denom": "stake",
          "amount": "10000000"
        }
      ]
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### TallyResult

`TallyResult`エンドポイントを使用すると、ユーザーは特定のプロポーザルの集計を照会できます。

```bash
cosmos.gov.v1beta1.Query/TallyResult
```

例:

```bash
grpcurl -plaintext \
    -d '{"proposal_id":"1"}' \
    localhost:9090 \
    cosmos.gov.v1beta1.Query/TallyResult
```

出力例:

```bash
{
  "tally": {
    "yes": "1000000",
    "abstain": "0",
    "no": "0",
    "noWithVeto": "0"
  }
}
```

## REST

ユーザーは、RESTエンドポイントを使用して `gov`モジュールにクエリを実行できます。

### proposal

`proposals`エンドポイントを使用すると、ユーザーは特定のプロポーザルを照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1
```

出力例:

```bash
{
  "proposal": {
    "proposal_id": "1",
    "content": {
      "@type": "/cosmos.gov.v1beta1.TextProposal",
      "title": "Test Proposal",
      "description": "testing, testing, 1, 2, 3"
    },
    "status": "PROPOSAL_STATUS_VOTING_PERIOD",
    "final_tally_result": {
      "yes": "0",
      "abstain": "0",
      "no": "0",
      "no_with_veto": "0"
    },
    "submit_time": "2021-09-16T19:40:08.712440474Z",
    "deposit_end_time": "2021-09-18T19:40:08.712440474Z",
    "total_deposit": [
      {
        "denom": "stake",
        "amount": "10000000"
      }
    ],
    "voting_start_time": "2021-09-16T19:40:08.712440474Z",
    "voting_end_time": "2021-09-18T19:40:08.712440474Z"
  }
}
```

### proposals

`proposals`エンドポイントを使用すると、ユーザーはオプションのフィルターを使用してすべての提案を照会することもできます。

```bash
/cosmos/gov/v1beta1/proposals
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals
```

出力例:

```bash
{
  "proposals": [
    {
      "proposal_id": "1",
      "content": {
        "@type": "/cosmos.gov.v1beta1.TextProposal",
        "title": "Test Proposal",
        "description": "testing, testing, 1, 2, 3"
      },
      "status": "PROPOSAL_STATUS_VOTING_PERIOD",
      "final_tally_result": {
        "yes": "0",
        "abstain": "0",
        "no": "0",
        "no_with_veto": "0"
      },
      "submit_time": "2021-09-16T19:40:08.712440474Z",
      "deposit_end_time": "2021-09-18T19:40:08.712440474Z",
      "total_deposit": [
        {
          "denom": "stake",
          "amount": "10000000"
        }
      ],
      "voting_start_time": "2021-09-16T19:40:08.712440474Z",
      "voting_end_time": "2021-09-18T19:40:08.712440474Z"
    },
    {
      "proposal_id": "2",
      "content": {
        "@type": "/cosmos.upgrade.v1beta1.CancelSoftwareUpgradeProposal",
        "title": "Test Proposal",
        "description": "testing, testing, 1, 2, 3"
      },
      "status": "PROPOSAL_STATUS_DEPOSIT_PERIOD",
      "final_tally_result": {
        "yes": "0",
        "abstain": "0",
        "no": "0",
        "no_with_veto": "0"
      },
      "submit_time": "2021-09-17T18:26:57.866854713Z",
      "deposit_end_time": "2021-09-19T18:26:57.866854713Z",
      "total_deposit": [
      ],
      "voting_start_time": "0001-01-01T00:00:00Z",
      "voting_end_time": "0001-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "next_key": null,
    "total": "2"
  }
}
```

### voter vote

`votes`エンドポイントを使用すると、ユーザーは特定の提案に対する投票を照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}/votes/{voter}
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1/votes/cosmos1..
```

出力例:

```bash
{
  "vote": {
    "proposal_id": "1",
    "voter": "cosmos1..",
    "option": "VOTE_OPTION_YES",
    "options": [
      {
        "option": "VOTE_OPTION_YES",
        "weight": "1.000000000000000000"
      }
    ]
  }
}
```

### votes

`votes`エンドポイントを使用すると、ユーザーは特定のプロポーザルのすべての投票を照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}/votes
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1/votes
```

出力例:

```bash
{
  "votes": [
    {
      "proposal_id": "1",
      "voter": "cosmos1..",
      "option": "VOTE_OPTION_YES",
      "options": [
        {
          "option": "VOTE_OPTION_YES",
          "weight": "1.000000000000000000"
        }
      ]
    }
  ],
  "pagination": {
    "next_key": null,
    "total": "1"
  }
}
```

### params

`params`エンドポイントを使用すると、ユーザーは` gov`モジュールのすべてのパラメーターを照会できます。

<!-- TODO: #10197 Querying governance params outputs nil values -->

```bash
/cosmos/gov/v1beta1/params/{params_type}
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/params/voting
```

出力例:

```bash
{
  "voting_params": {
    "voting_period": "172800s"
  },
  "deposit_params": {
    "min_deposit": [
    ],
    "max_deposit_period": "0s"
  },
  "tally_params": {
    "quorum": "0.000000000000000000",
    "threshold": "0.000000000000000000",
    "veto_threshold": "0.000000000000000000"
  }
}
```

### deposits

`deposits`エンドポイントを使用すると、ユーザーは特定の預金者からの特定の提案について預金を照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}/deposits/{depositor}
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1/deposits/cosmos1..
```

出力例:

```bash
{
  "deposit": {
    "proposal_id": "1",
    "depositor": "cosmos1..",
    "amount": [
      {
        "denom": "stake",
        "amount": "10000000"
      }
    ]
  }
}
```

### proposal deposits

`deposits`エンドポイントを使用すると、ユーザーは特定のプロポーザルのすべてのデポジットを照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}/deposits
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1/deposits
```

出力例:

```bash
{
  "deposits": [
    {
      "proposal_id": "1",
      "depositor": "cosmos1..",
      "amount": [
        {
          "denom": "stake",
          "amount": "10000000"
        }
      ]
    }
  ],
  "pagination": {
    "next_key": null,
    "total": "1"
  }
}
```

### tally

`tally`エンドポイントを使用すると、ユーザーは特定のプロポーザルの集計を照会できます。

```bash
/cosmos/gov/v1beta1/proposals/{proposal_id}/tally
```

例:

```bash
curl localhost:1317/cosmos/gov/v1beta1/proposals/1/tally
```

出力例:

```bash
{
  "tally": {
    "yes": "1000000",
    "abstain": "0",
    "no": "0",
    "no_with_veto": "0"
  }
}
```
