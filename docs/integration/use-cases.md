# Use Cases

## Introduction

Using Cloak as a building block in your wallet or dapp infrastructure, you can implement various use cases.
This page will present the most common ones.
Welcome to contact the Scroll team for exploring new features and use cases.

!!! warning

    Cloak is a pragmatic solution to on-chain confidentiality.
    As such, it does not aim to offer absolute privacy guarantees.
    **It is important to inform your users about the risks and limitations.**


## Shielded Balances and Confidential Transfers

The simplest use case for Cloak is for a wallet to offer confidential (shielded) balances.
The users flow would be as follows:

1. **Fund confidential account**.
   The wallet guides the user to bridge funds to Scroll, and then to deposit to a unique address on Cloak.
   Most of the complexities of bridging and depositing can be hidden from the user.

2. **Keep balance in confidential account**.
   By default, the user's tokens are in their confidential account, and cannot be queried by others.

3. **Make transfers with other confidential accounts**.
   For example, a merchant requests a payment from a customer.
   They can exchange wallet addresses off-chain (e.g. via a QR code).
   After this, the token transfer takes place between two confidential accounts on Cloak.
   This transfer is not visible to any unauthorized 3rd-parties.

4. **Transfer to unsafe wallets or to other chains**.
   For transferring to other chains, the wallet guides the user to withdraw to Scroll, and to bridge to other chains.


## Confidential DeFi

Apart from supporting the most basic use cases of holding and transferring tokens on Cloak, it can also support more complex operations via composability with Scroll.

**Example: Swap**.
It is not recommended to deploy a DEX on Cloak because of liquidity constraints.
Instead, dapps can utilize the fast withdraw and fast deposit features to simulate a swap inside Cloak:

1. Fast-withdraw some funds to a new address on Scroll.
   It is recommended to use a new, single-use address to avoid linkability with the users previous L2 and L3 identities.
2. Make a swap on Scroll.
3. Deposit the received tokens back to Cloak.

The dapp can execute these three operations quickly, or even batch them into a single operation.
This way, for the user this all happens in a single step within a few seconds.
