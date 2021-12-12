<!--
order: 0
-->

# List of Modules

Here are some production-grade modules that can be used in Cosmos SDK applications, along with their respective documentation:

- [Auth](auth/) - Authentication of accounts and transactions for Cosmos SDK applications.
- [Authz](authz/) - Authorization for accounts to perform actions on behalf of other accounts.
- [Bank](bank/) - Token transfer functionalities.
- [Capability](capability/) - Object capability implementation.
- [Crisis](crisis/) - Halting the blockchain under certain circumstances (e.g. if an invariant is broken).
- [Distribution](distribution/) - Fee distribution, and staking token provision distribution.
- [Epoching](epoching/) - Allows modules to queue messages for execution at a certain block height.
- [Evidence](evidence/) - Evidence handling for double signing, misbehaviour, etc.
- [Feegrant](feegrant/) - Grant fee allowances for executing transactions.
- [Governance](gov/) - On-chain proposals and voting.
- [Mint](mint/) - Creation of new units of staking token.
- [Params](params/) - Globally available parameter store.
- [Slashing](slashing/) - Validator punishment mechanisms.
- [Staking](staking/) - Proof-of-Stake layer for public blockchains.
- [Upgrade](upgrade/) - Software upgrades handling and coordination.

To learn more about the process of building modules, visit the [building modules reference documentation](/building-modules/intro.html).

## IBC

The IBC module for the SDK has moved to its [own repository](https://github.com/cosmos/ibc-go).
