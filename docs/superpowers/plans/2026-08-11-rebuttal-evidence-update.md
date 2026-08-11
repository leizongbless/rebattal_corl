# VPCE CoRL Rebuttal Evidence Update Implementation Plan

> **For AI agents:** Required sub-skill: use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan. Track steps with `- [ ]` checkboxes.

**Goal:** Produce and publish a one-page rebuttal that uses the current Legacy single-component evidence honestly and defines the minimum remaining robot experiments.

**Architecture:** `rebuttal_template.tex` is the submission-facing one-page response. The design document records the evidence boundary and causal interpretation. This plan records the ordered experiment and verification gates so later results can replace TBD entries without changing protocol.

**Technical stack:** LaTeX/IEEEtran, Tectonic, `pdfinfo`, `pdftotext`, Git.

---

### Task 1: Replace the preliminary table with isolated Legacy component evidence

**Files:**
- Modify: `rebuttal_template.tex`

- [ ] **Step 1: Encode only operator-reported aggregates**

  Add rows for Legacy Full N=20 as TBD, fixed Alpha1 loss 10/10,
  tactile-only mask 13/20, and bounded gate 7/20. Label the table
  operator-held and preliminary.

- [ ] **Step 2: Preserve evidence boundaries**

  State that historical N=5 pilots, trajectory-file counts, and unlabelled
  Phase/Progress reruns are excluded from the new table.

- [ ] **Step 3: Verify numerical text**

  Run:

  ```bash
  rg -n '10/10|13/20|7/20|TBD|operator-held' rebuttal_template.tex
  ```

  Expected: all three aggregates, the missing anchor, and the setup caveat are
  present.

### Task 2: Answer the four reviewer concerns without overclaiming

**Files:**
- Modify: `rebuttal_template.tex`

- [ ] **Step 1: Explain the implementation corrections**

  State the collapse mode of the learned weight and the fixed bounded `1+p`
  correction. State that tactile-only routing preserves proprioception and
  gripper width.

- [ ] **Step 2: Interpret bounded-gate evidence narrowly**

  State that 7/20 rejects the tested gate range and is consistent with visual
  over-suppression; do not claim that all gating is unnecessary.

- [ ] **Step 3: Keep Phase/Progress and fixed-mount conclusions pending**

  State that corrected timing and rigid-mount N=20 comparisons are required
  before claiming superiority to a progress prior or resolving human
  compliance.

- [ ] **Step 4: Check claim language**

  Run:

  ```bash
  ! rg -n 'proves|significantly better than Legacy Full|gating is useless' rebuttal_template.tex
  ```

  Expected: exit 0 because none of the prohibited overclaims appears.

### Task 3: Define the remaining experiment gates

**Files:**
- Modify: `docs/superpowers/specs/2026-08-11-rebuttal-evidence-update-design.md`

- [ ] **Step 1: Freeze the P0 anchor comparison**

  Require same-protocol Legacy Full N=20 and extend fixed Alpha1 to N=20.

- [ ] **Step 2: Freeze the fixed-mount core**

  Require DP-VT, Legacy Full, and best Alpha1 at N=20 each, randomized in
  blocks, with success defined as one complete internal roller rotation within
  60 seconds.

- [ ] **Step 3: Gate optional experiments**

  Run the combined Alpha1+tactile-only checkpoint only after the Alpha1 anchor
  holds. Select one weaker gate offline before N=20 robot evaluation. Repair
  Phase/Progress timing before counting their rollouts.

- [ ] **Step 4: Verify the protocol fields**

  Run:

  ```bash
  rg -n 'N=20|fixed-mount|randomized|one-rotation|Phase/Progress' \
    docs/superpowers/specs/2026-08-11-rebuttal-evidence-update-design.md
  ```

  Expected: each protocol and stopping gate is explicit.

### Task 4: Build and verify the one-page artifact

**Files:**
- Modify: `rebuttal_template.pdf` only if the repository tracks the PDF

- [ ] **Step 1: Build from a clean LaTeX invocation**

  Run:

  ```bash
  env -u LD_LIBRARY_PATH /home/hkust/miniconda3/envs/latex_cv/bin/tectonic \
    --keep-logs --keep-intermediates rebuttal_template.tex
  ```

  Expected: exit 0 and an updated `rebuttal_template.pdf`.

- [ ] **Step 2: Verify the page limit and visible text**

  Run:

  ```bash
  pdfinfo rebuttal_template.pdf | rg '^Pages:\s+1$'
  pdftotext rebuttal_template.pdf - | rg '10/10|13/20|7/20|rigidly mounted'
  ```

  Expected: both commands exit 0.

- [ ] **Step 3: Verify the Git patch**

  Run:

  ```bash
  git diff --check
  git status --short
  ```

  Expected: no whitespace errors; only the rebuttal, its PDF, and the two new
  planning documents are changed.

### Task 5: Commit and publish

**Files:**
- Stage only the files listed above

- [ ] **Step 1: Review the staged patch**

  Run:

  ```bash
  git diff --cached --stat
  git diff --cached --check
  ```

  Expected: the intended four files only and no whitespace errors.

- [ ] **Step 2: Commit**

  Run:

  ```bash
  git commit -m "docs: update VPCE rebuttal with component evidence"
  ```

  Expected: one new commit on `main`.

- [ ] **Step 3: Push the complete local main branch**

  Run:

  ```bash
  git push origin main
  ```

  Expected: `origin/main` advances to the new commit, including the three
  pre-existing local documentation commits.
