# FEX residual critic — pilot

**Question.** Can structural features of a *partial* FEX expression tree predict the
*final* fitting residual (after coefficient optimization)? If yes, a learned value
critic could supply dense reward shaping for FEX's sparse RL search.

**Pilot.** Toy 1D regression as a stand-in for PDE residual quality.
- Target `f(x) = sin(2*pi*x) + 0.5*x^2` on `[0,1]`, 100 points.
- Operator library (7): `sin, cos, +, -, *, x, const`.
- 2000 random depth-2 trees: `root_binary( leaf_L , leaf_R )`, each leaf
  `a*op(w*x)+b` (FEX-faithful affine form with inner input frequency `w`).
- Feature = 21-dim one-hot (`root|left|right` over the 7 ops).
- Optimize each tree's coefficients (Nelder-Mead, 6 restarts) -> final MSE.
- Train/test = 1500/500. MLP (64,64) predicts `log(MSE)` from the 21-dim feature.

**Result (`run_A.log`).**
- Spearman rho (pred vs actual log-MSE, test) = **0.918**.
- AUROC, good (`MSE<0.01`) vs bad (`MSE>0.2`) = **1.000** (276 good / 134 bad in test).
- Full-set MSE: min 2.8e-9, median 5.9e-3, max 0.36; 1080 good, 517 bad of 2000.

**Read.** Strong positive signal for the idea in this toy regime: partial-tree
structure essentially *determines* final residual quality, so a cheap critic
recovers it almost perfectly. AUROC=1.0 is expected here — with 3 categorical
slots the good/bad trees form cleanly separable clusters. The open question for a
real study is whether the signal survives larger libraries, deeper trees, and
genuine PDE residuals (where coefficient optimization is far less identifiable).

**Setup note.** The original spec used leaves `op(x)` (no inner `w`) and a bad
threshold `MSE>1.0`. Both make the toy degenerate for this target: `sin(x)` can't
reach the `2*pi` frequency (every tree fits equally poorly, MSE collapses to
0.20–0.39) and a bounded target can never exceed `MSE~0.39`, so `>1.0` is
unreachable. The literal-spec run still gives rho=0.977 but AUROC is undefined
(no good/bad trees exist). The fixes above (FEX-faithful leaves + achievable
bad threshold) keep the library/encoding/shape/target and make the good-vs-bad
question computable. See `probe_expressivity.py` for the diagnostic.

## Run
```
python3 -m venv .venv
.venv/bin/pip install numpy scipy scikit-learn
timeout 600 .venv/bin/python pilot_critic.py
```
