# FEX Synchro-Prior Pilot Notes

## 2026-06-17 v5 leakage and controller-proxy update

**Status:** complete
**Pilot signal:** POSITIVE_BOUNDED.

The v5 run keeps the v4 frequency-compiler proxy and adds two reviewer-driven
checks:

- a risk-seeking discrete frequency-action proxy, so the pilot no longer only
  reports deterministic dictionary enumeration; and
- an RHS decoy negative control, because a manufactured RHS can expose the answer
  if we blindly take FFT peaks as the prior.

Key numbers from `results/pilot_summary.json`:

- CuPy ran on the local RTX 4060 Ti; wall time was `18.88s`.
- In the policy-gradient frequency proxy, uniform sampling hit `(4,17)` in
  `23/64` runs with median first hit at `128` evaluations. RHS-FFT soft prior
  and oracle soft prior both hit in `64/64` runs with median first hit `7` and
  `8.5` evaluations. A wrong soft prior shifted by +3 still hit `57/64`, but the
  median first hit worsened to `106`, showing that a broad but wrong prior can
  degrade search.
- The decoy RHS control deliberately adds a strong unrelated `(11,3)` forcing
  peak. Naive RHS-FFT top-1 selects `(11,3)`, misses the true `(4,17)` frequency,
  and gives relative L2 `1.0`. This accepts the reviewer's leakage concern:
  RHS FFT cannot be the whole estimator story.
- Uniform-grid proxy timings remain positive in this run: FFT top-k gives
  `56.4x` and `92.9x` net proxy speedup on single/two-mode cases; NCPSD gives
  `7.8x` and `2.69x`. The non-uniform spectral scan still recovers the mode
  but is slower than dense sampled dictionary in this implementation
  (`0.32x`), so irregular-sample estimation is an engineering risk.
- A small real-FEX-family smoke check was run with the instrumented upstream
  Poisson runner from `workspace/fex-failure-taxonomy` (not Multi-Scale FEX).
  On `helmholtz_sine`, standard controller with seed 0, 20 epochs, batch 6 and
  200 finetune steps failed at relative L2 `5.21e-2`, while the analytic sine
  skeleton oracle succeeded at relative L2 `5.01e-8`.

Honest reading: v5 strengthens the case that a soft frequency prior can shorten
controller-side sampling, and it grounds the search-bottleneck story in one real
FEX-family smoke run. It also finds a concrete failure mode for naive RHS FFT.
The decisive Gate 0 is still true Multi-Scale FEX oracle-soft versus standard
controller on the wide-frequency benchmark.

## 2026-06-17 v4 proxy accounting update

**Status:** complete
**Pilot signal:** POSITIVE, still bounded to a frequency-compiler proxy.

The v4 run keeps the v3 data and adds two reviewer-requested measurements:
soft-prior width degeneration and estimator-overhead accounting. It still does
not run the real Multi-Scale FEX controller.

New measurements from `results/pilot_summary.json`:

- CuPy ran on the local RTX 4060 Ti; wall time was `4.99s`.
- FFT top-k plus estimator overhead is still cheaper than dense dictionary
  fitting in this proxy: `3.98x` net speedup for single mode and `10.67x` for
  two modes. NCPSD is wider but still positive: `5.20x` single mode and `2.45x`
  two modes.
- On non-uniform samples, dense sampled dictionary fitting costs `3.10s`; the
  RHS spectral scan top-1 path costs `0.061s` including estimator overhead
  (`50.8x` proxy speedup).
- Soft neighborhoods do become dense: radius 12 already covers `320/576`
  candidates for the single-mode case and `395/576` for the two-mode case.
  This supports an explicit prior-width cap in the next idea version.
- Error sweep: a +1 frequency error is fixed by soft radius 1; +2 needs radius
  2; +3 is not fixed by radius 2. This turns soft prior width into a measured
  robustness/budget tradeoff rather than an unbounded fallback.

Honest reading: the new pilot closes the cheap evidence gaps about overhead and
soft-prior degeneration inside the proxy. The decisive gate is unchanged:
oracle-soft versus standard Multi-Scale FEX on the real controller.

## 2026-06-17 v3 proxy suite

**Status:** complete
**Pilot signal:** POSITIVE, but still a proxy rather than a true FEX controller run.

The v3 run keeps the same question as v2: can an observable spectral signal
compile away a hard frequency choice before FEX searches expression structure?
It adds the reviewer-requested stress checks that are cheap enough to run before
the real Multi-Scale FEX integration:

- single wide-separated mode `(4,17)`;
- two wide-separated modes `(4,17)` and `(9,22)`;
- RHS-FFT top-k versus NCPSD 95% bandwidth prior;
- non-uniform RHS spectral scan on 4096 random samples;
- hard frequency-error sweep with soft radius-1/radius-2 recovery checks;
- 16 seed random no-prior controls.

Key numbers from `results/pilot_summary.json`:

- Single mode: RHS-FFT uses 1 candidate with relative L2 `0.0`; NCPSD uses
  68/576 candidates with relative L2 `9.97e-15`; random no-prior median
  relative L2 is `1.0`.
- Two modes: RHS-FFT uses 2 candidates with relative L2 `7.97e-17`; NCPSD uses
  198/576 candidates with relative L2 `9.42e-15`; random no-prior median
  relative L2 is `1.0` and 0/16 seeds hit both modes.
- Non-uniform samples: RHS spectral scan top-1 recovers `(4,17)` with relative
  L2 `3.47e-16`; random no-prior median relative L2 is `1.015`.
- A one-bin hard frequency error gives relative L2 `1.0`; soft radius-1
  recovers the true candidate with relative L2 `1.68e-15`.

Honest reading: this supports observable frequency-candidate compilation and
the need for soft/adaptive prior injection. It still does not prove that the
real FEX controller improves. The next gate is a true Multi-Scale FEX
oracle-frequency ablation.

**Date:** 2026-06-16
**Status:** complete
**Pilot signal:** POSITIVE, but only for the frequency-search premise.

## Question

The reviewer identified the cheapest falsifier: before building a full
synchrosqueezed FEX, test whether externally supplied frequency information
actually removes a hard part of the FEX search. This pilot isolates that axis.

## Setup

I used a manufactured 2D oscillatory PDE proxy:

`u(x,y)=sin(2*pi*4*x) sin(2*pi*17*y)`, with RHS
`-Delta u=(2*pi)^2(4^2+17^2)u`.

The candidate dictionary is all sine-product pairs `(m,n)` with
`1 <= m,n <= 24`, so the no-prior search has 576 frequency pairs. For a fixed
pair, the amplitude is solved by least squares against the PDE RHS. Frequency
estimation uses FFT on the RHS only, not samples of the exact solution. The
exact solution is used only to score relative L2 error after a candidate is fit.

This is not a full FEX run. It is a controlled proxy for the spectral
combinatorial-search part of Multi-Scale FEX.

## Results

See `results/pilot_summary.json` for the machine-readable record.

Measured pattern:

- Oracle frozen prior: 1 candidate evaluation, relative L2 `0.0`.
- RHS-FFT estimated frozen prior: recovered `(4,17)` from RHS samples only, 1
  candidate evaluation, relative L2 `0.0`.
- Dense grid: relative L2 `0.0`, but 576 candidate evaluations.
- Random no-prior search: 64 candidate evaluations per seed, hit the exact pair
  in 1/16 seeds, median relative L2 `1.0`, mean relative L2 `0.9375`.
- Perturbed frozen prior `(5,17)`: relative L2 `1.0`, which argues for a
  soft/adaptive prior in the revised method rather than a rigid hard freeze.
- Soft neighborhood prior around the estimate: 9 evaluations, relative L2 `0.0`.

## Honest Reading

The positive signal supports the narrow claim that an observable spectral prior
can compile away a wide-separated frequency choice in a FEX-like dictionary.
It does not yet prove that real FEX controller dynamics improve, that CWT/SST is
better than FFT/NCPSD, or that the method scales beyond low-dimensional separable
oscillations.

The revised idea should therefore make the first full experiment an
oracle-frequency FEX ablation, followed by RHS/residual-estimated priors and a
soft prior variant. It should not claim a broad PDE benchmark contribution.
