# FEX Curriculum-Depth Pilot

**Date:** 2026-06-16
**Hardware:** single RTX 4060 Ti (16 GB), shared with several sibling FEX jobs.
**Status:** complete. **Pilot signal: POSITIVE on the literal metric, but with a
large caveat about *why* — see "Honest reading" below.**

## Idea under test

FEX (Finite Expression Method, Liang & Yang 2022, arXiv:2206.10121) uses an RL
controller to search for a symbolic PDE-solution expression laid out in a
fixed-depth binary operator tree. The proposed twist: **start with a depth-1
tree, find the best shallow expression, lock it as a sub-tree, then grow the
tree one level and search only the new nodes** — a depth curriculum. Hypothesis:
the curriculum reaches a comparable solution with fewer controller iterations
than searching the full deep tree from scratch.

Success metric defined by the task: if the curriculum (60% of the standard
iteration budget) reaches a score >= 90% of the standard run's score, the signal
is POSITIVE.

## Target problem

Poisson equation from the repo's `fex/Poisson/function.py`, dimension **d=10**:

- PDE: `-Laplacian u = -d` on `[-1,1]^d`, Dirichlet BC from the true solution.
- True solution: `u*(x) = 0.5 * sum_i x_i^2`.

**Score** = the FEX reward of the best expression found during the search,
`score = 1 / (1 + sqrt(best_error))` in `(0,1]`, where `best_error` is the
in-loop regression error (PDE residual MSE + 100 * boundary MSE). This is the
same monotone transform the original code uses to turn error into reward, so the
"90%" threshold is well defined. (We do **not** run the original code's 20000-step
fine-tune + relative-L2 tail; it is far too slow to repeat across seeds and is
not needed for a go/no-go comparison.)

## Method / implementation

`curriculum_fex.py` is a self-contained driver that **reuses the repo's operator
set and PDE** (`function.py`) and tree topologies (copied verbatim from
`controller_poisson.py`), and re-implements the controller + search + reward loop
faithfully (same 2-layer MLP controller, same softmax-temperature/tanh,
percentile policy-gradient, per-candidate Adam-then-LBFGS inner fit of the tree's
learnable `a,b` and per-leaf `Linear(d,1)`).

Two arms, **identical hyperparameters** (only tree topology and iteration budget
differ):

- **standard**: depth-3 tree, searched from scratch.
- **curriculum**: 3 phases, depth1 -> depth2 -> depth3.

**Locking** (the novel bit). The inorder node indexing was verified empirically:
- depth1 = 3 nodes `0:U* 1:B 2:U*`
- depth2 = 9 nodes; nodes `0,1,2` are the same `U*,B,U*` block as depth1.
- depth3 = 21 nodes; nodes `0..8` are structurally identical to the whole depth2
  tree (verified), node 9 wraps them, node 10 is the new root, 11..20 are the new
  right half.

So phase 2 locks `{0,1,2}` to the depth1 best actions and searches nodes 3..8;
phase 3 locks `{0..8}` to the depth2 best actions and searches 9..20. Locking
fixes only the *structural action* (those nodes are not sampled and get no policy
gradient); the numeric `a,b` and `Linear` weights are still re-fit from scratch
in every reward evaluation, so the locked sub-expression is re-optimised in the
context of the larger tree.

Results are checkpointed to JSON after every controller step (`flush()` with
atomic rename), so a run killed by the timeout still leaves recoverable partial
data.

## Iteration budget (and why it is small)

The team-lead's spec was 1000 (standard) vs 200+200+200=600 (curriculum). At
paper-faithful settings a single depth-3 controller step costs ~7-20 s on this
GPU (double-autograd Laplacian over the batch, `bs=10` inner Adam+LBFGS fits),
and the card is shared with several sibling FEX jobs. 1000 depth-3 steps would be
multiple hours and blow past the `timeout 1200` (20 min) per-run constraint.

We therefore scaled the budget down while **preserving the 0.6 ratio and the
equal-3-phase structure** that actually test the hypothesis, and ran **4 seeds**
to offset the noise of the smaller budget:

- standard: **40** depth-3 iters.
- curriculum: **24** total = 8 depth1 + 8 depth2 + 8 depth3 (= 60%).

Shared hyperparameters: `bs=10, lr=0.002, domainbs=2000, bdbs=500,
inner_adam=8, inner_lbfgs=8, base=200000, [-1,1]^10`.

## Results (4 seeds)

| seed | standard score (err) | curriculum score (err) | ratio curr/std |
|------|----------------------|------------------------|----------------|
| 0 | 0.186 (19.05)  | 0.889 (0.0157) | 4.77 |
| 1 | 0.122 (52.24)  | 0.793 (0.0685) | 6.52 |
| 2 | 0.121 (53.17)  | 0.941 (0.00393)| 7.80 |
| 3 | 0.106 (71.08)  | 0.818 (0.0498) | 7.71 |

**Aggregate:** standard mean score 0.134 (err 48.9); curriculum mean score 0.860
(err 0.034). **Ratio of mean scores = 6.4x** (643%), versus the 0.9 (90%)
threshold. Curriculum also wins on wall-clock (~120-245 s vs ~340-466 s).

`PILOT SIGNAL: POSITIVE` by a wide margin. See `results/trajectory.png` for the
cumulative-best curves and `results/summary.json` for machine-readable numbers.

## Honest reading (important — do not over-claim "lock-and-grow works")

The per-phase breakdown changes the interpretation:

```
              within-phase best error
seed   depth1     depth2     depth3
 0     0.0157     29.71      88.0
 1     0.0685      1.33      83.5
 2     0.0039      2.12      77.6
 3     0.0498     10.23      69.1
```

In **every seed the depth-1 phase already produces the best result**, and the
later grow-phases are *worse*, not better. The curriculum's headline score is
entirely the depth-1 solution, carried forward as the running minimum. The plot
shows this clearly: the curriculum plateaus before the first phase boundary and
the depth2/depth3 phases are flat.

Why: `u* = 0.5*sum x_i^2` is **already representable at depth-1**, because each
depth-1 leaf is a full `Linear(d,1)` followed by a unary op, so one leaf can be
`x^2(W.x)` and the binary root sums it with a ~0 second leaf. The depth-1 best
expressions found bear this out — all contain an `x^2` leaf:
`[x^2, -, sin]`, `[x^3, +, x^2]`, `[sin, +, x^2]`, `[x^2, +, x]`. So depth-1's
tiny action space (3 nodes) converges in a handful of iterations, while
depth-3's 21-node action space cannot find the same thing in only 40 iters.

So what the pilot actually demonstrates is the **weaker** claim "try the shallow
hypothesis class first" — which trivially wins here because the target needs no
depth. It does **not** demonstrate the interesting claim that *locking a good
sub-expression and growing* finds solutions a flat deep search would miss; on
this problem growth had nothing to add and only hurt.

Caveat on the baseline: standard depth-3 at the paper's ~1000+ iters would very
likely also find this quadratic; the dramatic 6.4x gap is partly an artifact of
the small contention-limited budget. The fair takeaway is about *sample
efficiency at small budgets*, not absolute capability.

## Verdict and recommended next step

- **Literal metric: POSITIVE (6.4x, >> 90%).** The mechanism behind it — search
  the simplest tree first — is real and cheap, and worth keeping.
- **But the depth-curriculum's distinctive "lock-and-grow" step is unproven**
  here and should be tested on a problem where depth-1 is provably insufficient,
  e.g. a Poisson/Schrodinger target whose solution needs a genuine depth-2/3
  composition (nested transcendental, product of factors, etc.). The right
  experiment: pick a target where flat depth-1 plateaus at high error, and ask
  whether locking the best depth-1 block and growing beats both (a) flat depth-3
  from scratch and (b) depth-1 alone. Also re-run with a larger, equal-wall-time
  budget so the baseline is not starved.

## v2 mechanism check: radial-sine target

**Date:** 2026-06-16
**Hardware:** single RTX 4060 Ti.
**Status:** complete. **Mechanism signal: WEAK.**

To test the reviewer concern directly, I added a manufactured target

`u*(x) = sin(sum_i x_i^2)`, with analytic RHS
`-Delta u = -2d cos(sum_i x_i^2) + 4 sum_i x_i^2 sin(sum_i x_i^2)`.

For `d=2`, the target has a useful shallow substructure (`x1^2 + x2^2`) and a
deeper final composition (`sin(...)`). This is a better stress test than the
quadratic Poisson case because the best depth-1 approximation to the final
target need not be the sub-expression that should be locked.

Arms, all with `bs=8, domainbs=1000, bdbs=300, inner_adam=6,
inner_lbfgs=6`:

- flat depth-3: 60 controller steps.
- greedy curriculum: 20 + 20 + 20 steps.
- shallow-only depth-1: 20 steps.

Results across seeds 0, 1, 2:

| arm | mean score | mean ratio |
|-----|------------|------------|
| flat depth-3 | 0.251 | 1.00 |
| greedy curriculum | 0.303 | 1.205 vs flat |
| shallow-only depth-1 | 0.298 | curriculum is only 1.016x better |

Per-phase behavior matters. In seed 1 and seed 2, curriculum's final best is
exactly the depth-1 phase result; depth2 and depth3 do not improve it. In seed 0,
depth2 improves slightly, but depth3 degrades. This means hard greedy locking is
not yet evidence for useful sub-expression transfer. It mostly beats flat
depth-3 because the flat controller is starved at 60 steps, not because
grow-and-lock is reliably using the right module.

The revised idea should therefore use this as a falsifier for the original hard
locking story and move to a transfer-tested top-k soft module frontier: keep
several shallow candidates, test their downstream usefulness cheaply, and use a
soft sampling prior rather than permanently freezing one greedy winner.

## Files

- `curriculum_fex.py`  — self-contained driver (controller + search + locking + checkpointing).
- `run_pilot.sh`       — runs both arms x 4 seeds, each wrapped in `timeout 1200`.
- `run_mechanism_pilot.sh` — v2 radial-sine mechanism check, wrapped in `timeout 1200`.
- `analyze.py`         — aggregates JSONs, prints verdict, writes summary.json + trajectory.png.
- `analyze_mechanism.py` — aggregates the v2 mechanism check.
- `results/standard_seed{0..3}.json`, `results/curriculum_seed{0..3}.json` — per-run trajectories.
- `results/summary.json`, `results/trajectory.png` — aggregate.
- `results_v2/` — radial-sine mechanism check outputs.
- `transfer_probe.py`, `run_transfer_probe.sh`, `results_v3/` — v3 transfer-probe rank-correlation check.
- `data/MANIFEST.md` — data and generated-artifact handoff.
- `fex-code/` — upstream clone (gitignored; not committed).

## v3 transfer-probe check

**Date:** 2026-06-17
**Hardware:** single RTX 4060 Ti.
**Status:** complete. **Signal: NEGATIVE for the current tiny RL probe.**

I added a cheap rank-correlation test for the v2 review's most urgent signal:
does a downstream transfer score rank candidate depth-1 modules better than
their shallow reward?

Setup: radial-sine target, `d=2`. The script samples a 16-step depth-1 candidate
pool, keeps the top 8 shallow candidates, and adds the oracle-like depth-1
module `[x^2, +, x^2]` (`actions=[3,0,3]`) as a sanity check. Each candidate is
locked into a depth-2 tree, then evaluated with a 3-step probe and a 12-step
follow-up run under identical random seeds.

Result (`results_v3/transfer_probe_summary_seed0.json`):

- Spearman(shallow score, follow score): `0.667`.
- Spearman(3-step probe score, follow score): `0.101`.
- Best follow candidate: `actions=[5,2,3]`, follow score `0.267`.
- Oracle `x^2 + x^2`: follow score `0.258`, not clearly better than learned candidates.
- Verdict: `NEGATIVE`.

Interpretation: the first downstream probe implementation is too noisy and does
not improve selection over shallow reward. This does not kill the broader idea,
but it changes the next version's method: the probe itself must become the
scientific object and a hard gate. A viable v3 should define a more structured
held-out wrapper/context probe, require oracle-module success before learned
candidate claims, and fall back to a diagnostic result if transfer ranking is no
better than shallow reward or random ranking.
