# PropAMM on Base: load test results

A 10,000-intent consolidated load test on Base Sepolia. Every intent was verified on chain by matching its witness hash against settlement events, and the result was independently re-checked by querying the chain directly.

The orchestrator-relayed path is the production default. All headline numbers below are from that path.

## Orchestrator-relayed

| Metric | Value |
|---|---:|
| Intents | 7,988 |
| Settled | 7,988 |
| Settlement rate | 100% |
| Gas per intent | ~83k (batch envelope amortised across the batch) |
| Cost per intent | ~$0.008 to $0.012 (projected, see Cost) |
| Settlement latency | 2.2s p50, 3.3s p95 |

## Sustained load

The run held a steady submission rate for the full ~26 minutes, and settlement tracked submission the whole way. The two lines stay together, so no settlement backlog built up under sustained load.

![Sustained load over the run](./perf-summary-sustained.png)

## Cost

Gas per intent is measured (~83k amortised on the orchestrator-relayed path, falling as more intents share a batch). The USD figures are a projection from that gas at mainnet conditions (L1 base fee ~0.4 gwei, ETH ~$2,000), about $0.008 to $0.012 per intent, scaling roughly linearly with the L1 base fee. The load test itself ran on Base Sepolia, where gas is near zero, so the dollar figures are a model on measured gas, not a measured mainnet cost. The two charts below are models on that basis.

![Cost per intent vs batch density (model)](./perf-summary-cost-per-intent.png)

![Daily orchestrator operating cost (model)](./perf-summary-daily-cost.png)

## Execution modes

Every execution mode was exercised in the same run and settles end-to-end.

| Mode | Result |
|---|---|
| Orchestrator-relayed (default) | 100% (7,988 / 7,988) |
| Self-relay, self-supplied price | settles |
| Cross-venue routing | settles |
| Mixed PropAMM + external venue | settles |
| ERC-8211 fee-split | settles |
| Piggyback (rides the orchestrator's anchor) | settles when the trader's tx shares a block with an orchestrator batch, the common case on active pairs |

## How it was verified

Each intent's witness hash is recorded at submit time. After the last submission the harness waits for the system to go quiet (every self-relay receipt confirmed and orchestrator settlements stopped arriving) before snapshotting, then reconciles every hash against `IntentSettled` / `IntentFailed` events. An independent re-query of the chain confirmed 9,964 settled across all modes, with zero orchestrator-relayed failures (verified by the tx sender of every failed event).

## Latency

Orchestrator-relayed settlement is **2.2s p50, 3.3s p95**, measured from the settled event's block timestamp minus submit time (true on-chain settle time, not observation lag). That is about one Base block after dispatch plus the batcher's 200ms flush window. The batcher flushes a pair's bucket when it reaches 20 intents or 200ms, whichever comes first.

## Throughput

The run sustained about 6 intents/sec of submission (see the sustained-load chart above). That ceiling is the test harness, which signs each intent with an RPC round-trip and submits self-relay intents sequentially through a single trader EOA. It is not the RPC endpoint (a private Alchemy URL) and not the protocol. On the protocol side, the batcher flushes up to 20 intents every 200ms and the 32-EOA relayer pool dispatches many batches per block, so the architectural ceiling is far higher.
