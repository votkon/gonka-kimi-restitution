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
