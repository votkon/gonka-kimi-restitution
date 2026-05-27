# ComputeGroupCap Weight Analysis — Epochs 265–276

## The Issue

The ComputeGroupCap mechanism limits any single model subgroup to
`cap_factor (0.75) × N-1 total network weight`. When the Kimi subgroup's
`confirmation_weight` exceeds this cap, the chain scales down every Kimi
participant's effective weight proportionally — causing systematic underpayment
relative to their actual confirmed work.

The cap was first breached in e265 and remained triggered through e276. The issue
was resolved by the v0.2.13 upgrade (block 4,267,300, mid-e276) which introduced a
lower WeightScaleFactor for Kimi (0.78), reducing its effective confirmation weight
below the 75% threshold. e277 is the first clean epoch.

---

## Kimi Weight vs Network — Full History

| Epoch | Kimi val_w | Kimi conf_w | Total val_w | Kimi val% | Kimi conf% | Participants | Nodes |
|-------|-----------|------------|------------|----------|-----------|-------------|-------|
| e265  | 377,276   | 640,858    | 904,177    | 41.7%    | **70.9%** | 20          | 44    |
| e266  | 59,933    | 115,164    | 335,159    | 17.9%    | 34.4%     | 8           | 9     |
| e267  | 658,820   | 915,743    | 541,415    | 121.7%   | **169.1%**| 27          | 81    |
| e268  | 449,753   | 595,505    | 698,639    | 64.4%    | **85.2%** | 20          | 52    |
| e269  | 450,982   | 594,890    | 679,397    | 66.4%    | **87.6%** | 30          | 56    |
| e270  | 521,085   | 681,025    | 717,467    | 72.6%    | **94.9%** | 23          | 70    |
| e271  | 488,471   | 640,601    | 796,030    | 61.4%    | **80.5%** | 23          | 65    |
| e272  | 508,968   | 658,739    | 823,183    | 61.8%    | **80.0%** | 23          | 65    |
| e273  | 631,147   | 815,951    | 758,715    | 83.2%    | **107.5%**| 30          | 81    |
| e274  | 524,596   | 678,323    | 766,804    | 68.4%    | **88.5%** | 24          | 73    |
| e275  | 589,904   | 763,391    | 736,925    | 80.0%    | **103.6%**| 24          | 80    |
| e276  | 629,257   | 812,469    | 798,029    | 78.9%    | **101.8%**| 23          | 83    |
| e277+ | —         | —          | —          | —        | <75%      | —           | —     |

**Bold** = cap breached (>75% of total network weight).

---

## Compensation Paid Per Epoch

| Epoch | Issue type | Affected | Compensation (GONKA) | % of epoch reward |
|-------|-----------|---------|---------------------|-------------------|
| e265  | CPoC bug (separate) | 3 | 30,592 | 10.7% |
| e266  | Nonce bug (separate) | 18 operators + 9 delegators | 188,698 | 66.3% |
| e267  | ComputeGroupCap | 21 | 88,917 | 31.2% |
| e268  | ComputeGroupCap | 11 | 65,241 | 22.9% |
| e269  | ComputeGroupCap | 19 | 48,013 | 16.9% |
| e270  | ComputeGroupCap | 16 | 30,966 | 10.9% |
| e271  | ComputeGroupCap | 17 | 38,179 | 13.4% |
| e272  | ComputeGroupCap | 11 | 32,434 | 11.4% |
| e273  | ComputeGroupCap | 19 | 50,078 | 17.6% |
| e274  | ComputeGroupCap | 9  | 47,719 | 16.8% |
| e275  | ComputeGroupCap | 15 | 33,939 | 12.0% |

**Total ComputeGroupCap restitution (e267–e275): 435,486 GONKA**

Note: e265 and e266 compensated different bugs. The ComputeGroupCap issue
begins at e267 — the first epoch where Kimi's weight structurally dominated
the network.

---

## How the Weight Became Unbalanced

### e265 — first breach, near miss

Kimi conf_weight reached 70.9% of total network — just below the 75% cap
in absolute terms but enough to trigger the cap mechanism for the largest
participants. Only 3 operators were affected. The network total weight was
still high (~904k) because many Qwen operators were active.

### e266 — collapse and reset

A nonce bug (separate issue) caused most Kimi operators to be excluded.
Only 8 Kimi participants remained, with minimal weight (17.9% val share).
Total network weight dropped sharply to 335k as the Kimi exodus removed
most of the weight. This epoch is anomalous — the cap was not the issue here.

### e267 — severe breach, weight inversion

With Kimi operators returning after the e266 nonce fix, Kimi val_weight
jumped to 658k — **121.7% of total network weight of 541k**. This is a
weight inversion: the Kimi subgroup alone outweighed the entire reported
network total. Kimi conf_weight hit 169.1% of total. The cap triggered hard,
causing 88,917 GONKA in compensation — the largest single-epoch payout.

The inversion occurred because the N-1 total weight used by the cap formula
referenced e266's collapsed network (335k) while e267 had recovered to a much
larger Kimi cohort. This lag in the reference weight amplified the breach.

### e268 — partial recovery

Total network weight recovered to 698k as more Qwen participants joined.
Kimi conf_weight fell to 85.2% of total — still above cap but improving.
Compensation dropped to 65,241 GONKA.

### e269–e272 — stabilisation attempt

Total network weight grew toward 700–820k as Qwen operators joined.
Kimi's conf_weight share oscillated between 80–95%, consistently above the
75% cap but no longer in inversion territory. Compensation settled into the
30–48k GONKA range per epoch.

The e272 dip (ratio 0.882, only 11 affected) coincided with several Kimi
operators being excluded mid-epoch (CPoC failures, statistical invalidations),
temporarily reducing Kimi's effective weight.

### e273 — second spike

Kimi node count jumped from 65 to 81 (30 participants). Kimi conf_weight
reached 107.5% of total — a second inversion. Total network weight had dipped
back to 758k as some Qwen operators churned out. Compensation spiked to 50,078 GONKA.

### e274–e275 — worsening trend

Despite Kimi node count returning to ~73–80, Kimi conf_weight remained above
100% of total network weight in e275 (103.6%). The ratio hit its highest value
yet at 1.284x. The trend shows Kimi's per-node confirmation weight was growing
faster than Qwen network capacity was being added.

### e276 — final affected epoch, partial

v0.2.13 activated at block 4,267,300 (mid-epoch 276). For the pre-upgrade
portion of the epoch, Kimi conf_weight was at 101.8% of total (ratio 1.060),
cap still breached. Compensation is calculated against group data at block
4,267,299. After the upgrade Kimi WSF dropped to 0.78, bringing the ratio
below 0.75 for the remainder of the epoch and all subsequent epochs.

### e277 — resolved

Kimi conf_weight dropped to 51.3% of total network weight — well below the
75% cap. The issue is closed.

---

## conf/val Ratio Trend (e267–e276)

```
e267  1.749x  |###################################
e268  0.854x  |#################
e269  0.997x  |###################
e270  1.226x  |########################
e271  0.948x  |##################
e272  0.882x  |#################
e273  1.177x  |#######################
e274  0.967x  |###################
e275  1.284x  |#########################
e276  1.060x  |#####################  (pre-upgrade only)
e277  0.512x  |##########  RESOLVED
              0                        1.75x
```

- **Mean (e267–e276)**: 1.069x
- **Peak**: e267 at 1.749x (weight inversion)
- **Resolution**: e277 at 0.512x after v0.2.13 lowered Kimi WSF to 0.78

---

## Why Kimi conf_weight Persistently Exceeds val_weight

1. **CPoC accumulation**: Kimi nodes earn `confirmation_weight` through each
   successful CPoC round on top of their base weight. High-capacity nodes with
   many ML nodes accumulate CW much faster than smaller Qwen operators.

2. **Per-node inference volume**: Kimi handles significantly more inference
   requests per epoch due to higher capacity, directly feeding more CPoC
   validations and faster CW growth.

3. **Node count growth outpacing Qwen**: e273 and e275 saw Kimi at 80–81 nodes
   while Qwen participation was flat or declining. Each new Kimi node adds
   directly to the cap-exceeding numerator.

4. **N-1 weight lag**: the cap formula uses the previous epoch's total weight
   as its reference. When Kimi grows quickly between epochs (as in e267), the
   denominator lags behind the actual network state, amplifying the breach.

---

## Final Compensation Summary (e267–e276)

| Epoch | Affected | Compensation (GONKA) | Notes |
|-------|---------|---------------------|-------|
| e267  | 21      | 88,917.16           | weight inversion, worst epoch |
| e268  | 11      | 65,241.37           | |
| e269  | 19      | 48,012.52           | |
| e270  | 16      | 30,965.83           | |
| e271  | 17      | 38,178.79           | |
| e272  | 11      | 32,434.44           | |
| e273  | 19      | 50,077.52           | second inversion |
| e274  | 9       | 47,718.95           | |
| e275  | 15      | 33,938.75           | |
| e276  | 11      | 55,996.81           | partial — pre-upgrade only |
| **TOTAL** | | **491,482.15 GONKA** | |

Issue resolved in e277 following v0.2.13 upgrade (block 4,267,300).
No further compensation required.
