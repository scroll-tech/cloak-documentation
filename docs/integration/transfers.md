# Making Transfers inside Cloak

## Introduction

Cloak is an exact copy of Scroll's EVM.
As it is a full fledged EVM execution environment, one can do ERC20 token transfers or interact with smart contracts.

The simplest [use case](/integration/use-cases) is to let users hold tokens in a wallet on Cloak (similar to a shielded account) and to make transfers to other users.
In this document we will present this use case.


## Making Token Transfers

!!! info "See the full example at [transfer.ts](https://github.com/scroll-tech/cloak-js/tree/main/examples/viem/transfer.ts) example in the `@scroll-tech/cloak-js` package."

To make ERC20 transfers (for L3 WETH or any other deposited token), simply call the token contract `transfer` function, just like you would on other EVM chains.

=== "viem"

    ```js linenums="1"
    import { abis, cloak } from '@scroll-tech/cloak-js';

    const c = cloak('local-devnet');

    // configure client with access token...

    const hash = await l3Wallet.writeContract({
      address: c.contracts().ValidiumWeth,
      abi: abis.ERC20,
      functionName: 'transfer',
      args: [recipient.address, amount],
    });

    const receipt = await l3Client.waitForTransactionReceipt({ hash });
    ```

=== "ethers"

    ```js linenums="1"
    import { abis, cloak } from '@scroll-tech/cloak-js';

    const c = cloak('local-devnet');

    // configure client with access token...

    const validiumWeth = new Contract(
      c.contracts().ValidiumWeth,
      abis.ERC20,
      l3Wallet,
    );

    const tx = await validiumWeth.transfer(recipient.address, amount);
    ```

By default, this transaction and transaction receipt can only be queried by the sender and the recipient, as well as the chain admins.
Other users cannot access this information.