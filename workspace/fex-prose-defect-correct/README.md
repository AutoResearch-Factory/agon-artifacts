# Pilot: Defect-correction depth reduction (1D Poisson)

**Question.** For `-u''(x)=f(x)` on `[0,1]`, `u(0)=u(1)=0`: given a cheap coarse
solution `u0`, does the correction `delta_u = u* - u0` have *lower symbolic
complexity* (smaller expression-tree depth) than `u*`? If so, a symbolic search
(FEX) could target `delta_u` with a smaller tree.

**Method.** No full FEX run. Fixed expression *templates* of increasing depth
(0=const, 1=single term, 2=two terms), coefficients fit by multi-start
least-squares on 200 collocation points. Record the smallest depth at which each
target reaches MSE < 1e-6. Coarse solutions: linear FEM with 3 and 5 elements,
and a truncated Fourier (lowest sine mode only).

Run: `timeout 300 .venv/bin/python pilot_defect.py`

## Result (headline)

The answer is **conditional on the coarse-solution type**, not universal.

| coarse method | u* depth | delta_u depth | verdict |
|---|---|---|---|
| **Fourier 1-mode** (A) | 1 | **0** | BETTER |
| **Fourier 1-mode** (B) | 2 | **1** | BETTER |
| Fourier 1-mode (C) | 2 | 2 | TIE |
| FEM 3/5-elem (all cases) | 1–2 | **none** | WORSE |

- **Mode-removing coarse solutions help.** Subtracting a smooth Fourier mode
  removes whole terms, so `delta_u` is strictly simpler (B: two modes -> one;
  A: one mode -> exactly zero). This is the regime the idea wants.
- **FEM coarse solutions hurt.** A P1 FEM solution is *piecewise linear*, so
  `delta_u = smooth - piecewise-linear` has kinks at element nodes. The smooth
  templates cannot represent those kinks, leaving a residual floor (MSE ~1e-3 to
  1e-4) that never drops below 1e-6 at any depth tested. Finer mesh (5 vs 3
  elem) lowers the floor but does not cross the threshold.
- **Mixed case C** (sine + polynomial): Fourier removes only the sine part, the
  polynomial `x(1-x)` tail remains, so `delta_u` still needs depth 2 (tie). The
  best single-sin fit bottoms out at 2.06e-6 > 1e-6 — verified stable, not a
  fitting artifact.

## Takeaway for the research idea

The defect-correction-simplifies-the-target hypothesis is **real but not
automatic**. It holds when `u0` removes *symbolic structure that the search
would otherwise have to rediscover* (smooth spectral modes), and fails when `u0`
is in a different function class than the target (piecewise-linear FEM vs smooth
sines) — there it *adds* non-smooth structure the symbolic search cannot
express. Implication: to exploit defect correction in FEX, the coarse solver
should live in (or project onto) the same smooth basis family as the search
grammar, e.g. a low-order *spectral* coarse solve, not a low-order FEM solve.
