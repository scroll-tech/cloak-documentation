# Glossary of Technical Terms

* <a id="confidentiality-chain"></a>[**Confidentiality chain**](#confidentiality-chain):
  A private blockchain that ensures confidentiality of user data.
  Usually deployed as an [L3](#l3-chain) [validium](#validium).

* <a id="deposit"></a> [**Deposit**](#deposit):
  The act of moving tokens from the [host chain](#host-chain) to the [confidentiality chain](#confidentiality-chain).

* <a id="host-chain"></a> [**Host chain**](#host-chain):
  The chain on which the [confidentiality chain](#confidentiality-chain) is deployed.
  Typically this is Scroll (and L2), in which case the confidentiality chain is an [L3](#l3-chain).
  The host chain ensures the security of the confidentiality chain, by storing data commitments and verifying [ZK proofs](#zkevm).
  Users can [deposit](#deposit) from and [withdraw](#withdrawal) back to the host chain.

* <a id="l3-chain"></a> [**L3 chain**](#l3-chain):
  A chain deployed on top of an L2 like Scroll, inheriting (part of) its security.

* <a id="rollup"></a> [**Rollup**](#rollup):
  [Rollups](https://ethereum.org/developers/docs/scaling/#rollups) are L2 chains that post data to their host chain (Ethereum L1), as well as validity proofs to ensure that the L2 state transitions were correct.

* <a id="validium"></a> [**Validium**](#validium):
  Just like [rollups](#rollup), [validium](https://ethereum.org/developers/docs/scaling/#validium) chains post validity proofs to their host chain.
  However, validiums do not post data, so 3rd-parties cannot independently rebuild the validium state.

* <a id="withdrawal"></a> [**Withdrawal**](#withdrawal):
  The act of moving tokens from the [confidentiality chain](#confidentiality-chain) back to the [host chain](#host-chain).

* <a id="zkevm"></a> [**zkEVM**](#zkevm):
  A mechanism for producing and verifying validity proofs for EVM execution.
  Succinct validity proofs (also known as ZK proofs) attest that the chain operator executed all transactions correctly, following the EVM protocol rules.
