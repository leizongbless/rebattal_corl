# VPCE-DP Rebuttal: Real-Robot Facts and Handoff

Last reconciled: 2026-08-12 CST. This document is the operational source of
truth for rebuttal writing and real-robot evaluation. Do not pool results across
rows unless the checkpoint SHA, setup mode, success rule, and trial manifest
match.

## Non-negotiable rules

1. `hand_stabilized` / operator-held and fixed-mount results are different
   protocols. Never place them in the same success-rate column or denominator.
2. A completed inference CSV is not a success label. Count a trial only after
   confirming one full card-reader roller rotation within 60 seconds.
3. Do not infer success from action traces, process exit status, or CSV count.
   Keep invalid trials in the manifest and replace them only for the planned
   valid-trial count.
4. Do not pool early N=5 pilots, intermediate snapshots, cumulative oral
   aggregates, and later reruns. They are different batches unless a per-trial
   manifest proves otherwise.
5. Every counted trial must record timestamp, profile, checkpoint SHA, setup
   mode, valid/invalid status, success/failure, failure mode, video path, and
   whether a person touched the prop after rollout start.

## Frozen submitted-paper results

These are the original paper results. They should remain distinct from rebuttal
experiments.

| Method | SWIPE | WIPE | INSERT | Protocol |
|---|---:|---:|---:|---|
| DP-V | 1/20 (5%) | 0.45 mean erased-area fraction | 0/20 (0%) | Original paper evaluation |
| DP-VT | 2/20 (10%) | 0.60 mean erased-area fraction | 3/20 (15%) | Original paper evaluation |
| VPCE-DP-V | 17/20 (85%) | 0.50 mean erased-area fraction | 6/20 (30%) | Original paper evaluation |
| VPCE-DP | 18/20 (90%) | 0.70 mean erased-area fraction | 9/20 (45%) | Original paper evaluation |

For SWIPE and INSERT, success is binary. WIPE reports erased-area fraction; its
binary success threshold is at least 90% erased area and must not be substituted
for the reported mean without a new trial-level annotation pass.

## Offline VPCE versus time-only diagnostics

These are held-out contact-prediction diagnostics, not policy rollouts.

| Task | Phase-4 AUC/F1 | Progress-20 AUC/F1 | VPCE AUC/F1 |
|---|---:|---:|---:|
| SWIPE | .670/.000 | .853/.597 | .963/.759 |
| WIPE | .666/.000 | .743/.406 | .885/.705 |
| INSERT | .684/.000 | .808/.539 | .946/.735 |

Use only this claim: phase/progress encode nominal elapsed task order, whereas
VPCE is an online, state-conditioned 15-step future-contact horizon predicted
from wrist RGB and recent robot motion. Progress explains part of timing, but
not all of the contact signal. Do not claim policy-level superiority over
phase/progress until their corrected runtime conditioning and per-trial outcomes
are audited.

## Transfer boundary

| Teacher evaluation | AUC/F1 | Interpretation |
|---|---:|---|
| Source teacher -> board, zero-shot | .500/.095 | No useful zero-shot board transfer |
| Source teacher -> card, zero-shot | .662/.666 | Partial ranking transfer only |
| Target-adapted board teacher | .897/.750 | Target adaptation restores useful prediction |

The target sets contain 54/59/61 tactile trajectories for WIPE/SWIPE/INSERT.
The deployed WIPE teacher uses BCF joint retraining over all 174 target
trajectories (148/26 train/val), while SWIPE and INSERT use single-task scratch
splits of 47/12 and 49/12. Contact labels are automatically derived from
recorded tactile signals; there is no manual frame-level annotation. Do not
claim a universal or task-agnostic zero-shot contact representation.

## Auditable rebuttal component checkpoints and results

All rows below use the card/SWIPE task. The results are operator-held unless a
later trial manifest explicitly says `fixed_mount`. They are useful for
mechanism diagnosis but are not fixed-environment robustness results.

| Isolated change from Legacy | Checkpoint SHA-256 | Modes | Result status | Result |
|---|---|---|---|---:|
| Fixed loss, alpha=1 | `94968a52...3217ee` | `legacy_whole_lowdim`, `legacy_unbounded`, `fixed_alpha1` | Operator-reported intermediate snapshot; checkpoint verified | 10/10 |
| Tactile-only post-encoder mask | `71d0531b...528eb` | `tactile_only`, `legacy_unbounded`, `legacy_learned` | Operator-reported aggregate; checkpoint verified | 13/20 |
| Bounded gate | `f9a6d974...86d4c1` | `legacy_whole_lowdim`, `bounded`, `legacy_learned` | Operator-reported aggregate; checkpoint verified | 7/20 |
| Legacy Full | original `card_vpce_vt_dp_9d` | Legacy | Historical hand-stabilized pilot | 4/5 |

Important interpretation:

- The learned loss weight was ill-posed: checkpoint inspection gives weights
  around `1.000--1.005` for Card (`w(1)=1.0047`). The lower bound prevents
  below-baseline weighting, but the learned weighting becomes ineffective. This
  collapse is task-dependent: original Board and Flower retain `w(1)=2.119`
  and `4.436`. The corrected parameter-free Card schedule is
  `w(c)=clip(1+c,1,2)`.
- `10/10` is a specific intermediate snapshot of the **Legacy + fixed Alpha1
  only** checkpoint. It is not a result for the clean tactile-only-mask policy,
  not the submitted Full model, and not evidence that Alpha1 beats Legacy Full.
- The latest operator-reported cumulative aggregates are approximately Legacy
  Full `18/20` and Legacy + fixed Alpha1 `20/25`. These lack a complete
  per-trial outcome manifest and setup re-confirmation, so label them
  **operator-reported preliminary aggregates** only. Do not pool them with the
  `10/10` snapshot or use them for a formal paired comparison.
- The current evidence supports horizon conditioning and tactile-aware masking
  as the central architecture story. Loss weighting is a secondary training
  correction. Gating is auxiliary and task-dependent: it is not supported as a
  necessary source of binary success.
- The submitted Legacy masking ablation is Full `9/10` versus no masking
  `0/10`. That switch removes both input tactile masking/dropout and the old
  post-encoder whole-lowdim suppression. It supports the combined Legacy
  masking pathway, but it does not isolate corrected tactile-only masking.
- The submitted Legacy no-gating result is `9/10`, equal to Full `9/10` in the
  small diagnostic. Checkpoint probing also shows the tested bounded gate
  saturates many high-contact dimensions at visual/tactile scales `0.5/2.5`.
  Therefore write that the tested gating is too aggressive or auxiliary, not
  that every possible tactile gate is useless.
- The author-reported 10-trial Alpha sweep is alpha=1 `10/10`, alpha=.5
  `7/10`, alpha=.25 `6/10`, and alpha=2 `3/10`. The remote audit does not bind
  the alpha=.5/.25 outcomes to the newly downloaded Legacy checkpoint SHAs,
  whose canonical ledger still says robot outcome TBD. Treat the sweep as an
  author-reported matched batch until its exact checkpoint/profile and trial
  rows are restored; never infer that the new `.5/.25` checkpoints already
  produced those outcomes.

## Checkpoints available on hkust-z790

Root: `~/fyt/workspace/VPCE/re/rebuttal_corrected_checkpoints_20260809`.
Use `latest.ckpt` only after confirming no `.tmp` file is present.

| Folder | SHA-256 prefix | Purpose | Outcome status |
|---|---|---|---|
| `card_legacy_plus_fixed_alpha1_loss_seed42` | `94968a52` | Legacy + fixed `1+c` only | Snapshot 10/10; later aggregate about 20/25, not manifest-audited |
| `card_legacy_plus_fixed_alpha05_loss_seed42` | `fb11e49b` | Legacy + alpha=.5 only | Training verified; robot outcome TBD |
| `card_legacy_plus_fixed_alpha025_loss_seed42` | `3e8f5d72` | Legacy + alpha=.25 only | Training verified; robot outcome TBD |
| `card_legacy_plus_tactile_only_mask_seed42` | `71d0531b` | Mask only tactile/contact features | 13/20 operator-held aggregate |
| `card_legacy_plus_bounded_gate_seed42` | `f9a6d974` | Bounded gate only | 7/20 operator-held aggregate |
| `card_vpce_dp_clean_alpha1_mask_tactile_only_seed42` | `99e254fb` | Clean full, tactile-only mask | No final outcome manifest |
| `card_vpce_dp_clean_alpha1_mask_whole_lowdim_seed42` | `cad62d40` | Clean full, whole-lowdim mask | No final outcome manifest |
| `card_vpce_dp_clean_alpha1_no_gating_tactile_only_seed42` | `a882c57d` | Clean no-gating | No final outcome manifest |
| `card_vpce_dp_clean_alpha1_no_horizon_tactile_only_seed42` | `f1b5c2a8` | Clean no-horizon ablation | No final outcome manifest |
| `card_vpce_dp_clean_alpha1_no_lossweight_tactile_only_seed42` | `575eda44` | Clean no-loss-weight ablation | No final outcome manifest |
| `card_vpce_dp_clean_alpha1_no_masking_tactile_only_seed42` | `63bdcf1a` | Clean no-masking ablation | No final outcome manifest |
| `flower_vpce_dp_clean_alpha1_full_tactile_only_seed42` | `6d1daa5e` | Clean full INSERT model | Training verified; robot outcome TBD |
| `board_vpce_dp_clean_alpha1_full_tactile_only_seed42` | `c3f7dd67` | Clean full WIPE model | Training verified; robot outcome TBD |

The folders `card_a1_full_tactile_seed42_4090ctrl` and the similarly named
clean checkpoints are not interchangeable with the Legacy single-component
checkpoints. Resolve the active Hydra configuration and record its SHA before
testing.

## Phase/progress and expanded-rollout status

Early hand-stabilized pilots were Phase-DP 2/5 and Progress-DP 3/5. These are
historical pilots only. Later Phase/Progress reruns generated trajectories but
the canonical audit found no per-trial success manifest. The latest rebuttal
contains author-reported aggregates Phase-DP `12/20`, Progress-DP `14/20`, and
VPCE-DP `28/30` on SWIPE, plus expanded VPCE-DP results `22/30` on WIPE and
`18/30` on INSERT. These values may be used as the authors' latest manually
labelled summaries, but the 4090 agent must label them `author-reported; raw
manifest audit pending` and must not regenerate, pool, or alter them from CSV
counts. Before camera-ready use, verify trial IDs, checkpoint identity, setup
mode, and that Phase/Progress used corrected elapsed-control-time conditioning;
the older runner advanced a private clock by eight steps per inference and
could saturate ahead of physical control time.

## Current policy wording for the rebuttal

- **Q1:** state the offline AUC/F1 comparison. Say phase/progress are
  time-indexed while VPCE is state-conditioned. If the latest real-robot
  aggregates are retained, identify them internally as author-reported pending
  raw-manifest audit; do not infer them from file counts.
- **Q2:** acknowledge learned-weight collapse and replace it with the fixed
  schedule. Do not quote the early Alpha1 `10/10` snapshot alone after the same
  checkpoint was extended; use the submitted no-weighting `8/10` to show that
  the main contact-anticipation conclusion does not rely on this component.
- **Q3:** the original whole-lowdim mask could remove proprioception and gripper
  width. The corrected semantic design must preserve them and mask only
  tactile/contact features. `13/20` shows viability, not superiority over
  Legacy Full.
- **Q4:** horizon conditioning and tactile-only masking are the main mechanisms;
  gating is auxiliary. Move the ablation table to the paper main text.
- **Q5:** state that operator-held props can ease alignment. The next formal
  comparison must use a rigid or force-limited fixture; do not claim
  fixed-environment robustness from hand-held trials.

## Required next experiment: fixed-mount core protocol

Run the following in randomized blocks, with the reader rigidly mounted or in a
documented force-limited fixture. No person may touch card or reader after a
rollout begins.

1. Legacy Full, N=20 valid trials.
2. Legacy + fixed Alpha1, N=20 valid trials.
3. DP-VT, N=20 valid trials.
4. Only after validation: corrected tactile-only-mask or clean full, N=20.

Freeze the initial pose, card, reader mount, gripper target, controller rate,
workspace limits, inference settings, 60-second timeout, and success rule before
the first counted trial. Report `success/N`, Wilson 95% intervals, and raw trial
manifest paths. Do not tune alpha or gate settings based on these counted
outcomes.

## Evidence locations

- Canonical remote audit: `~/fyt/workspace/Prometheus/experiments/VPCE_REAL_ROBOT_RESULTS.md`
- Early aggregate: `~/fyt/workspace/Prometheus/experiments/vpce_rebuttal_20260809_aggregate.csv`
- Early trial log: `~/fyt/workspace/Prometheus/experiments/vpce_rebuttal_20260809_trial_log.csv`
- Checkpoint manifests: `~/fyt/workspace/VPCE/re/rebuttal_corrected_checkpoints_20260809/*/transfer_manifest.json`
- This handoff: `~/fyt/workspace/VPCE/re/REAL_ROBOT_REBUTTAL_FACTS_20260812.md`
