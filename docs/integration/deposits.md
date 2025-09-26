# Depositing Funds to Cloak

## Introduction

Most Cloak instances are deployed as [L3 chains](glossary#l3-chain) on top of Scroll.
That means that there is a canonical smart contract token bridge between Scroll and Cloak.

The general deposit process for Cloak chain operators is:

1. Bridge assets to Scroll via the [canonical bridge](https://portal.scroll.io/bridge) (from Ethereum) or via [3rd-party bridges](https://scroll.io/ecosystem).
2. Deposit to Cloak via the Cloak bridge.

The remainder of this document will focus on Step 2.

!!! info "Integration options"

    For bridging your users' assets to Cloak, you can implement the above process in your frontend.
    Alternatively, you can also implement your own liquidity bridge by bridging a certain amount of tokens to Cloak, and distributing them to users based on your application logic.


## Overview of the Cloak Bridge

The Cloak bridge implements a general message-passing interface between L2 (Scroll) and L3 (Cloak).
Most of the bridge code is inherited from the Scroll bridge.

- The user calls one of the token gateway contracts to initiate the deposit: `HostERC20Gateway.depositERC20` or `HostETHGateway.depositETH`.
- The token gateway relays the message to `HostMessenger`, which enqueues a message on `HostMessageQueue`.
- The L3 sequencer node picks up the message and relays it to Cloak via `ValidiumERC20Gateway`.

Most of the above contracts are implementation details; users generally mainly interact with the gateway contracts.


### A Note on Stealth Deposits

Unlike the Scroll bridge, the Cloak bridge uses *stealth deposits*, meaning that the deposit target address is encrypted.
This is necessary, so that the L3 confidential identity is not exposed publicly on L2.
We also want to avoid easily linking L2 and L3 accounts.


## Prerequisites

We recommend using the [`@scroll-tech/cloak-js`](https://github.com/scroll-tech/cloak-js/tree/main) package.
This package currently supports `viem`, with plans to supports `ethers-js`.

To use this package in your JavaScript or TypeScript project, simply install and then import it:

```js
import { abis, cloak } from '@scroll-tech/cloak-js';

const c = cloak('local-devnet');
console.log(c.contracts());
```


## Depositing ETH to Cloak

!!! info "See the full example at [deposit-eth.ts](https://github.com/scroll-tech/cloak-js/tree/main/examples/viem/deposit-eth.ts) example in the `@scroll-tech/cloak-js` package."

The deposit process is as follows.

1. Fetch current encryption key.

    ```js linenums="1"
    const [keyId, encryptionKey] = await l2Client.readContract({
      address: c.contracts().HostValidium,
      abi: abis.HostValidium,
      functionName: 'getLatestEncryptionKey',
    });
    ```

2. Encrypt target address using the encryption key.

    ```js linenums="1"
    const recipient = c.encryptAddress(l3Address/* (1)! */, encryptionKey);
    ```

    1. It is recommended to generate a unique L3 address for the user.
       One that is not used on another networks.

3. Send deposit to L2 bridge.

    ```js linenums="1"
    await l2Wallet.writeContract({
      chain: null,
      address: c.contracts().HostWethGateway,
      value: amount,
      abi: abis.HostWethGateway,
      functionName: 'deposit',
      args: [recipient, amount, keyId],
    });
    ```

4. Wait for deposit to be confirmed on L3.

    ```js linenums="1"
    const l3Receipt = await l3Client.waitForTransactionReceipt({
      hash: deposit.possibleL3TxHashes.decrypted!
    });
    ```


## ERC20 Token Mapping

The Scroll-Cloak bridge can permissionlessly bridge any valid ERC20 token.
The bridge will deterministically compute the corresponding L3 token address, which will not change after the first deposit.
L2 ETH is bridged into L3 Wrapped ETH.

!!! info "ETH deposits"

    By default, L2 ETH is deposited into L3 Wrapped ETH (ERC20 token), not into the L3 gas token.
    L3 gas is free, so users do not need to hold any gas token.

The L2-L3 token mapping can be queried via the `HostERC20Gateway` contract on L2:

```js linenums="1"
const l3TokenAddress = await l2Client.readContract({
  address: c.contracts().HostERC20Gateway,
  abi: abis.HostERC20Gateway,
  functionName: 'getL2ERC20Address',
  args: [c.contracts().HostWeth],
});
```
