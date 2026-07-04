# FEX Active Collocation — Pilot

**Date:** 2026-06-16
**Hardware:** 1x NVIDIA RTX 4060 Ti (16 GB), CUDA 12.4, PyTorch 2.6.0+cu124
**PDE:** Poisson d=10, `-Δu = d` on `[-1,1]^10`, true solution `u = 0.5·Σ xᵢ²`.

## v2 refinement handoff

The refined idea should not claim generic acceleration from active collocation. The clean version is conditional: query-by-committee sampling may help when reward estimation is the bottleneck, but this pilot says smooth Poisson `d=10` is dominated by discrete structure discovery and is hurt by rapidly changing hard sets. Data and artifact handoff lives in `data/MANIFEST.md`.

## Verdict: **NEGATIVE**

Active collocation does **not** reach score > 0.99 in fewer iterations than standard
FEX. When the adaptive resampling is given a refresh interval short enough to engage
during the search (refresh=5), it crosses the 0.99 threshold **later** (step 23 vs 18,
i.e. ~28% *more* iterations). With the spec's refresh=50, it is bit-identical to
standard because the search converges (step 18) before the first refresh ever fires.

| run                  | refresh | cross score>0.99 | final best error | wall time |
|----------------------|---------|------------------|------------------|-----------|
| standard             | —       | **step 18**      | 5.12e-06         | 648 s     |
| active               | 50      | step 18 (tie\*)  | 9.99e-05         | 1036 s    |
| active               | 5       | **step 23**      | 2.13e-06         | 608 s     |

\* identical to standard: the first hard-set refresh at step 50 happens *after* the
crossing at step 18, so the collocation pool is never changed before convergence.

Success metric was: active reaches score > 0.99 in >30% fewer iterations → POSITIVE.
Observed: refresh=50 → 0% change; refresh=5 → +28% (slower). **Signal is negative.**

![comparison](results/comparison.png)

## What "score" means here

The FEX repo logs `error = MSE(LHS_pde, RHS_pde) + 100·MSE(boundary)` and a reward
`score = 1/(1+sqrt(error))`. So "score > 0.99" ⇔ `error < 1.03e-4`. We log both.

## What active collocation does (implementation)

`pilot_fex.py` reuses the FEX Poisson machinery (`Controller`,
`learnable_compuatation_tree`, `LHS_pde`/`RHS_pde`/`true_solution`) but reimplements
the RL search loop so we can (a) swap the interior collocation sampler and (b) log a
clean score-vs-iteration trajectory. (We deliberately skip the original 20000-iter
finetune and 1000×100k relative-L2 tail in `controller_poisson.py`, which run *after*
the search and are irrelevant to a "score vs search-iteration" comparison.)

- **standard**: the `domainbs=5000` interior collocation points are resampled
  uniformly at random every RL step (this matches stock `controller_poisson.py`,
  which draws a fresh `torch.rand(domainbs, dim)` in `get_reward`).
- **active**: every step, 80% (4000) of the interior pool is fresh uniform; the other
  20% (1000) are drawn from a **hard set**. The hard set is recomputed every `refresh`
  steps: take the top-5 buffer candidates, briefly re-fit each one's constants (the
  buffer only stores the discrete action), evaluate per-point |PDE residual| on a dense
  20k grid, normalize each candidate's field by its mean, then pick the grid points
  with **maximum variance of residuals across candidates** (max disagreement). 80%
  uniform is kept to avoid overfitting to the hard set.

Boundary points (`bdbs=1000`) are resampled every inner optimization step in both
modes, exactly as in the original.

## Why it fails on this problem (interpretation)

The bottleneck for Poisson d=10 in FEX is the **discrete structure search** (the
controller discovering the operator sequence for `0.5·Σ xᵢ²`), not collocation-point
coverage. The trajectory shows the controller sitting at error ≈ 0.5 for ~14 steps,
then a single-step jump to ≈1e-4 once it samples the right structure.

- The true solution is globally smooth and simple, so uniform Monte-Carlo collocation
  already estimates the loss well — there is no sharp/localized feature for adaptive
  points to capture (unlike, e.g., a boundary layer or shock).
- Worse, recomputing the hard set every few steps makes the **reward signal
  non-stationary**: the same candidate gets a different error depending on which hard
  points are currently in the pool. This added noise *delayed* the controller's
  breakthrough (step 23 vs 18). The divergence is visible from step 5: at step 9 the
  active-r5 batch-min error is 21.5 vs standard's 9.85 on the same seed.

In short: residual-disagreement sampling is the wrong lever for a problem whose
difficulty is combinatorial (which symbols) rather than numerical (where to sample).

## Caveats / honesty

- **Single seed (0).** The crossing step is somewhat fragile because the final
  precision sits right at the 0.99 floor (error 9.99e-5 ≈ threshold 1.03e-4). The
  *direction* of the result (active not faster) is, however, consistent and mechanistic
  (non-stationary reward), not a precision artifact: with refresh=50 it is provably
  identical to standard, and with refresh=5 it is clearly later in both the score and
  error curves.
- **Reduced iteration budget.** Ran 100 RL steps, not the 500 in the brief. The
  crossing occurs at step ≤23 in every run and the curves are flat thereafter, so 100
  steps fully captures the metric while fitting `timeout 1200` (each run 600–1000 s
  under heavy shared-GPU contention from sibling experiments).
- **No finetune/relative-L2.** The pilot measures the *search* trajectory only.
- The active variant did reach a slightly lower error *floor* (2.1e-6) once converged,
  but this is post-crossing and within run-to-run noise; it is not the pilot metric.

## v3 reward-geometry re-grounding

**Date:** 2026-06-17

The v2 review asked for the missing mechanism check: fixed-candidate reward variance,
controller entropy, and overlap between QBC-selected and residual-selected points. I
added `diagnose_reward_geometry.py`, which runs a short Poisson controller warmup on
GPU, keeps the best buffer actions, then re-evaluates the same discrete candidates
over uniform pools and 80/20 QBC-mixed pools.

This is a diagnostic probe, not a full replacement for the earlier 100-step search
pilot. It uses smaller pools and fewer constant-optimization steps so that it can be
run inside the idea-refinement budget. The stronger probe used `warmup_steps=24`,
`bs=8`, `domainbs=3000`, `bdbs=600`, `grid=8000`, and 4 evaluation pools.

| run | candidate errors | QBC/residual overlap | score corr | mean reward-error CV uniform -> QBC | rank stability uniform -> QBC |
|-----|------------------|----------------------|------------|-------------------------------------|-------------------------------|
| quick | 28.02, 30.15, 33.30, 34.96 | 0.075 | -0.304 | 0.151 -> 0.280 | 0.867 -> 0.700 |
| stronger | 2.57, 2.72, 4.56, 6.46 | 0.712 | 0.247 | 0.535 -> 1.032 | 0.333 -> 0.167 |

Interpretation: QBC-mixed pools are not just a residual proxy in the quick probe
(low overlap), but in the stronger probe they partly collapse toward residual-heavy
regions (71.2% overlap). In both cases, QBC increases fixed-candidate reward
coefficient of variation by about 1.8-1.9x and lowers rank stability. That supports
the v2 hypothesis that smooth Poisson is structure-search dominated and that changing
the hard set can make the reward less comparable across steps.

Controller entropy stayed near 1.0 during these short probes, so the diagnostic does
not show early policy collapse. The failure mode is reward non-stationarity and weak
candidate ranking, not premature controller determinism.

## Files

- `pilot_fex.py` — self-contained driver (modes: standard / active), writes JSON
  trajectory incrementally (survives timeout).
- `diagnose_reward_geometry.py` — fixed-candidate reward variance and
  QBC-vs-residual overlap probe for the v3 refinement.
- `compare.py` — plots score- and error-vs-iteration, prints crossing table.
- `results/standard_seed0.json`, `results/active_seed0.json` (refresh=50),
  `results/active_refresh5_seed0.json` — trajectories + metadata.
- `results/reward_geometry_poisson_seed0.json` and
  `results/reward_geometry_poisson_seed0_stronger.json` — v3 diagnostic probes.
- `results/*.log` — full run logs.
- `results/comparison.png` — the figure above.
- `fex-code/` — upstream FEX clone. Only change: commented out the `torchvision`
  import in `fex/Poisson/utils/__init__.py` (image utils, unused for PDEs).

## Reproduce

```bash
cd workspace/fex-active-collocation
timeout 1200 env PYTHONUNBUFFERED=1 .venv/bin/python pilot_fex.py \
    --mode standard --seed 0 --epoch 100 --bs 8 --dim 10 --out results/standard_seed0.json
timeout 1200 env PYTHONUNBUFFERED=1 .venv/bin/python pilot_fex.py \
    --mode active --seed 0 --epoch 100 --bs 8 --dim 10 --refresh 5 \
    --out results/active_refresh5_seed0.json
timeout 1200 env PYTHONUNBUFFERED=1 .venv/bin/python diagnose_reward_geometry.py \
    --warmup_steps 24 --bs 8 --domainbs 3000 --bdbs 600 --grid 8000 \
    --eval_pools 4 --fit_adam 12 --fit_lbfgs 12 --field_fit_adam 18 \
    --out results/reward_geometry_poisson_seed0_stronger.json
timeout 1200 .venv/bin/python compare.py
```

## If pursued further (not recommended for Poisson)

Active collocation would only plausibly help on PDEs with **localized features** the
uniform sampler under-resolves (boundary layers, shocks, steep gradients — e.g. the
Conservation-law example, or anisotropic / singular RHS). It should also use a **slowly
updated / EMA** hard set rather than a hard swap every few steps to avoid the
non-stationary-reward problem seen here. On smooth high-d problems like this Poisson
case, it is strictly a cost with no benefit.
