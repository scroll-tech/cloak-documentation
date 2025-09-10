# Making Transfers inside Cloak

## Introduction

Cloak is an exact copy of Scroll's EVM.
As it is a full fledged EVM execution environment, one can do ERC20 token transfers or interact with smart contracts.

The simplest [use case](/integration/use-cases) is to let users hold tokens in a wallet on Cloak (similar to a shielded account) and to make transfers to other users.
In this document we will present this use case.


## Making Token Transfers

To make ERC20 transfers (for L3 WETH or any other deposited token), simply call the token contract `transfer` function, just like you would on other EVM chains.

```js linenums="1"
const hash = await l3Wallet.writeContract({
  abi,
  address: deposit.l3Token,
  functionName: 'transfer',
  args: [address, parseEther('0.01')],

  // gas is free
  maxFeePerGas: 0,
  maxPriorityFeePerGas: 0,
});

const receipt = await l3Client.waitForTransactionReceipt({ hash });
```

By default, this transaction and transaction receipt can only be queried by the sender and the recipient, as well as the chain admins.
Other users cannot access this information.