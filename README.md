# Biconomy x Base: PropAMM

Documentation for the propAMM settlement infrastructure Biconomy is bringing to Base. Deployed and verified on Base mainnet, fill path load-validated on Base Sepolia, designed to scale propAMM volumes on Base.

## Contents

- [`docs/propamm-on-base.md`](docs/propamm-on-base.md): partnership context. The problem propAMMs hit on permissionless L2 deployments, what we built to close the dominant toxic-flow vector at the contract layer (ERC-8211 freshness predicates developed with the Ethereum Foundation), why Base is the right operational home, MM commitments, unit economics.
- [`docs/transaction-ordering.md`](docs/transaction-ordering.md): how PropAMM would use Base transaction-ordering primitives to close the intra-block toxic-flow vector, the two integration approaches, why ordering matters for Base, and open questions.
