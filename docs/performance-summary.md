# PropAMM on Base: v3 Sepolia load test results

Measurements from a 10,000-intent consolidated run on Base Sepolia. Every intent was verified on chain per-intent, by matching its witness hash against `IntentSettled` / `IntentFailed` events. To be re-confirmed on mainnet after the external audit.

Headline reliability is measured from the orchestrator-relayed path, the production surface where the orchestrator controls price freshness and relayer nonces. The self-relay surfaces are reported as feature validation, not folded into the headline.

## Headline: orchestrator-relayed (production path)

| Metric | Value |
|---|---:|
| Intents (orchestrator-relayed) | 7,995 |
| **Settled** | **7,995** |
| **Settlement rate** | **100.00%** |
| Protocol failures | 0 |
| Latency p50 / p95 | 4.9 s / 7.9 s |
| Gas per intent | ~83k amortised |

![Per-channel settlement reliability](./perf-summary-throughput.png)

## Feature validation: self-relay surfaces

Every self-relay flow settled end-to-end. These are correctness checks, not the headline number.

| Channel | Submitted | Settled | By-design reverts | Protocol failures | Verdict |
|---|---:|---:|---:|---:|:--:|
| `self-pu-erc20` | 540 | 540 | 0 | 0 | PASS |
| `self-pu-native` | 209 | 209 | 0 | 0 | PASS |
| `self-mockdex` (external venue) | 308 | 308 | 0 | 0 | PASS |
| `self-mixed` (PropAMM + external) | 258 | 258 | 0 | 0 | PASS |
| `self-erc8211` (delegatecall fee-split) | 271 | 271 | 0 | 0 | PASS |
| `piggyback-erc20` | 310 | 282 | 28 | 0 | PASS |
| `piggyback-native` | 109 | 104 | 5 | 0 | PASS |

Piggyback rides the orchestrator's anchor with no price update of its own. Its 33 reverts are by-design: the trader's tx landed in a block with no orchestrator batch. That is the documented piggyback tradeoff, not a failure. Zero protocol failures on any surface.

## Whole-run integrity

| Outcome | Count |
|---|---:|
| Settled | 9,967 |
| By-design reverts (piggyback) | 33 |
| Protocol failures | 0 |
| Harness artifacts | 0 |
| Truly missing | 0 |
| In-flight at cutoff | 0 |

Protocol-attributable settlement across all channels: 9,967 / 9,967 = 100.00%. Raw settled over submitted: 9,967 / 10,000 = 99.67% (the 33 are the by-design piggyback reverts).

## How it was verified

The harness records each intent's witness hash at submit time, and self-relay submissions also record their broadcast tx hash. After the last submission it drains to quiescence: it awaits every self-relay receipt, then waits for orchestrator settlements to go quiet, only then snapshots the final block. It reconciles each hash against on-chain events. A receipt- or watcher-confirmed settlement is never downgraded by a windowed query that misses it. The run had zero false-missing, zero in-flight at cutoff, and zero measurement artifacts.

## Latency

Orchestrator-relayed p50 4.9 s (one Base block plus batcher flush plus per-EOA dispatch queueing under load). Self-relay p50 0.6 to 1.1 s. All-channel p95 under 8 s.

## Gas and cost

![Gas per intent and cost vs batch density](./perf-summary-cost-per-intent.png)

Per-intent gas ranges from ~133k (external venue) to ~233k (composed ERC-8211 route) on self-relay, and around 83k amortised on the orchestrator-relayed path (the batch envelope spread across roughly 2.6 intents per batch).

At current Base gas (L1 base fee around 0.4 gwei, ETH around $2,000), the orchestrator-relayed path is about $0.008 to $0.012 per intent end to end. Cost scales roughly linearly with the L1 base fee.

![Daily orchestrator operating cost](./perf-summary-daily-cost.png)

## Throughput

Sustained submit rate was around 6.5 IPS, bounded by the shared Alchemy RPC tier, not the protocol. With the 32-EOA relayer pool round-robin, the architectural ceiling lifts substantially on a dedicated RPC tier.

![Notional throughput capacity](./perf-summary-scale.png)

## What this run validates

- Orchestrator-relayed settlement at 100% under sustained load
- Every self-relay surface settles end-to-end (self-PU, piggyback, cross-venue, mixed, ERC-8211)
- Market makers own their output pricing; the protocol enforces only the user's minimum
- A measurement framework with zero false-missing, in-flight, or artifacts across 10,000 intents

## What this run does NOT validate

- Higher target IPS, which needs a dedicated RPC tier
- Mainnet gas in USD (Base Sepolia's near-zero gas floor; the per-intent gas counts transfer, the USD figures are a model)
- External audit findings, which will be re-validated after the audit
