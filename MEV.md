# Maximal Extractable Value (MEV): Summary and Notes

## Overview
- **MEV**: Profit extracted by miners/validators by reordering, including, or excluding transactions in a blockchain block.
- **Main Activities**:
  1. **Arbitrage**: Exploiting price differences across Decentralized Exchanges (DEXs).
  2. **Liquidations**: Closing positions when certain conditions are met.
  3. **Sandwiching**: Manipulating trades by placing buy/sell orders around them.

## Key Issues
- MEV poses risks to Ethereum’s **network stability**.
- Leads to **frontrunning**, undermining user trust.
- Centralization risks arise due to MEV concentration.

## Key Statistics
- **2020-2021**: Over $773M earned through MEV on Ethereum.
- **2021**: Ethereum miners alone earned $730M from MEV.
- **Projection**: Over $750M in annual MEV revenue.

## Solutions
### Flashbots and Community Efforts
- **Flashbots**: Tools for democratizing MEV access and reducing risks.
- Key approaches to mitigate MEV:
  1. Updates to **Ethereum’s consensus protocol**.
  2. Improvements to **decentralized application (dApp) design**.
  3. **Education** about on-chain MEV strategies.

### Transition to Proof-of-Stake (PoS)
- Ethereum’s transition from **Proof-of-Work (PoW)** to **Proof-of-Stake (PoS)** will shift transaction reordering powers from miners to validators.

## MEV’s Dual Nature
- **Positive Impacts**:
  - Enhances **market efficiency** by addressing price inconsistencies.
  - Encourages **participation and transparency**.
- **Negative Impacts**:
  - Disrupts **network consensus**.
  - Introduces unforeseen **slippage** or **attacks** on user trades.

## Observations
- MEV is an inevitable byproduct of Ethereum’s **permissionless and Turing-complete design**.
- Mitigation efforts need to balance transparency and avoid unintended consequences, such as driving activity into dark pools.


`MEV` is both beneficial and problematic, as described in the report:

## Positive Aspects

- Improves market efficiency by addressing price discrepancies.
- Promotes participation and transparency in decentralized finance (DeFi).

## Negative Aspects
- Undermines network stability through manipulative practices like frontrunning.
- Risks centralization and user trust due to excessive miner/validator control.

The report concludes MEV is an inevitable byproduct of Ethereum’s design. The key is to mitigate its harms while fostering transparency, ensuring it benefits the ecosystem without destabilizing it. https://www.galaxy.com/insights/research/mev-how-flashboys-became-flashbots/

### Links to check out
https://x.com/bertcmiller/status/1463195221995495432

https://github.com/bnb-chain/bsc/issues/658

https://arxiv.org/abs/2112.01472

https://www.reddit.com/r/ethereum/comments/2d84yv/miners_frontrunning/?rdt=65313

https://ethresear.ch/t/proposer-block-builder-separation-friendly-fee-market-designs/9725


---

## Diagrams

### MEV Activities
```plaintext
+-------------------+      +------------------+      +------------------+
| Arbitrage         | ---> | Liquidations     | ---> | Sandwiching       |
+-------------------+      +------------------+      +------------------+
(Profits from DEXs)        (Forced closures)          (Manipulating trades)

Proof-of-Work (PoW) --> Miners control transaction ordering
                    --> Proof-of-Stake (PoS) --> Validators take over ordering.

