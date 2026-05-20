# Gonka Network — Epoch 266–270 Restitution Case

## Overview

This repository contains the restitution analysis for epochs 266 through 270 of the
Gonka network. Across all affected epochs, nonces submitted for the
`moonshotai/Kimi-K2.6` model were not correctly counted when computing participant
weights, causing a significant number of nodes to either be fully excluded from epochs
or to receive far less weight — and therefore far less reward — than they were entitled to.

Root cause: TBD.

Epoch 266 is the primary case and is fully analysed here. Epochs 267 and 268 are complete; 269–270 are in progress.

---

## Epoch Block Heights

| Epoch | PoC Start | Epoch End | Reward Pool |
|-------|-----------|-----------|-------------|
| 266 | 4,105,361 | 4,120,751 | 284,797.19 GONKA |
| 267 | 4,120,752 | 4,136,142 | 284,661.95 GONKA |
| 268 | 4,136,143 | 4,151,533 | 284,526.76 GONKA |
| 269 | 4,151,534 | 4,166,924 | 284,391.65 GONKA |
| 270 | 4,166,925 | 4,182,315 | 284,256.59 GONKA |

Epoch length: 15,391 blocks. Reward formula: `323,000 × e^(−0.000475 × (epoch − 1))` GONKA.

---

## Eligibility Criteria

A participant is eligible for compensation only if they **successfully completed the
epoch** — meaning they both confirmed work (have a `confirmation_weight` on-chain) AND
received actual rewards from the chain. Participants who failed the epoch for any reason
(dropped out of CPoC, invalidated, received zero rewards) are excluded from restitution:
the bug caused underpayment to healthy participants, not an obligation to pay those who
didn't pass.

This applies to all epochs and both compensation types.

---

## Delegation Impact

Gonka uses a PoC delegation system where participants without direct MLNodes can
delegate their consensus weight to an operator for a given model. Delegations are
snapshotted `deploy_window = 500` blocks before the epoch PoC start.

When an operator fails to enter the epoch (e.g. due to the Kimi nonce bug), the chain
resolves all their delegators into **ModeNone** instead of **ModeDelegate**:

| Mode | Condition | Weight effect |
|------|-----------|---------------|
| `ModeDelegate` | Operator entered epoch | −5% transferred to operator |
| `ModeNone` | Operator did not enter | −15% deducted as penalty |

The net extra loss for a delegator whose Kimi operator was excluded is **10%** of their
original consensus weight. This loss is directly attributable to the bug.

Full mechanics documented in [`e266/DELEGATION.md`](e266/DELEGATION.md).

---

## Epoch 266 — Full Analysis

### The Bug

9 participants submitted Kimi nonces and had their commits recorded on-chain, but were
never registered in the epoch group. A further 6 participants entered the epoch but
with their Kimi contribution zeroed out, receiving only Qwen-derived weight.

The on-chain total epoch weight was **335,159** — the correctly reconstructed weight
from all nonce commits is **1,079,698**, revealing that roughly 69% of the network's
true Kimi work was invisible to the reward calculation.

### Excluded Operators (submitted Kimi nonces, not in epoch group)

| Address | Kimi nonces | Reconstructed Kimi weight |
|---------|-------------|--------------------------|
| `gonka1q5xt54wncgzk7dxv9x64uln68455g83wu9tugg` | 89,984 | 113,567.5 |
| `gonka1qa90tgczc0k5dvk4l5nvlf5y6phgm6mg22sfjv` | 55,552 | 70,111.4 |
| `gonka1jrgm47v5eg876udmzg6j6glqcsd5x0vk6crpax` | 25,664 | 32,390.2 |
| `gonka1ujnc662v6g69jm6fgxnr79a2m7ehzeut059239` | 22,272 | 28,109.2 |
| `gonka1xwkesaxvdadh9wt9yyladu0r260s7whklcktds` | 12,896 | 16,275.9 |
| `gonka1c6fwzedfsmpu4jnjekv4cn7mvr7x7fuqd6uqt9` | 12,384 | 15,629.7 |
| `gonka1wkgawwdzj623ss8eywayzdj6qcgr2llygactje` | 6,496 | 8,198.5 |
| `gonka18xeqnspxpg2vncufnjne485rkaagwvz7whyn0d` | 6,080 | 7,673.5 |
| `gonka125n6kr5gvdup0lndfkps7t6rd6592panhrg3np` | 6,048 | 7,633.1 |
| `gonka1yal0ysgzc860zt3y8cds8656tnueusgymftvkw` | 5,664 | 7,148.5 |

### Compensation Summary — Epoch 266

#### Part 1: Nonce compensation (18 participants)

Methodology: reconstruct each participant's weight from on-chain commits, compute their
fair share of the epoch reward pool, subtract actual rewards received.

`compensation = max(0, reconstructed_weight / total_reconstructed_weight × epoch_reward − actual_rewards)`

| Address | Reconstructed weight | Fair share (GONKA) | Actual (GONKA) | Owed (GONKA) |
|---------|---------------------|--------------------|----------------|--------------|
| `gonka17pw6099q758qwzewtrqmqpf5c2lrhr97fwqexu` | 214,919.1 | 56,690.25 | 14,350.37 | **42,339.89** |
| `gonka1q5xt54wncgzk7dxv9x64uln68455g83wu9tugg` | 113,567.5 | 29,956.26 | 0.00 | **29,956.26** |
| `gonka1wthc28t25pg63hzvl07rl8e8r6km6hesl6jhsz` | 109,684.6 | 28,932.04 | 2,148.14 | **26,783.90** |
| `gonka1j7x6dv42xehe9e5au4ku3wvzwtqlegfjhlvzj6` | 72,610.3 | 19,152.78 | 239.63 | **18,913.15** |
| `gonka1qa90tgczc0k5dvk4l5nvlf5y6phgm6mg22sfjv` | 70,111.4 | 18,493.62 | 0.00 | **18,493.62** |
| `gonka10mmdjau4dnj8krs7sh7t7635ttnmq9u3vqgz09` | 64,214.9 | 16,938.28 | 5,869.99 | **11,068.29** |
| `gonka1jrgm47v5eg876udmzg6j6glqcsd5x0vk6crpax` | 32,390.2 | 8,543.71 | 0.00 | **8,543.71** |
| `gonka1ujnc662v6g69jm6fgxnr79a2m7ehzeut059239` | 28,109.2 | 7,414.49 | 0.00 | **7,414.49** |
| *(10 more — see compensation_266_nonces.csv)* | | | | |

**Part 1 total: 183,605.74 GONKA**

#### Part 2: Delegation compensation (9 participants)

9 participants delegated their Kimi weight to `gonka1q5xt54...` (excluded operator).
The chain penalised them 15% (ModeNone) instead of transferring 5% (ModeDelegate).
Net extra weight loss: 10% of original consensus weight per participant.

`compensation = (original_weight × 0.10) / total_epoch_weight × epoch_reward`

| Address | Extra weight lost | Owed (GONKA) |
|---------|------------------|--------------|
| `gonka1tja3g2da45efhe2p83gk3whtussmgmtsdlgprt` | 2,124.3 | **1,805.14** |
| `gonka1hwvel7n3zuk6wruefuzc356l9myske9stckwnz` | 1,854.1 | **1,575.51** |
| `gonka12pcu9mcrpa4w4sjd9y3dsksnvu495ss6f9r4ra` | 1,372.7 | **1,166.44** |
| `gonka1tlvg4kjx7ljd5thgd5fkgh39q6lu8cmxupktgg` | 207.4 | **176.25** |
| `gonka1fvly5jrewyjmjfgwah3khy9rttq4cqajcesv9p` | 134.0 | **113.86** |
| `gonka1cuwejs77gectp3n32wg8q27hlsa4m3hqspf4ww` | 127.5 | **108.37** |
| `gonka1tmk2tzdneht6smu34pkmqdvu7p34qavvmwtwq2` | 118.3 | **100.57** |
| `gonka1gyk0aahvr3qeju4zx0nplfreej6cy4jjk8svc5` | 36.5 | **30.99** |
| `gonka14ef2pxjge75gflqftn7m2wy0xv59gq9uc7qnct` | 18.4 | **15.60** |

**Part 2 total: 5,092.73 GONKA**

### Epoch 266 Grand Total: 188,698.47 GONKA

---

## Epoch 267 — Summary

### The Bug

All Kimi operators entered the epoch, but the Kimi model group's raw weight (**899,487**)
exceeded the ComputeGroupCap (`cap_factor 0.75 × epoch 266 total weight 335,159 = 251,369`).
The chain applied a uniform scale factor of ~0.28× to every Kimi participant's
`ValidationWeight.Weight`, systematically underpaying all Kimi contributors.

No delegation compensation is needed: all operators entered the epoch group.

### Compensation Summary — Epoch 267

Methodology: use `confirmation_weight` (CPoC-verified actual work, unaffected by the cap)
as the fair reward basis.

`compensation = max(0, confirmation_weight / total_confirmation_weight × epoch_reward − actual_rewards)`

| Metric | Value |
|--------|-------|
| Kimi participants in epoch | 27 |
| Affected participants | 21 |
| Excluded (no Kimi commits or failed epoch) | 30 |
| Total compensation | **88,917.16 GONKA** |

Full output: [`e267/compensation_267.csv`](e267/compensation_267.csv) · [`e267/compensation_267.json`](e267/compensation_267.json)

Cap mechanics documented in [`e267/GROUP_CAP.md`](e267/GROUP_CAP.md).

---

## Epoch 268 — Summary

### The Bug

Same ComputeGroupCap issue as epoch 267. The cap baseline rose (using epoch 267's total
weight), but Kimi raw weight grew proportionally, keeping the group above the cap.
The conf/weight ratio of ~0.86× indicates a less severe squeeze than epoch 267
but still resulted in systematic underpayment.

No delegation compensation needed.

### Compensation Summary — Epoch 268

| Metric | Value |
|--------|-------|
| Kimi participants in epoch | 14 |
| Affected participants | 11 |
| Excluded (no Kimi commits or failed epoch) | 46 |
| Total compensation | **65,241.37 GONKA** |

Full output: [`e268/compensation_268.csv`](e268/compensation_268.csv) · [`e268/compensation_268.json`](e268/compensation_268.json)

---

## Running Total (Epochs 266–268)

| Epoch | Compensation |
|-------|-------------|
| 266 | 188,698.47 GONKA |
| 267 | 88,917.16 GONKA |
| 268 | 65,241.37 GONKA |
| 269 | in progress |
| 270 | in progress |
| **Total** | **342,857.00 GONKA** (partial) |

---

## Repository Structure

```
gonka-e26x-issue/
├── README.md                        ← this file
├── poc_commits/                     ← reference: raw nonce commits fetched from chain
│   ├── README.md                    ← fetch commands and block heights
│   ├── epoch266_commits.json
│   ├── epoch267_commits.json
│   ├── epoch268_commits.json
│   └── epoch269_commits.json
├── e266/                            ← epoch 266 full analysis
│   ├── calculate_compensation_266.py  ← single script, fetches all data from chain
│   ├── DELEGATION.md                  ← delegation mechanics and impact analysis
│   ├── compensation_266_nonces.csv    ← Part 1 output
│   ├── compensation_266_delegation.csv ← Part 2 output
│   ├── compensation_266_combined.csv  ← all addresses, both components, totals
│   └── compensation_266.json          ← full structured output
├── e267/                            ← epoch 267 full analysis (ComputeGroupCap)
│   ├── calculate_compensation_267.py
│   ├── GROUP_CAP.md                   ← cap mechanics and impact analysis
│   ├── compensation_267.csv
│   └── compensation_267.json
└── e268/                            ← epoch 268 full analysis (ComputeGroupCap)
    ├── calculate_compensation_268.py
    ├── compensation_268.csv
    └── compensation_268.json
```

Epochs 269–270 will follow the same structure once those epochs conclude.

---

## Running the Analysis

Requires the `inferenced` binary and archive node access. Configure via `.env`:

```
ARCHIVE_NODE_URL=http://<archive-node>:26657
INFERENCED_BINARY=/path/to/inferenced
```

The `.env` is loaded from `../gonka-segment-report/.env` (archive node IP is not
committed to this repo).

```bash
# Epoch 266
python3 e266/calculate_compensation_266.py
```

The script fetches all data live from the archive node (commits, group data,
performance summaries, delegation snapshots) and writes outputs into `e266/`.
The JSON files in `e266/` and `poc_commits/` are reference artifacts produced by
prior runs and kept for auditability.

---

## Chain Query Reference

All queries use the archive node with `--height` pinned to the epoch end block to
read state before on-chain pruning removes it.

```bash
# PoC nonce commits for an epoch
inferenced query inference all-poc-v2-store-commits <poc_start> \
  --node <ARCHIVE_NODE> --height <epoch_end> -o json

# Epoch group data (on-chain weights)
inferenced query inference show-epoch-group-data <epoch> \
  --node <ARCHIVE_NODE> --height <epoch_end> -o json

# Per-participant reward summary
inferenced query inference show-epoch-performance-summary-by-participant \
  <epoch> <address> --node <ARCHIVE_NODE> -o json

# Delegation snapshot (query at poc_start - 500)
inferenced query inference poc-delegation <address> \
  --node <ARCHIVE_NODE> --height <poc_start - 500> -o json
```

| Epoch | poc_start | epoch_end | snapshot_height |
|-------|-----------|-----------|-----------------|
| 266 | 4,105,361 | 4,120,751 | 4,104,861 |
| 267 | 4,120,752 | 4,136,142 | 4,120,252 |
| 268 | 4,136,143 | 4,151,533 | 4,135,643 |
| 269 | 4,151,534 | 4,166,924 | 4,151,034 |
| 270 | 4,166,925 | 4,182,315 | 4,166,425 |
