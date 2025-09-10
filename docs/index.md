# Cloak Documentation

Welcome to the documentation site for Cloak — the pragmatic confidentiality solution developed by [Scroll](https://scroll.io).
On this site, you can learn about Cloak's design philosophy and technical details.
We will also show how you can integrate Cloak into your system.

!!! warning

    As of September 2025, Cloak is still experimental, unaudited software.


## What is Cloak?

Rollups and stablecoins are effective tools of financial inclusion:
They offer cheap, secure, programmable access to cross-border transfers, higher yield, and many more services that individuals and companies around the world need.
Projects like Xen, USX, [Ether.fi](https://www.ether.fi), and [Shiga](https://shiga.io) all build towards these goals.

However, in the current web3 landscape, **lack of confidentiality is a major pain point for users**.
Token balances and transactions are all public by default.
Existing privacy tools are complex, expensive, and raise compliance risks.

**Cloak is a pragmatic confidentiality solution**.

* As an [L3](glossary#l3-chain) [validium](glossary#validium) appchain, Cloak proves every transaction using Scroll's [zkEVM](glossary#zkevm) technology, ensuring that state can only be updated according to the protocol rules.

* Cloak achieves confidentiality via fine-grained access control.
  This allows only chain operators and regulators to access the data.
  Users can freely access their own accounts, but they cannot query other users' data.

* Cloak allows interoperability with the [host chain](glossary#host-chain) via fast deposits and fast withdrawals.

Cloak is a reusable technology stack, built using Scroll's battle-tested [rollup](glossary#rollup) technology.
You can deploy new, isolated instances of Cloak on Scroll or any other EVM chain.


## Next Steps

* Visit the [Integration Guide](integration/overview.md) to learn how you can use Cloak in your system.
* Go to the [Technology](technology/introduction.md) section to get an in-depth overview of how Cloak works under the hood.
