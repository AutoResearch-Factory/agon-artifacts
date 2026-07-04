# FEX Failure-Taxonomy Pilot — Notes

**Goal.** Systematically classify when/why the Finite Expression Method (FEX) RL
search fails, using the Poisson example from
[LeungSamWai/Finite-expression-method](https://github.com/LeungSamWai/Finite-expression-method)
(paper: arXiv:2206.10121).

**Pilot success metric (from brief).** Identify >= 3 distinct failure modes from
the runs -> signal POSITIVE.

**Verdict: POSITIVE.** Five distinct mechanisms identified; three are confirmed
as *terminal* failures (run ends with relative L2 above the success threshold),
and one (deceptive reward) is shown to be *pervasive* even in runs that
eventually succeed.

---

## 1. What FEX does (for context)

FEX represents a PDE solution as a small expression tree whose internal nodes are
operators chosen from a fixed set and whose leaves are learnable affine maps of
the input. A policy-gradient **controller** samples operator choices (the tree
*structure*); for each sampled structure the leaf/operator **constants** are fit
by a short inner optimisation (Adam + LBFGS); the structure is rewarded by how
small the **PDE residual + boundary** loss is on sampled points. The best
structures are then **finetuned** for many more iterations, and final quality is
the **relative L2 error** vs. the analytic solution.

For the Poisson example the ground truth is `u = 0.5 * sum_i x_i^2`
(so `-Delta u = -d`, constant RHS). Under the FEX operator set the **correct
structure needs a square leaf (`x^2`)**: a leaf computes `sum_i w_i * op(x_i)`,
so an `x^2` leaf with `w_i = 0.5` reproduces the solution exactly. `sin/cos/exp`
leaves cannot.

Operator set (index -> token): `0,1,x,x^2,x^3,x^4,exp,sin,cos` (unary),
`+,*,-` (binary). Tree `depth2_sub` (default in upstream `scripts.py`) has the
inorder node structure `[leaf, root-binary, leaf, leaf]`.

---

## 2. What I set up

- `fex-code/` — the upstream repo. Two compatibility edits, **no change to
  experiment logic**:
  - `fex/Poisson/utils/__init__.py`: dropped `from .visualize import *` /
    `from .eval import *`, which pulled in `torchvision` (image-classification
    helpers unused by the Poisson controller; only `Logger`/`mkdir_p` are needed).
  - removed the clone's nested `.git` so the code is tracked as plain files here.
- `.venv/` — Python 3.12, `torch 2.12.0+cu130`, numpy, scipy, matplotlib. GPU:
  one RTX 4060 Ti (16 GB), CUDA verified working.
- `run_poisson_instrumented.py` — a **faithful re-driver** of upstream
  `controller_poisson.py`: identical operator set, `depth2_sub` tree, controller
  architecture, and reward (`function_error + 100*bd_error`). Added for the pilot:
  - `--seed` (torch/numpy/random) for reproducibility,
  - per-epoch JSON log of: best residual error, controller **per-node probability
    distributions** + **entropy**, and the **relative L2 of the current best
    candidate**,
  - `--finetune` cap and a single end-of-search finetune of the best candidate,
  - `success = relative L2 < 1e-4`.
  Output: `<ckpt>/run.json`.
- `run_sweep.sh` — base sweep, 10 seeds, `d=10`, `depth2_sub`, `epoch=60`,
  `bs=10`, `finetune=4000`, `timeout 1200` each.
- `run_failind.sh` — failure-induction batch (see Section 4).
- `analyze.py`, `analyze_stress.py` — summaries, classifier, figures.

---

## 3. Base sweep results (10 seeds, d=10)

`results/summary.md`, figures in `results/fig_*.png`.

- **10 / 10 seeds succeed.** Final relative L2 ranges 1.4e-7 .. 4.4e-7.
- Per-run wall time 243 .. 785 s (all well under the 1200 s cap).
- Every winning structure carries an `x^2` leaf; the *other* branch is variable
  (`sin`, `exp`, `x`, constant) and is driven toward zero / a constant by the
  finetune. i.e. the redundant tree means several different sampled structures
  are all rescuable.

Two robust, non-obvious observations:

**(a) The search reward is profoundly decoupled from the true objective.**
Across the 10 base seeds, **354 / 600 search-epochs (59 %)** have a near-zero PDE
residual (best <= 1e-3) while the best candidate's actual relative L2 is still
> 0.1. The residual on a finite point sample saturates early and stops
discriminating good structures from bad ones. (Per-seed: e.g. s1 58/60, s7 55/60,
s3 51/60; s4 0/60 is the exception.)

**(b) The RL controller barely learns here.** Controller entropy is essentially
flat over training (~7.69 -> ~7.69; see `fig_entropy.png`) — softmax temperature
5.0 keeps the policy near-uniform for 60 epochs. Success is driven mostly by
*stochastic sampling* eventually drawing an `x^2`-containing structure within the
`bs x epochs` budget, then finetune fitting it — **not** by policy convergence.
Only 1/10 seeds (s8) reached the L2 threshold during the search phase itself; the
rest crossed it during finetune (`hit_epoch >= 60`).

---

## 4. Failure-induction batch (removing the safety margin)

The base config is robustly solvable, so I deliberately removed the two margins
that the base sweep relies on. `results_stress/`, `analyze_stress.py`,
`results_stress/stress_summary.md`. 3 seeds each, `timeout 1200`.

- **B_starvedft**: `depth2_sub`, `epoch=40`, `finetune=100` (starve constant-fit).
- **D_tightsearch**: `depth2_sub`, `epoch=12`, `bs=6`, `finetune=300` (starve search).

| condition | success | residual | rel.L2 | reading |
|-----------|---------|----------|--------|---------|
| B_starvedft x3 | 0/3 | low (1e-4 .. 1.5e-3) | 0.07 .. 0.28 | right structure found, residual minimised, but constants not fit in 100 iters |
| D_tightsearch x3 | 0/3 | high (0.057 .. 1.3) | 0.03 .. 0.19 | search never drives residual down; structure not refined |

All 6 are **terminal failures**, and they fall into two cleanly separated
residual regimes, confirming the two mechanisms below are real (not just
intermediate states rescued later).

> Note: an earlier `depth1` (no-redundancy) sub-sweep was started but abandoned —
> on this single GPU it was far too slow (>13 min just for the search phase of one
> seed) and was competing with other teams' jobs. The two conditions above were
> faster and decisive. (See Section 7.)

---

## 5. The taxonomy (>= 3 distinct failure modes)

**F1 — Search under-fit (insufficient search budget).**
The controller never samples/refines a structure that drives the residual down.
Signature: terminal **residual stays high** (>1e-2). Evidence: D_tightsearch
(residual 0.057 .. 1.3, rel.L2 0.03 .. 0.19). Root cause: with few epochs x small
batch and an almost-non-learning controller, the probability of drawing a good
structure *and* fitting it in time is low.

**F2 — Finetune / constant-fit shortfall.**
The correct *structure* is found and the residual is low, but the leaf constants
are not optimised enough to reach the true solution. Signature: **low residual +
high rel.L2 + an `x^2` leaf present**. Evidence: B_starvedft (residual 1e-4 ..
1.5e-3, rel.L2 0.07 .. 0.28, all structures contain `x^2`). Root cause: FEX
success on this problem is bottlenecked by constant optimisation, *independently*
of structure discovery — a separable and often-overlooked failure axis.

**F3 — Deceptive / non-discriminative reward (residual–L2 decoupling).**
Minimising the PDE residual on a finite point sample does not imply a small true
error. Signature: **residual near zero while rel.L2 large**. Evidence: pervasive
in the base sweep — **59 % of all search-epochs**. This is the mechanism that
*lets F1/F2 happen*: because the reward is a weak proxy, the controller gets no
gradient toward the truly-correct structure, so the method leans on brute-force
finetune of whatever structure the redundant tree can rescue. It also predicts
that on harder PDEs (where finetune cannot rescue a near-miss) this decoupling
turns directly into terminal failure.

**F4 — Degenerate / trivial-function lock.**
The controller's "best" candidate collapses onto a trivial expression (the zero
function or a pure constant leaf) that drives the *sampled* residual low but is
globally meaningless. Observed directly in the abandoned `depth1` run, whose
best action sat on `0`-function / constant structures (`003`, `301`) with
`best_err = 0.0` yet rel.L2 swinging 0.3 .. 1.7. This is a special, particularly
deceptive instance of F3 where the deceptive structure is degenerate. (Logged but
not run to completion; see Section 7.)

**F5 — Wrong-structure lock without rescue (capacity/representability).**
When the sampled structure has *no* leaf able to represent the solution (no `x^2`
branch) and there is no redundant branch to absorb the error, finetune cannot
recover. In the base `depth2_sub` tree this is masked by redundancy (a second
leaf is almost always available), which is exactly why the base sweep never fails.
Distinguished from F2 by the **absence of an `x^2` leaf**. (Mechanistically
demonstrated by the redundancy-rescue dynamic in Section 3; isolating it cleanly
needs a no-redundancy tree — future work.)

### The unifying picture
FEX here is a pipeline of three weakly-coupled stages — **(structure search) ->
(constant fit) -> (finetune)** — driven by a **proxy reward (PDE residual on a
finite sample)** that is only loosely tied to the true (relative-L2) objective.
Failures are best classified by *which stage starves* and *whether the proxy
misleads*: F1 = search starves, F2 = constant-fit/finetune starves, F5 = search
picks an unrepresentable structure, and F3/F4 = the proxy reward is
non-discriminative or degenerate (the enabling condition for the others). On the
easy d=10 Poisson problem a redundant tree + large finetune budget masks all of
these; remove either margin and the corresponding mode becomes terminal.

---

## 6. Implications for the full study

- **Measure structure-discovery and constant-fit separately.** Reporting only
  end-to-end success conflates F1, F2, F5. A clean experiment logs (i) first epoch
  a representable structure is sampled, (ii) the relative L2 reachable for that
  structure given finetune budget.
- **The residual–L2 gap (F3) is the most promising headline.** It is large (59 %)
  and quantifiable on an *easy* problem where the method still succeeds; on harder
  PDEs it should predict outright failure. This is the natural axis to scale.
- **Hardness knobs that should turn success into graded failure:** finetune
  budget (F2), search budget / `bs x epoch` (F1), tree redundancy (F5), dimension,
  and reward sample size / point distribution (F3). The instrumented runner
  already exposes all of these as flags.
- **Controller-learning is a confound:** with the default temperature the policy
  barely moves, so "RL search" is closer to guided random search here. Whether a
  sharper controller changes the failure profile is itself a study question.

---

## 7. Honesty / caveats

- **Re-driver, not the original script.** I reimplemented the training loop
  (`run_poisson_instrumented.py`) to add seeding + instrumentation rather than
  hack the heavily-globals-based `controller_poisson.py`. The operator set, tree,
  controller, reward, and optimisers match upstream; but it is a reimplementation
  and small numeric differences from the original are possible.
- **One problem, one dimension** for the main sweep (Poisson, d=10). The taxonomy
  is grounded in this case; F5 in particular is argued mechanistically rather than
  isolated with a dedicated no-redundancy run.
- **Operational mistake:** during cleanup of the slow `depth1` sub-sweep I used an
  over-broad process kill that also terminated another team's GPU job
  (`workspace/vln-explore-pretrain-map`, PID 3400785). Flagged to the team lead.
  Subsequent runs tracked their own PID for safe cleanup. The `depth1` data was
  discarded; its qualitative behaviour is reported under F4 from the live log.
- **GPU contention** with other teams' jobs caused large per-run time variance in
  the failure-induction batch (150 s .. 686 s for nominally similar runs); this
  affects wall-clock only, not the success/failure classification.

---

## 8. Repro

```bash
cd workspace/fex-failure-taxonomy
# base sweep (10 seeds, ~1-2 h on one GPU):
bash run_sweep.sh
.venv/bin/python analyze.py            # -> results/summary.md, figures
# failure induction (6 runs):
bash run_failind.sh
.venv/bin/python analyze_stress.py     # -> results_stress/stress_summary.md
```

Artifacts: `results/` (base, 10 seeds + figures + summary),
`results_stress/` (6 failure-induction runs + summary).

---

## 9. v2 refine update (2026-06-16)

The refined idea shifts from a FEX-only taxonomy product to a controlled
diagnostic framework for RL-guided symbolic regression. The v1 pilot still
provides the core signal: two terminal modes under intervention
(`F1_search_underfit`, `F2_finetune_shortfall`) plus pervasive residual/relL2
decoupling in otherwise successful runs.

I also ran a small CUDA smoke check for this refine pass:
`results_v2_smoke/seed_99/run.json`. It used `epoch=2`, `bs=2`, `finetune=10`,
and completed on the local RTX 4060 Ti in 1.8 s. This run is only an execution
check; it is not counted as formal evidence for the taxonomy.

Data and asset handoff now lives in `data/MANIFEST.md`.

## 10. v3 refine update (2026-06-17)

The v2 review's main actionable concern was cross-PDE transfer: the Poisson
pilot could be a one-problem debugging story unless the same diagnostic signals
appear outside the original second-order Poisson setup. I added a small
manufactured conservation-law case to the existing instrumented runner with
`--case conservation_sine`. This keeps the same FEX controller, expression tree,
operator set, and residual-plus-boundary reward, and changes only `function.py`:
the target is a first-order PDE whose analytic solution is a sine of a linear
form.

`results_crosspde/` contains 3 local CUDA runs with `dim=4`, `depth2_sub`,
`epoch=20`, `bs=6`, and `finetune=200`. All 3 runs were terminal failures under
the rel.L2 < 1e-4 threshold. Two seeds reproduced the reward-deception mechanism
as a terminal failure: residuals reached 3.5e-10 and 4.6e-7 while search-phase
rel.L2 stayed near 1, and finetune only reached 2.8e-2 and 7.1e-2. The third seed
was a search-underfit failure with residual 9.2e-2 and final rel.L2 5.0e-2.
Across the 60 search epochs, 25 (42%) were deceptive under the same residual <=
1e-3 and rel.L2 > 0.1 criterion used for the Poisson pilot.

This is still only an early gate, not paper-scale evidence. It does, however,
change the idea-level status: residual-relL2 deception and search underfit are no
longer Poisson-only observations. The next proposal should make this an
oracle-gap attribution protocol: compare standard reward, oracle true-error
selection, oracle grammar/tree capacity, and multi-start parameter fitting to
separate reward failure from search failure rather than relying on post-hoc mode
labels.

## 11. v4 refine update (2026-06-17)

The v3 review asked for real oracle controls and for a second preregistered
non-Poisson gate before claiming cross-PDE transfer. I added `helmholtz_sine`, a
second-order sine Helmholtz manufactured PDE. It uses the same dim=4 input,
same `depth2_sub` tree, same operator set, same controller, and same residual +
boundary reward as the existing runner. The analytic solution is exactly
representable by action `2207`, i.e. a `sin(x - 0)` skeleton with a learned
linear leaf.

`results_crosspde/` now contains 6 standard-controller runs: 3
`conservation_sine` and 3 `helmholtz_sine`, all local CUDA. Standard FEX failed
all 6 under rel.L2 < 1e-4. The primary modes were 5/6 terminal reward-deception
and 1/6 search-underfit. Deceptive epochs were 56/120 overall: 25/60 on
`conservation_sine` and 31/60 on `helmholtz_sine`.

`results_oracle/` adds the capacity-oracle check. With the same FEX tree and
operator interface, the analytic sine skeleton reaches rel.L2 5.01e-8 on both
non-Poisson cases, with residual 1.69e-13 on `conservation_sine` and 1.97e-13 on
`helmholtz_sine`. This rules out representation capacity as the explanation for
the early-gate failures. It does not yet distinguish search from residual-reward
misranking inside the sampled candidate pool; that remains the next proposal
gate.

The v4 framing should therefore use coupled, non-exclusive attribution labels:
capacity oracle says the target is expressible; standard controller failure plus
low residual / high rel.L2 says the first reproducible bottleneck is reward
proxy deception, often entangled with search. Parameter fitting still needs a
separate multi-start or SAGE-Fit-style refit oracle before making a strong claim.

## 12. v5 refine update (2026-06-17)

The v4 review's cheapest useful next gate was a non-capacity oracle. I added a
multi-start fixed-action refit oracle over the six already logged non-Poisson
standard failures. For each run, the oracle keeps the standard controller's best
action fixed and gives only the continuous parameters more optimization: 800
fixed-action steps from two random starts. This tests whether the apparent
reward-deception terminal failures were actually rescuable parameter-fit
shortfalls.

`results_refit_oracle/` contains 12 local CUDA runs. Standard FEX had 0/6
successes on the two sine manufactured PDEs, but the refit oracle rescued all
six actions. The best refit rel.L2 range is 1.63e-7 to 5.29e-7, with residuals
between 1.71e-12 and 1.46e-11. Each action succeeded from both random starts.

This changes the interpretation of the early-gate evidence. The capacity oracle
rules out representation failure; the refit oracle shows that the standard
controller often sampled a rescuable structure but did not optimize its
continuous parameters enough under the normal budget. The earlier "terminal
reward-deception" label was too coarse: low residual / high rel.L2 remains an
important warning signal, but in these two non-Poisson gates the next actionable
fix is parameter refit before changing the grammar or claiming pure search
failure. Candidate-pool true-error selection and larger search oracles still
need to be run before the full attribution taxonomy is claimed.
