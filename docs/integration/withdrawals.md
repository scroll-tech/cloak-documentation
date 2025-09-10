# Withdrawing Funds

## Introduction

Withdrawals are the counterpart of [deposits](/integration/deposits):
they are transactions that move tokens from Cloak (L3) back to the host chain (Scroll).

Cloak offers two ways to withdraw funds:

1. **Slow withdrawals**:
   Same mechanism as Scroll:
   The user submits a withdraw transaction on L3, waits for it to be finalized (validity proof submitted on the host chain), then claims on L2 using a Merkle proof.
   This withdraw path has guaranteed liquidity from the L2 bridge (since assets are locked 1:1), but having to wait for finalization makes it slow.

2. **Fast withdrawals**:
   Alternatively, we offer a sequencer-maintained liquidity bridge.
   The user submits a withdraw transaction to the host `FastWithdrawVault` contract.
   The Cloak sequencer checks that the transaction is valid.
   If the checks are passed, the sequencer immediately releases funds to the user on L2.
   This withdraw path is very fast, but requires liquidity management from the sequencer, and as such it might fail if the vault is depleted.

We recommend using **fast withdrawals** for frequent, low-value withdrawals, and **slow withdrawals** for large-value transactions.

## Initiating Withdrawals

In all cases, withdrawals are initiated via the `ValidiumERC20Gateway.withdrawERC20AndCall` method.

```js linenums="1"
const hash = await l3Wallet.writeContract({
    abi,
    address: contracts.l3.ERC20Gateway,
    functionName: 'withdrawERC20AndCall',
    args: [l3Token, l2Address, amount, payload, 0],

    // gas is free
    maxFeePerGas: 0,
    maxPriorityFeePerGas: 0,
  });

  const receipt = await l3Client.waitForTransactionReceipt({ hash });
```

This will enqueue a withdraw message on `ValidiumMessageQueue`.


## Withdrawing via the Fast Withdraw Vault

To use the fast withdraw vault, simply send a withdrawal with these parameters:
- `l2Address` should be the `HostFastWithdrawVault` contract.
- `payload` should be the target address on L2.

```js linenums="1" hl_lines="5-5"
const hash = await l3Wallet.writeContract({
    abi,
    address: contracts.l3.ERC20Gateway,
    functionName: 'withdrawERC20AndCall',
    args: [l3Token, fastWithdrawVault/* (1)! */, amount, l2Address/* (2)! */, 0],

    // gas is free
    maxFeePerGas: 0,
    maxPriorityFeePerGas: 0,
  });

  const receipt = await l3Client.waitForTransactionReceipt({ hash });
```

1. Withdraw target address is `HostFastWithdrawVault`, which checks the sequencer permit and releases the funds.
2. The actual target address should be specified as the withdraw message payload.

Next, the sequencer will index this withdraw transaction and run some checks.
If all checks pass, the sequencer will sign a permit authorizing the withdrawal, and it submits it on L2, releasing the tokens to the specified `l2Address`.

**TBA: Finding the withdraw transaction on L2**


## Withdrawing via the Canonical Bridge

**TBA**

1. Withdraw on L3.
2. Wait for finalization.
3. Claim via Merkle proof.