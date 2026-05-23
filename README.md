# Gonka Network — Epoch 265–270 Restitution Case

## Overview

This repository contains the restitution analysis for epochs 265 through 270 of the
Gonka network. Across all affected epochs, nonces submitted for the
`moonshotai/Kimi-K2.6` model were not correctly counted when computing participant
weights, causing a significant number of nodes to either be fully excluded from epochs
or to receive far less weight — and therefore far less reward — than they were entitled to.

Root cause: TBD.

The issue was first observed in epoch 265, where CPoC confirmation weights for Kimi
participants dropped abnormally at block **4,103,171**. Epoch 266 is the primary nonce
exclusion case and is fully analysed. Epochs 265–271 are complete.

---

## Epoch Block Heights

| Epoch | PoC Start | Epoch End | Reward Pool |
|-------|-----------|-----------|-------------|
| 265 | 4,089,970 | 4,105,360 | 284,932.49 GONKA |
| 266 | 4,105,361 | 4,120,751 | 284,797.19 GONKA |
| 267 | 4,120,752 | 4,136,142 | 284,661.95 GONKA |
| 268 | 4,136,143 | 4,151,533 | 284,526.76 GONKA |
| 269 | 4,151,534 | 4,166,924 | 284,391.65 GONKA |
| 270 | 4,166,925 | 4,182,315 | 284,256.59 GONKA |

Epoch length: 15,391 blocks. Reward formula: `323,000 × e^(−0.000475 × (epoch − 1))` GONKA.

---

## Eligibility Criteria

Eligibility rules differ by epoch type:

**Epochs 265–266 (nonce exclusion):** All participants who submitted valid Kimi nonces
are eligible, including those who received zero rewards because the bug locked them out
entirely. Zero rewards here is a direct consequence of the bug, not a failure by the
participant.

**Epochs 267–270 (ComputeGroupCap underpayment):** A participant is eligible only if
they **successfully completed the epoch** — meaning they both confirmed work
(`confirmation_weight` on-chain) AND received actual rewards. Participants who failed
the epoch for any reason (dropped out of CPoC, invalidated, received zero rewards) are
excluded: the bug caused underpayment to healthy participants, not an obligation to pay
those who didn't pass.

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

## Epoch 265 — Summary

### The Bug

All Kimi operators entered the epoch group — no exclusions. However, CPoC confirmation
weights for Kimi participants dropped abnormally mid-epoch. Binary search on the archive
node pinpointed the exact block:

| State | Block | `gonka1j7x6dv42...` confirmation_weight |
|-------|-------|----------------------------------------|
| Last healthy | **4,103,170** | 66,311 |
| First corrupted | **4,103,171** | 323 |

3 participants had a CW drop >5% attributable to the bug. Compensation uses the
standard epoch reward formula — `weight / total_epoch_weight × epoch_reward` —
applied at the victim's actual epoch weight, minus actual rewards received.
Total epoch weight (verified constant at 904,177 from block 4,095,000 through epoch end).

`fair_reward = weight / total_epoch_weight × epoch_reward`
`compensation = max(0, fair_reward − actual_rewards)`

### Compensation Summary — Epoch 265

| Address | weight | cw@healthy | cw@end | drop | Owed (GONKA) |
|---------|--------|-----------|--------|------|-------------|
| `gonka1j7x6dv42xehe9e5au4ku3wvzwtqlegfjhlvzj6` | 66,311 | 66,311 | 323 | 99.5% | **20,896.53** |
| `gonka17pw6099q758qwzewtrqmqpf5c2lrhr97fwqexu` | 189,884 | 186,719 | 172,607 | 7.6% | **5,444.49** |
| `gonka1830lqug50lse998x2lakk4pj5ypfumz5pasz0y` | 13,490 | 7,031 | 0 | 100% | **4,251.09** |

**Epoch 265 Total: 30,592.10 GONKA**

Full output: [`e265/compensation_265.csv`](e265/compensation_265.csv) · [`e265/compensation_265.json`](e265/compensation_265.json)

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

## Epoch 269 — Summary

### The Bug

Same ComputeGroupCap issue as epochs 267–268. Kimi group weight again exceeded the cap
(`cap_factor 0.75 × epoch 268 total weight`), applying a uniform scale factor to all
Kimi participants' `ValidationWeight.Weight`. CW/weight ratio ~1.00× at epoch end
reflects the cap still in effect.

No delegation compensation needed.

### Compensation Summary — Epoch 269

| Metric | Value |
|--------|-------|
| Kimi participants in epoch | 26 |
| Affected participants | 19 |
| Excluded (no conf_w or failed epoch) | 7 |
| Total compensation | **48,012.52 GONKA** |

Full output: [`e269/compensation_269.csv`](e269/compensation_269.csv) · [`e269/compensation_269.json`](e269/compensation_269.json)

---

## Epoch 270 — Summary

### The Bug

Same ComputeGroupCap issue as epochs 267–269. Scale factor of 0.755 — worse than
projected due to higher Kimi nonce volume (549,664 vs ~438k average). CW/weight
ratio of 1.23× confirms significant underpayment.

No delegation compensation needed.

### Compensation Summary — Epoch 270

| Metric | Value |
|--------|-------|
| Kimi participants in epoch | 22 |
| Affected participants | 16 |
| Excluded (no conf_w or failed epoch) | 3 |
| Total compensation | **30,965.83 GONKA** |

Full output: [`e270/compensation_270.csv`](e270/compensation_270.csv) · [`e270/compensation_270.json`](e270/compensation_270.json)

---

## Epoch 271 — Summary

### The Bug

Same ComputeGroupCap issue. Scale factor ~0.86× (cap 538k vs raw 625k). CW/weight
ratio of 0.95× at epoch end — cap still firing despite the ratio inverting, because
the chain's weight formula applies the cap before CPoC confirmation runs.

No delegation compensation needed.

### Compensation Summary — Epoch 271

| Metric | Value |
|--------|-------|
| Kimi participants in epoch | 20 |
| Affected participants | 17 |
| Excluded (no conf_w or failed epoch) | 3 |
| Total compensation | **38,178.79 GONKA** |

Full output: [`e271/compensation_271.csv`](e271/compensation_271.csv) · [`e271/compensation_271.json`](e271/compensation_271.json)

---

## Running Total (Epochs 265–271)


| Epoch | Compensation |
|-------|-------------|
| 265 | 30,592.10 GONKA |
| 266 | 188,698.47 GONKA |
| 267 | 88,917.16 GONKA |
| 268 | 65,241.37 GONKA |
| 269 | 48,012.52 GONKA |
| 270 | 30,965.83 GONKA |
| 271 | 38,178.79 GONKA |
| **Total** | **490,606.24 GONKA** |

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
├── e265/                            ← epoch 265 analysis (CPoC degradation)
│   ├── calculate_compensation_265.py
│   ├── compensation_265.csv
│   └── compensation_265.json
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
├── e268/                            ← epoch 268 full analysis (ComputeGroupCap)
│   ├── calculate_compensation_268.py
│   ├── compensation_268.csv
│   └── compensation_268.json
├── e269/                            ← epoch 269 full analysis (ComputeGroupCap)
│   ├── calculate_compensation_269.py
│   ├── compensation_269.csv
│   └── compensation_269.json
├── e270/                            ← epoch 270 full analysis (ComputeGroupCap)
│   ├── calculate_compensation_270.py
│   ├── compensation_270.csv
│   └── compensation_270.json
└── e271/                            ← epoch 271 full analysis (ComputeGroupCap)
    ├── calculate_compensation_271.py
    ├── compensation_271.csv
    └── compensation_271.json
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

| Epoch | poc_start | epoch_end | snapshot_height | cpoc_cutoff |
|-------|-----------|-----------|-----------------|-------------|
| 265 | 4,089,970 | 4,105,360 | 4,089,470 | 4,103,170 |
| 266 | 4,105,361 | 4,120,751 | 4,104,861 | — |
| 267 | 4,120,752 | 4,136,142 | 4,120,252 | — |
| 268 | 4,136,143 | 4,151,533 | 4,135,643 | — |
| 269 | 4,151,534 | 4,166,924 | 4,151,034 | — |
| 270 | 4,166,925 | 4,182,315 | 4,166,425 | — |
