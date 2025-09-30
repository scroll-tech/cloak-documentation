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


## Slow Withdrawals: Withdrawing via the Canonical Bridge

!!! info "See the full example at [withdraw.ts](https://github.com/scroll-tech/cloak-js/tree/main/examples/viem/withdraw.ts) example in the `@scroll-tech/cloak-js` package."

In all cases, withdrawals are initiated via the `ValidiumERC20Gateway.withdrawERC20AndCall` method.

=== "viem"

    ```js linenums="1" hl_lines="5-5"
    const withdrawalHash = await l3Wallet.writeContract({
      chain: null,
      address: c.contracts().ValidiumERC20Gateway,
      abi: abis.ValidiumERC20Gateway,
      functionName: 'withdrawERC20AndCall',
      args: [l3Token, l2Account.address, amount, '0x', 0n],
    });

    const withdrawalReceipt = await l3Client.waitForTransactionReceipt({
      hash: withdrawalHash
    });
    ```

=== "ethers"

    ```js linenums="1" hl_lines="7-7"
    const erc20Gateway = new Contract(
      c.contracts().ValidiumERC20Gateway,
      abis.ValidiumERC20Gateway,
      l3Wallet,
    );

    const withdrawalTx = await erc20Gateway.withdrawERC20AndCall(
      l3Token,
      l2Wallet.address,
      amount,
      '0x',
      0n,
    );
    ```

This will create a withdraw request on L3, enqueuing it on the `ValidiumMessageQueue` contract.

Once the transaction has been proven and finalized, the withdraw proof can be queried using a special RPC endpoint:

=== "viem"

    ```js linenums="1" hl_lines="3-3"
    const [w] = await l3Client.request({
      // This RPC returns Merkle proofs for finalized withdrawals.
      method: 'scroll_withdrawalsByTransaction',
      params: [withdrawalHash],
    });
    ```

=== "ethers"

    ```js linenums="1" hl_lines="2-2"
    const [w] = await l3Provider.send(
      // This RPC returns Merkle proofs for finalized withdrawals.
      'scroll_withdrawalsByTransaction',
      [txHash],
    );
    ```

Finally, the user can claim on L2 using this withdraw proof:

=== "viem"

    ```js linenums="1" hl_lines="5-5"
    const claimHash = await l2Wallet.writeContract({
      chain: null,
      address: c.contracts().HostMessenger,
      abi: abis.HostMessenger,
      functionName: 'relayMessageWithProof',
      args: [
        w.from,
        w.to,
        BigInt(w.value),
        BigInt(w.nonce),
        w.message,
        { batchIndex: BigInt(w.batch_index), merkleProof: w.proof }
      ],
    });
    ```

=== "ethers"

    ```js linenums="1" hl_lines="7-7"
    const hostMessenger = new Contract(
      c.contracts().HostMessenger,
      abis.HostMessenger,
      l2Wallet,
    );

    const claimTx = await hostMessenger.relayMessageWithProof(
      w.from,
      w.to,
      BigInt(w.value),
      BigInt(w.nonce),
      w.message,
      { batchIndex: BigInt(w.batch_index), merkleProof: w.proof },
    );
    ```


## Fast Withdrawals: Withdrawing via the Fast Withdraw Vault

!!! warning "Fast withdrawals are still experimental."

To use the fast withdraw vault, simply send a withdrawal with these parameters:

- `l2Address` should be the `HostFastWithdrawVault` contract.
- `payload` should be the target address on L2.

=== "viem"

    ```js linenums="1" hl_lines="6-6"
    const withdrawalHash = await l3Wallet.writeContract({
      chain: null,
      address: c.contracts().ValidiumERC20Gateway,
      abi: abis.ValidiumERC20Gateway,
      functionName: 'withdrawERC20AndCall',
      args: [l3Token, fastWithdrawVault/* (1)! */, amount, l2Address/* (2)! */, 0n],
    });

    const withdrawalReceipt = await l3Client.waitForTransactionReceipt({
      hash: withdrawalHash
    });
    ```

    1. Withdraw target address is `HostFastWithdrawVault`, which checks the sequencer permit and releases the funds.
    2. The actual target address should be specified as the withdraw message payload.

=== "ethers"

    ```js linenums="1" hl_lines="9-11"
    const erc20Gateway = new Contract(
      c.contracts().ValidiumERC20Gateway,
      abis.ValidiumERC20Gateway,
      l3Wallet,
    );

    const withdrawalTx = await erc20Gateway.withdrawERC20AndCall(
      l3Token,
      fastWithdrawVault/* (1)! */,
      amount,
      l2Address/* (2)! */,
      0n,
    );
    ```

    1. Withdraw target address is `HostFastWithdrawVault`, which checks the sequencer permit and releases the funds.
    2. The actual target address should be specified as the withdraw message payload.

Next, the sequencer will index this withdraw transaction and run some checks.
If all checks pass, the sequencer will sign a permit authorizing the withdrawal, and it submits it on L2, releasing the tokens to the specified `l2Address`.
