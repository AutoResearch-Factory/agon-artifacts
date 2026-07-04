# Pilot: is FEX coefficient optimization convex once the skeleton is known?

**Idea under test.** FEX solves PDEs by (1) RL search over expression *structure*,
then (2) Adam+BFGS over the *coefficients*. PROSE-PDE showed that knowing only the
skeleton (not the coefficients) recovers almost the full-equation accuracy, hinting
that stage (2) is "easy". This pilot numerically probes whether the coefficient
problem is actually convex.

**Setup.** 1D Poisson `-u''(x) = f(x)` on `[0,1]`, `u(0)=u(1)=0`, true solution
`u*(x) = sin(pi x)` so `f(x) = pi^2 sin(pi x)`. Loss is the squared PDE residual
plus a boundary penalty, integrated with 100-pt Gauss-Legendre quadrature:

```
L(c) = ∫_0^1 ( -u''(x;c) - f(x) )^2 dx  +  ( u(0;c)^2 + u(1;c)^2 )
```

We optimize 5 skeletons (multi-start Nelder-Mead + BFGS polish), compute the
finite-difference Hessian at the optimum, check positive-definiteness, and measure
convexity along 50 random 1D slices at several radii.

## Run

```
.venv/bin/python pilot_convexity.py        # < 5 s
```

## Result

| skeleton | dim | final loss | min eig(H) | PD? | cvx@0.2 | cvx@2.0 |
|----------|-----|-----------|-----------|-----|---------|---------|
| (a) `c1 sin(c2 x)`              | 2 | 1.5e-32 | 27.99    | yes | 100% | 20%  |
| (b) `c1 x(1-x)+c2 x²(1-x)²`     | 2 | 2.9e-02 | 1.60     | yes | 100% | 100% |
| (c) `c1 sin(c2 x)+c3 cos(c4 x)` | 4 | 7.5e-33 | 6.4e-33  | **no (PSD)** | 100% | 44% |
| (d) `c1 exp(c2 x) sin(c3 x)`    | 3 | 1.2e-20 | 5.94     | yes | 100% | 26% |
| (e) `Σ c_k sin(k pi x)`         | 3 | 7.5e-20 | 97.41    | yes | 100% | 100% |

## Takeaways

1. **The optimum is a strictly-convex basin in every case** — `min eig(H) > 0` (or
   `≈ 0` for the over-parameterized (c)), and **100% of random slices are convex at
   small radius**. So *locally*, once you are near the right coefficients, the problem
   behaves like a well-conditioned convex problem. This supports the "coefficient
   fitting is easy near the solution" intuition behind PROSE/FEX stage (2).

2. **The claim "convex" does NOT hold globally for skeletons nonlinear in their
   coefficients.** Skeletons (a), (c), (d) are convex only in a local basin and
   non-convex at radius ~2 (cvx@2.0 = 20–44%). The culprit is the *frequency*-type
   coefficient: the residual contains `c1 c2² sin(c2 x)`, so the loss grows like
   `c2⁴` and oscillates in `c2`, giving a non-convex global landscape with steep
   non-quadratic walls. **PL/convexity is a local, not global, property here.**

3. **Skeletons that are LINEAR in their coefficients are globally convex.** (b) and (e)
   give `L` quadratic in `c` ⇒ a single global minimum reached from any start
   (verified: 40/60 random restarts all land on the same optimum). (e) (correct
   Fourier basis) hits machine-zero loss; (b) (wrong polynomial structure) has a
   convex landscape but an *irreducible positive residual floor* (2.9e-2) because a
   quartic cannot represent `sin(pi x)`.

4. **Over-parameterization shows up as a flat (PSD, not PD) Hessian.** (c) has a near-
   zero eigenvalue: the `cos` branch is redundant at the optimum (`c3 → 0`), leaving a
   non-identifiable / degenerate direction. The minimizer is a flat valley, not an
   isolated point.

**Bottom line for the idea.** The convexity claim is *true locally* (strictly-convex
basin at the solution) and *true globally only when the skeleton is linear in its free
coefficients* (e.g. a fixed basis with only outer linear weights). For skeletons with
nonlinear "shape" parameters (frequencies, exp rates) — which is exactly what FEX's RL
search produces — the coefficient problem is **non-convex globally**, so the BFGS stage
still relies on good initialization / the RL stage landing in the right basin. A useful
follow-up: characterize *which* operator/skeleton families yield coefficient-linear
(hence globally convex) fits, since those are the cases where stage (2) is provably easy.
