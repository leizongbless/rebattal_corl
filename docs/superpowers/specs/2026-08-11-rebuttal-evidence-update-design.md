# VPCE CoRL Rebuttal Evidence Update Design

## Objective

Revise the one-page rebuttal around the evidence that is currently auditable,
without presenting hand-held pilots, incomplete artifact directories, or
runtime-confounded Phase/Progress runs as formal comparisons.

## Evidence hierarchy

1. Report the three operator-reported Legacy single-component aggregates as
   preliminary, operator-held results: fixed Alpha1 loss 10/10, tactile-only
   masking 13/20, and bounded gating 7/20.
2. Keep the same-protocol Legacy Full N=20 anchor as TBD. The historical 4/5
   Legacy result remains a separate hand-stabilized pilot and is not pooled.
3. Do not convert trajectory-file counts into outcomes. Do not report the
   Phase/Progress reruns until outcomes and corrected runtime conditioning are
   verified.
4. State qualitative failure evidence only as diagnosis: the current bounded
   gate appears to suppress vision too strongly, while corrected combined Full
   can stall with small, reversing proposals.

## Rebuttal argument

- The learned loss-weight formulation has a real collapse mode. Replacing it
  with fixed bounded `1+p` is an implementation correction, and it is also the
  strongest isolated mechanism in the current robot results.
- The original whole-lowdim mask can remove proprioception. Tactile-only
  masking fixes the semantics and remains usable, but 13/20 is not yet evidence
  that it beats the Legacy Full anchor.
- The 7/20 bounded-gate result rejects the tested gate range, not gating as a
  general idea. The response should narrow the claim and promise a weaker,
  pre-registered gate setting rather than tuning on robot outcomes.
- Human compliance remains a protocol confound. Formal claims require a rigid
  mount and randomized same-controller trials.
- Phase/Progress comparisons remain TBD until their conditioning follows real
  elapsed/control time and their outcomes are manually labelled.

## Next-experiment decision gates

1. Finish same-protocol Legacy Full at N=20 and extend fixed Alpha1 from N=10
   to N=20. This decides whether the isolated loss correction improves on the
   actual anchor.
2. Run a fixed-mount core comparison of DP-VT, Legacy Full, and the best
   Alpha1 variant at N=20 each, randomized in blocks with one-rotation success.
3. Only if fixed Alpha1 remains better than Legacy Full, test one combined
   Legacy + fixed Alpha1 + tactile-only checkpoint, first N=5 sanity and then
   N=20.
4. Select one weaker gate setting offline from prediction/action traces, freeze
   it before robot evaluation, and compare it with Legacy Full at N=20. Do not
   sweep gate parameters on robot success.
5. Repair and verify Phase/Progress timing, then run N=20 fixed-mount only if
   the runtime trace proves that conditioning matches elapsed/control time.

## Prioritized experiment matrix

### P0: minimum causal component comparison

| Group | Additional valid trials | Setup | Purpose |
|---|---:|---|---|
| Legacy Full exact anchor | 20 | Same operator-held protocol as the three isolated variants | Supplies the missing common reference. |
| Legacy + fixed Alpha1 loss | 10 (to 20 total) | Same protocol and controller | Tests the only currently promising isolated correction at matched N. |

Do not combine the historical Legacy Full 4/5 pilot with the new anchor. Stop
and diagnose if checkpoint SHA, initialization, controller rate, workspace
limits, or setup mode differs from the isolated-component runs.

### P0: minimum reviewer-facing fixed-mount comparison

| Group | Valid trials | Setup | Purpose |
|---|---:|---|---|
| DP-VT | 20 | Rigid reader mount | Non-VPCE tactile baseline. |
| Legacy Full | 20 | Rigid reader mount | Original VPCE policy under the corrected protocol. |
| Legacy + fixed Alpha1 loss | 20 | Rigid reader mount | Best isolated correction. |

Randomize the three policies in complete blocks. Freeze checkpoint SHA,
initial pose, gripper target, 60-second timeout, control/inference settings,
workspace limits, card, reader mount, and table height before the first counted
trial. No hand contact with the reader or card is permitted after a trial
starts. An invalid trial is replaced but retained in the manifest.

### P1: conditional mechanism tests

- **Combined policy:** train Legacy + fixed Alpha1 + tactile-only with all other
  fields fixed. Run five non-counted sanity trials; expand to 20 counted trials
  only if runtime/source checks pass and no new systematic failure appears.
- **Weaker gate:** choose one range offline from frozen observation traces and
  record the selection rule before training. Compare it with Legacy Full at
  N=20; do not select among several gate values using robot success.
- **Phase/Progress:** add fake-clock tests that prove monotonically aligned
  conditioning under the measured control schedule. Only then run the same
  fixed-mount N=20 protocol.

### P2: evidence that should be offline first

- On one prospective trajectory-level held-out manifest, compare VPCE,
  phase/progress, current-contact, and time-to-contact predictions using AUC,
  F1, Brier score, ECE, and per-trajectory bootstrap intervals.
- Report label density over the pre-declared contact-threshold sweep. Do not
  call density sensitivity a performance-sensitivity result without retraining
  or reevaluating the predictor.
- Evaluate source-teacher zero-shot ranking before target adaptation and report
  negative results. This can answer the transfer concern without extra robot
  rollouts.
- Use completed Flower/Board checkpoints first for offline load, routing, and
  trace checks; schedule cross-task robot trials only after the Card protocol
  closes.

## Required per-trial record

Every attempted trial must append one immutable row containing:

```text
timestamp | block/order | profile | policy target | checkpoint SHA | seed |
setup mode | initial pose | gripper target | controller settings |
success/failure/invalid | failure type | one-rotation time | hand assistance |
video path | tactile path | proposal-to-sent trace path | notes
```

Count success only when the internal roller completes one effective rotation
within 60 seconds. Preserve invalid and aborted attempts but exclude them from
the success-rate denominator. Report success/N, Wilson intervals, and exact
pairwise tests with multiplicity correction for pre-declared comparisons.

## Deliverables and validation

- Update `rebuttal_template.tex` while preserving the one-page limit.
- Add a reproducible experiment plan with exact priorities and stopping gates.
- Rebuild the PDF, verify one page, inspect extracted text, run `git diff
  --check`, and push only after these checks pass.
