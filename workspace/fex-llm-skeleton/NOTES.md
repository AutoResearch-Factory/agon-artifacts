# FEX-LLM-skeleton pilot

Idea: `ideas/fex-llm-skeleton.v1.md` — use an LLM to propose full expression-tree
*skeletons* as a warm-start for FEX's RL search, combining LLM scientific priors
with FEX's exact parameter optimization.

This pilot tests the simplest falsifiable version of the claim on the **standard
FEX Poisson benchmark**:

> PDE  −Δu = f  on [−1,1]^d, boundary u = g, with f = −d (constant).
> True solution u\*(x) = 0.5·Σ x_i².

We compare three arms:
- **(a) LLM alone** — best relative-L2 of the raw LLM postfix candidates, using the
  constants exactly as written (only an affine wrapper at identity).
- **(b) FEX alone** — run the real FEX controller (operator search over the
  `depth2_sub` tree), take the best discovered skeleton, finetune its parameters.
- **(c) LLM + BFGS** — each LLM candidate's free parameters tuned by Adam→LBFGS.

**Pilot success metric:** some LLM candidate, after BFGS parameter tuning, reaches
relative-L2 < 1e-3.

## Result: SIGNAL POSITIVE

| arm | rel-L2 (d=5, seed 0) | how it found the structure |
|---|---|---|
| (a) LLM alone (raw consts) | **4.0** | 1 API call; right *form*, wrong constant (2.5 vs 0.5) |
| (b) FEX alone | **1.84e-6** | ~14 controller epochs ×10 cand ≈ 140 evals, 152 s search |
| (c) LLM + BFGS | **7.37e-7** | 1 API call + 1 BFGS fit (~1 s) |

LLM+BFGS clears the 1e-3 bar by ~3 orders of magnitude. Stable across seeds and
dimension:

| run | dim | seed | best raw | best LLM+BFGS | success |
|---|---|---|---|---|---|
| results_llm.json        | 5  | 0 | 4.0 | 7.37e-7 | yes |
| results_llm_seed1.json  | 5  | 1 | 4.0 | 7.67e-7 | yes |
| results_llm_seed2.json  | 5  | 2 | 4.0 | 1.37e-8 | yes |
| results_llm_d10.json    | 10 | 0 | 9.0 | 6.18e-8 | yes |

## What the LLM actually proposed

deepseek-chat, asked for 5 postfix candidates, returned the **correct sum-of-squares
skeleton every time**, differing only in the (wrong) leading constant and occasional
spurious extra terms. Examples (d=5):

```
1. x1 x^2 x2 x^2 x3 x^2 x4 x^2 x5 x^2 + + + + 2.5 *
2. ... 2.5 * 1 +
3. ... 2.5 * x1 +          # spurious +x1
4. ... 2.5 * x2 x3 * +     # spurious +x2*x3
5. ... 2.5 * x4 sin +      # spurious +sin(x4)
```

So the LLM nails the *structure* but not the *coefficient* — exactly the
division of labor the idea bets on (LLM = structure, FEX/BFGS = parameters). Raw
LLM output (arm a) is useless (rel-L2 = 4.0) because the constant is 5× too big;
one BFGS fit recovers scale≈0.5 and the error collapses to ~1e-7.

Clean skeletons (cands 1–2) reach ~1e-7; skeletons polluted with multiplicative /
variable junk (cands 3–4 at d=5) only reach ~1.3e-3 — just above threshold —
because BFGS can only shrink the spurious term via the global scale. A *constant*
offset (d=10 cands 3–5) is harmless: the affine wrapper absorbs it. Takeaway:
skeleton quality matters; the LLM's best candidates are clean.

## Important caveat: this benchmark does not stress the search

FEX's `depth2_sub` tree template already hard-codes a quadratic basis (`x^2`
leaves summed). In the **full** earlier run (`fex_baseline_ckpt/log0.txt`,
original 20000-iter finetune) the controller's tracked `smallest_error` sat at
~2e-8 **from epoch 1** — the structure is essentially given by the template.

In the budget-matched run (`fex_baseline.py`, lighter inner loop) the controller
still took ~14 epochs of stochastic sampling to reliably select the all-`x^2`
action. Either way, on this PDE FEX finds the structure cheaply, so the LLM's
warm-start advantage here is real but modest *in wall-clock*. The honest read:

- **Positive**: LLM proposes the correct PDE-solution skeleton zero-shot, and
  BFGS finishes the job to ~1e-7. The structure-vs-parameter split works.
- **Caveat / generalization risk**: on this easy, separable, symmetric solution
  the LLM may be recalling a textbook answer, and FEX's template already encodes
  it. The idea's headline claim (>70% search-iteration reduction) needs a PDE
  whose solution is **not** trivially in the tree template and **not** a memorized
  closed form — e.g. a non-separable solution, or one requiring an unusual
  operator arrangement. That is the right next experiment.

## Cost

deepseek-chat, ~500 input + ~400 output tokens total across all queries
≈ **$0.0006** (responses cached in `llm_response*.json`; re-runs are free).
Well under the $0.10 budget. All compute on local RTX 4060 Ti (CUDA), shared with
other FEX teammates — runs were kept small.

## Files

- `expr.py` — postfix parser + parametrized torch evaluator (operators
  `+ - * / sin cos exp x^2 x^3 x^4`; every numeric literal becomes a trainable
  param; whole expression wrapped as `scale*expr+shift`).
- `pde.py` — FEX-matched Poisson loss (`function_error + 100*bd_error`, MSE),
  `get_boundary` replicated from the FEX controller, Adam→LBFGS fit, relative-L2.
- `llm_gen.py` — deepseek-chat query (prompt per team-lead spec) + candidate parse.
- `run_pilot.py` — arms (a) and (c); writes `results_llm*.json`.
- `fex_baseline.py` — arm (b); imports the cloned `controller_poisson.py` so the
  tree/loss are byte-for-byte FEX, runs the search, finetunes with the
  LLM-matched budget; writes `results_fex_d5.json`.
- `fex-code/` — upstream clone (https://github.com/LeungSamWai/Finite-expression-method).
- `fex_baseline_ckpt/log0.txt` — trajectory of one full standard FEX run (killed
  before its 20000-iter finetune; kept for the "structure found at epoch 1" note).

## Reproduce

```bash
source venv/bin/activate
python run_pilot.py --dim 5  --seed 0            # arms (a)+(c)
python run_pilot.py --dim 10 --seed 0 --out results_llm_d10.json --cache_llm llm_response_d10.json
python fex_baseline.py --dim 5 --epochs 20 --seed 0   # arm (b)
```

## Recommendation

Promote past pilot. The structure/parameter division of labor is validated. The
next experiment must use a PDE where the solution skeleton is genuinely hard for
FEX's template (non-separable / non-memorizable) to test the real search-speedup
claim, and should inject the LLM skeleton into the FEX *controller initialization*
(the idea's actual mechanism) rather than only doing standalone BFGS.

## v2 re-grounding: prior granularity map

Reviewer feedback pushed the idea away from "LLM writes a full skeleton for FEX"
and toward the sharper question: what granularity of structure prior should FEX
trust, and how brittle is a hard skeleton when it is slightly wrong?

I added `prior_granularity_pilot.py`, which fits candidate skeletons under a
generic FEX-style PDE residual plus boundary loss on three analytic PDE cases:

| case | full skeleton | corrupted hard skeleton | partial skeleton | best random operator-set proxy |
|---|---:|---:|---:|---:|
| Poisson quadratic d=5 | 6.27e-7 | 9.76e-3 | 2.60e-1 | 4.21e-1 |
| rotated quadratic d=5 | 9.04e-7 | 9.91e-3 | 1.30e-1 | 4.83e-1 |
| sine interaction d=4 | 1.59e-6 | 1.34e-2 | 4.32e-2 | 4.82e-1 |

Run details:

```bash
timeout 1200 venv/bin/python data/build_pde_suite.py
timeout 1200 venv/bin/python prior_granularity_pilot.py --out results/prior_granularity_pilot.json
```

The signal is positive for the revised framing. Correct full skeletons are easy
for Adam+LBFGS to finish, even with wrong coefficients. Partial skeletons and
hard polluted skeletons are clearly worse, and a random operator-set-only proxy
does not come close in this small budget. This is not yet a FEX-controller
result; the next proposal must replace the random proxy with Bhatnagar-style
operator-set restriction and inject top-k LLM skeletons as a soft controller bias.

## v3 re-grounding: recoverable soft-prior proxy

Reviewer feedback correctly noted that v2 still underspecified how a top-k
skeleton prior remains recoverable when the top candidate is wrong. I added
`soft_prior_controller_pilot.py`, a finite-library controller proxy. It scores
sampled candidates with the same FEX-style PDE residual + boundary loss as the
v2 granularity pilot, but changes the search policy:

- hard corrupted top-1: evaluate only the wrong top skeleton;
- soft top-k with support: evaluate an imperfect LLM ranking where rank-1 is
  corrupted and rank-2 is the correct skeleton, with fallback support;
- operator-set uniform proxy: unranked search over the same finite support;
- oracle: correct full skeleton first.

Run:

```bash
timeout 1200 venv/bin/python soft_prior_controller_pilot.py --out results/soft_prior_controller_pilot.json
```

CUDA result (RTX 4060 Ti, 9.5s): hard corrupted top-1 fails on all three cases
(relL2 1.1e-2 to 1.3e-2). Soft top-k succeeds in 2 evaluations on all three
cases (best relL2 1.05e-7 to 3.48e-7), within one evaluation of the oracle.
Unranked operator-set search succeeds on Poisson in 5/5 shuffled trials
(median 5 evals), but only 1/5 trials on rotated quadratic and sine interaction
within the same 8-eval budget. This supports the recoverability mechanism but
still does not replace the proposal-stage real FEX controller logit/KL-bias
sweep against Bhatnagar-style operator-set restriction.
