# Offline Contact-Prior Table Design

## Goal

Add a compact offline comparison to the one-page CoRL rebuttal that directly
addresses whether VPCE is only a phase/progress signal, without conflating
offline contact prediction with real-robot task success.

## Approved content

- Keep the existing SWIPE real-robot success table unchanged.
- Add a three-row table comparing Phase, Progress, and VPCE contact priors.
- Report held-out contact-prediction AUC and F1, not an offline "success rate."
- Use `TBD` for unavailable measurements.
- State that true normalized test-trajectory phase/progress is an oracle offline
  comparator, whereas VPCE is predicted online from RGB and robot motion.
- State that the fixed-clock Phase/Progress policy rollouts and the offline
  oracle diagnostic answer different questions.

## One-page layout

The new table will use `\footnotesize`, three rows, and four columns. The Q1
paragraph will be shortened so that the compiled rebuttal remains exactly one
page. Other rebuttal sections and existing pilot values will not be changed.

## Validation

- Compile `rebuttal_template.tex` without LaTeX errors.
- Confirm the generated PDF has exactly one page.
- Extract PDF text to verify the new table, oracle caveat, and `TBD` values are
  present.
- Inspect the Git diff before committing and pushing to `origin/main`.
