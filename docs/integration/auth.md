# Authentication

## Introduction

Cloak achieves confidentiality via fine-grained access control.
While Cloak supports the standard Ethereum [JSON-RPC](https://ethereum.org/developers/docs/apis/json-rpc) APIs, clients must present a [bearer token](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication#bearer) to access data.
The token scope determines what data the client can or cannot access.

```bash title="Example: Unauthorized access"
cast rpc --rpc-url https://validium-devnet-rpc.scroll.systems eth_getBalance 0x5FDD39Bd675B5951BE2e58741c1910573A5774F1 latest
```

```bash
Error: server returned an error response:
error code -32603: unauthorized, data: "unauthorized" # (1)!
```

1. The RPC method `eth_getBalance` is token-gated.
   We did not specify a bearer token, so we get an `unauthorized` error.


## Admin Tokens

If you access a Cloak instance via a secure backend, then you can acquire an admin token.
In the following examples, we use the example admin token `admin-token-1-abcdefg`.

=== "cast"

    ```bash title="Example: Admin token"
    cast rpc --rpc-url https://validium-devnet-rpc.scroll.systems --rpc-headers "Authorization: Bearer admin-token-1-abcdefg" eth_getBalance 0x5FDD39Bd675B5951BE2e58741c1910573A5774F1 latest
    ```

=== "viem"

    ```js title="Example: Admin token" linenums="1"
    const { createPublicClient, http } = require('viem');

    const endpoint = 'https://validium-devnet-rpc.scroll.systems';
    const token = 'admin-token-1-abcdefg';
    const address = '0x5FDD39Bd675B5951BE2e58741c1910573A5774F1';

    const opts = {
      fetchOptions: {
        headers: {
          Authorization: `Bearer ${token}`,
        }
      }
    };

    const client = createPublicClient({ transport: http(endpoint, opts) });

    await client.getBalance({ address });
    ```

=== "ethers-js"

    ```js title="Example: Admin token" linenums="1"
    const { ethers } = require('ethers');

    const endpoint = 'https://validium-devnet-rpc.scroll.systems';
    const token = 'admin-token-1-abcdefg';
    const address = '0x5FDD39Bd675B5951BE2e58741c1910573A5774F1';

    const request = new ethers.FetchRequest(endpoint);
    request.setHeader('Authorization', `Bearer ${token}`);
    const provider = new ethers.JsonRpcProvider(request);

    const balance = await provider.getBalance(address);
    ```

Requests using an admin token can access all data.

!!! warning "Warning about admin token security"

    Admin tokens allow access to sensitive data. It is important that they are stored securely and rotated regularly.


## SIWE JWT Tokens

Cloak supports [Sign-In with Ethereum](https://docs.login.xyz) (SIWE), which allows users to authenticate using their wallet.
The user must sign a message proving that they have access to their private key.
After verifying the signature, the RPC endpoint issues a scoped [JWT token](https://auth0.com/docs/secure/tokens/json-web-tokens).
Using this token, the user can then access data related to their own account.

The login flow is the following:

``` mermaid
sequenceDiagram
  autonumber
  client->>rpc: siwe_getNonce
  Note right of rpc: Generate unique nonce
  rpc-->>client: nonce
  Note left of client: Construct and sign <br> SIWE message
  client->>rpc: siwe_signIn
  Note right of rpc: Verify message and signature
  rpc-->>client: jwt token
  Note left of client: Configure token <br> on provider
  client->>rpc: eth_getBalance
  rpc-->>client: 1234
```

You can use any [SIWE libraries](https://docs.login.xyz), or any library capable of EIP-712 typed structured data hashing and signing.
Below is an example using siwe-js with `ethers`.

1. Acquire a unique nonce via `siwe_getNonce`.

    ```js title="Example: SIWE with browser wallet (1/3)" linenums="1" hl_lines="14-14"
    const { ethers } = require('ethers');
    const { SiweMessage } = require('siwe');

    const endpoint = 'https://validium-devnet-rpc.scroll.systems';

    const wallet = new ethers.BrowserProvider(window.ethereum);
    const provider = new ethers.JsonRpcProvider(endpoint);

    // connect wallet
    await wallet.send('eth_requestAccounts', []);
    const signer = await wallet.getSigner();

    // fetch nonce
    const nonce = await provider.send('siwe_getNonce', []);
    ```

2. Construct and sign SIWE message.

    ```js title="Example: SIWE with browser wallet (2/3)" linenums="16" hl_lines="17-17"
    // construct siwe message
    const network = await provider.getNetwork();

    const draft = new SiweMessage({
      domain: window.location.host,
      address: signer.address,
      statement: 'Sign in with Ethereum to the app.',
      uri: endpoint,
      version: '1',
      chainId: network.chainId,
      nonce,
    });

    const message = draft.prepareMessage();

    // prompt wallet signature
    const signature = await signer.signMessage(message);
    ```

3. Submit signature via `siwe_signIn` to acquire a JWT token.

    ```js title="Example: SIWE with browser wallet (3/3)" linenums="34" hl_lines="2-2"
    // submit signature to acquire jwt token
    const jwt = await provider.send('siwe_signIn', [message, signature]);

    // create authenticated provider
    const request = new ethers.FetchRequest(endpoint);
    request.setHeader('Authorization', `Bearer ${jwt}`);
    const authProvider = new ethers.JsonRpcProvider(request);

    const balance = await authProvider.getBalance(signer.address);
    ```
