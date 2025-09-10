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


## Executing Deposits

The deposit process is as follows.

1. Fetch current encryption key.

    ```js linenums="1"
    const encryptionKey = await l2Client.readContract({
        abi,
        address: contracts.l2.Validium,
        functionName: 'encryptionKey',
    });
    ```

2. Encrypt target address using the encryption key.

    ```js linenums="1"
    const ecies = require('eciesjs');

    function encryptAddress(address) {
      const plaintext = Buffer.from(address.replace(/^0x/, ''), 'hex');
      const ciphertext = ecies.encrypt(config.EncryptionKey, plaintext);
      return '0x' + ciphertext.toString('hex');
    }

    const l3Account = privateKeyToAccount(generatePrivateKey());// (1)!
    const addressCiphertext = encryptAddress(l3Account.address);
    ```

    1. It is recommended to generate a unique L3 address for the user.
       One that is not used on another networks.

3. Send deposit to L2 bridge.

    ```js linenums="1"
    await l2Wallet.sendTransaction({
      to: contracts.l2.WethGateway,
      data: encodeFunctionData({
          abi,
          functionName: 'deposit',
          args: [addressCiphertext, amount],
      }),
      value: amount + parseEther('0.001'),// (1)!
    });
    ```

    1. deposit amount + fee (remaining fee is refunded)


4. Wait for deposit to be confirmed on L3.

  TBA


## ERC20 Token Mapping

The Scroll-Cloak bridge can permissionlessly bridge any valid ERC20 token.
The bridge will deterministically compute the corresponding L3 token address, which will not change after the first deposit.
L2 ETH is bridged into L3 Wrapped ETH.

!!! info "ETH deposits"

    By default, L2 ETH is deposited into L3 Wrapped ETH (ERC20 token), not into the L3 gas token.
    L3 gas is free, so users do not need to hold any gas token.