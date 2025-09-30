# Source Code

## Main Repositories

Cloak is built on top of the following codebases.
Some of these are existing Scroll components with some modifications, while others were newly developed for Cloak.

| Codebase                 | Repository                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Contracts                | [https://github.com/scroll-tech/scroll-contracts](https://github.com/scroll-tech/scroll-contracts)           |
| Sequencer                | [https://github.com/scroll-tech/go-ethereum](https://github.com/scroll-tech/go-ethereum)                     |
| Rollup-Node              | [https://github.com/scroll-tech/scroll](https://github.com/scroll-tech/scroll)                               |
| Circuits                 | [https://github.com/scroll-tech/zkvm-prover](https://github.com/scroll-tech/zkvm-prover)                     |
| Authenticated RPC proxy  | [https://github.com/scroll-tech/rpc-auth-proxy](https://github.com/scroll-tech/rpc-auth-proxy)               |
| Fast withdraw service    | [https://github.com/scroll-tech/cloak-fast-withdraw](https://github.com/scroll-tech/cloak-fast-withdraw)     |
| Withdraw proofs service  | [https://github.com/scroll-tech/cloak-withdraw-proofs](https://github.com/scroll-tech/cloak-withdraw-proofs) |
| TypeScript SDK           | [https://github.com/scroll-tech/cloak-js](https://github.com/scroll-tech/cloak-js)                           |
| Devnet                   | Coming soon                                                                                                  |


## Smart Contracts

Main contracts deployed on Scroll (L2):

- [ScrollChainValidium](https://github.com/scroll-tech/scroll-contracts/blob/main/src/validium/ScrollChainValidium.sol)
- [L1MessageQueue](https://github.com/scroll-tech/scroll-contracts/blob/main/src/L1/rollup/L1MessageQueueV2.sol)
- [ScrollMessengerValidium](https://github.com/scroll-tech/scroll-contracts/blob/main/src/validium/L1ScrollMessengerValidium.sol)
- [L1ERC20GatewayValidium](https://github.com/scroll-tech/scroll-contracts/blob/main/src/validium/L1ERC20GatewayValidium.sol)
- [L1WETHGatewayValidium](https://github.com/scroll-tech/scroll-contracts/blob/main/src/validium/L1WETHGatewayValidium.sol)
- [FastWithdrawVault](https://github.com/scroll-tech/scroll-contracts/blob/main/src/validium/FastWithdrawVault.sol)

Main contracts deployed on Cloak (L3):

- [L2ScrollMessenger](https://github.com/scroll-tech/scroll-contracts/blob/main/src/L2/L2ScrollMessenger.sol)
- [L2StandardERC20Gateway](https://github.com/scroll-tech/scroll-contracts/blob/main/src/L2/gateways/L2StandardERC20Gateway.sol)

See the [Address Book](/integration/address-book) for the contract addresses.