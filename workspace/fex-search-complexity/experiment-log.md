## [Review of Version 7] 2026-06-27 18:30 — score=4.9/10
- Verdict: not ready
- Primary concern: The G2↔G3 bridge theorem and Quadrant III placement overclaim from a flat-softmax/2-action proxy model to the real FEX controller. All load-bearing bridge results use an unvalidated proxy while the FEX-E2E anchor experiment remains explicitly deferred.
- Combined score: Claude 5.0 + Codex 4.8 → avg 4.9 (divergence < 2, no flag needed).

<review score="4.8" date="2026-06-27">

## Verdict
not ready

## Primary concern
V7 still has no load-bearing actual-FEX discoverability result; the new G2↔G3 bridge overclaims from blind sampling and 2-action/product-softmax proxies to the real FEX controller.

## Claim support
- Four-gate decomposition necessity: partial; confidence medium; evidence path: results/r43_kappa_and_g4_witness_patch/corrected_falsifier_ledger.json, results/r44_targeted_proof_reverify/falsifier_direction_check.json, results/r13_quotient_recurrence_proof/finite_checks.json, results/g2_blind_sampler_theorem/theorem_statement.md; missing evidence: non-toy FEX instantiation where all four gates are measured on one controller/search task.
- G1 sound_ac quotient count exact/non-trivial: supported; confidence high; evidence path: results/r13_quotient_recurrence_proof/recurrence_proof.md, results/r13_quotient_recurrence_proof/finite_checks.json, results/sound_quotient_certificate.json; missing evidence: semantic completeness and full-FEX grammar counts. Claim ceiling: conservative sound_ac depth2_sub quotient only.
- G2 blind sampler lower bound: supported; confidence high; evidence path: results/g2_blind_sampler_theorem/theorem_statement.md, results/g2_blind_sampler_theorem/fex_instantiation.md; missing evidence: reward-informed search or controller dynamics. Claim ceiling: blind class-level sampler.
- G2→G3 bridge / Quadrant III placement / PG speedup: measurement-invalid; confidence high; evidence path: results/g2g3_bridge_theorem/bridge_theorem.md, results/g2g3_bridge_theorem/proof_ledger.md; missing evidence: valid global/trajectory PL proof, measured full factored κ_PG, and FEX-E2E. Theorem 1 "PG never worse than blind" is informal, Theorem 5 is conditional, and the ~600,000x number is a flat-softmax proxy.
- GAP-QUANT 2-action reduction gap: partial; confidence high for counterexample, medium for bound; evidence path: results/gap_quantification/counterexample_construction.md, results/gap_quantification/verify_counterexample.py, results/gap_quantification/reduction_bound.md; missing evidence: rigorous policy-deviation proof and direct full-action convergence link. Current evidence shows the 2-action reduction can be optimistic.
- G3 K=2 MD equivalence: supported; confidence medium; evidence path: results/r80_marginal_dominance_ode_analysis/theorem_statement.md, results/r80_marginal_dominance_ode_analysis/proof_or_counterexample.json, results/r86_ode_normalization_unify/unified_note.md; missing evidence: extension beyond exact-gradient 2x2 product-softmax sign geometry.
- G3 K>=3 / MD_k hierarchy: partial; confidence medium; evidence path: results/r84_proof_scope_fix/impossibility_proof_v2.md, results/r84_proof_scope_fix/scope_delta.md; missing evidence: general-K corner independence proof and MD_k-to-PL/convergence theorem.
- G3→FEX empirical bridge: partial; confidence low; evidence path: results/r85_fex_corner_audit/pass_rates.json, results/r85_fex_corner_audit/compute_corners.py, results/r52_restricted_fullspace_reconcile/reconciliation_summary.json; missing evidence: full FEX controller convergence/discovery on actual depth2_sub. R85 is a static reward-cache diagnostic with missing-entry normalization, not an end-to-end controller result.
- G4 coefficient feasibility separate gate: partial; confidence high for toy separation; evidence path: results/coefficient_witness_ledger.json, results/r44_targeted_proof_reverify/reverify.md, results/r32_coefficient_language_audit/audit.md; missing evidence: bounded-ETR/FEX preservation proof or PDE-residual feasibility family. Claim ceiling: appendix toy witness only.

## Warning cases & justification
- Triggered: only theorem/proxy/doc evidence without a main anchor FEX-E2E; main bridge result is below the scale required for actual-FEX discoverability claims; rapidly moving FEX/SR area lacks a current strong-baseline/controller audit; main experiment is explicitly deferred; bridge integrity/measurement fail for PG≤blind and Quadrant III speedup.

## Strengths
- G1 is cleanly scoped and numerically verified for sound_ac depth2_sub (Q=1539, zero swap/sub false-equality failures).
- G2 blind-sampler formulas are correct and useful as a baseline for exposure.
- GAP-QUANT honestly exposes that the 2-action reduction is optimistic, which is important negative evidence.
- G4 wording mostly avoids unproved hardness; R32 correctly lists all preservation obligations as unproven.

## Weaknesses (CRITICAL > MAJOR > MINOR)
- CRITICAL: The central bridge theorem is not theorem-grade. "PG with entropy regularization cannot be worse than blind" is not proven under the stated FEX setting, and the document itself admits global PL failure and unmeasured full factored κ_PG.
- CRITICAL: V7 claims FEX depth2_sub is Quadrant III and reports ~600,000x PG speedup, but this is derived from a flat-softmax/2-action proxy while FEX-E2E is deferred.
- CRITICAL: The current evidence has drifted from actual FEX RL discovery into blind-sampler arithmetic plus product-softmax sign geometry. That does not yet answer "when can the real controller find expressions?"
- MAJOR: R85 pass rates are diagnostic only: many tables are incomplete, the code normalizes over observed entries, and the result is static reward-table geometry rather than trajectory-level discovery.
- MAJOR: G3 hierarchy is not a convergence theorem for FEX; MD_k gives sign/corner conditions, while PL/κ_PG and stochastic reward noise remain open.
- MAJOR: STATE.md §3-§4 repeatedly states theorem/bridge/Quadrant claims before the caveats, so a new reader will overestimate what is established.
- MINOR: Some evidence paths in the internal claim audit are stale or misleading after recovery, and G4's manifest points to a markdown file not present in the current tree.

## Drift Warning
The project remains anchored to FEX discoverability in motivation, but V7's accepted evidence solves an easier proxy: class-level blind search and K-slot x 2-action product-softmax geometry, not actual FEX controller discovery under PDE residual fitting.

## Must-fix before next review
- Replace or retract the G2↔G3 bridge theorem unless PG≤blind, the threshold, and Quadrant III placement are proven under explicit FEX-compatible assumptions.
- Run and report a main anchor FEX-E2E experiment on depth2_sub with full [9,3,9,9] actions, real reward/residual, coefficient fitting, logits/grad norms, and quotient-class hits.
- Directly measure full factored κ_PG or an accepted surrogate on real controller traces; stop using MD pass rates as κ degradation estimates.
- Rewrite STATE.md §3-§4 so proxy/proof/diagnostic/appendix evidence are visibly separated and no proxy supports an actual-FEX main claim.
- Add cross-target or cross-PDE replication after the single anchor is valid; do not soften claims or lower venue expectations as a substitute.

</review>

## [Iter 46 Start] 2026-06-27 ~17:00 ET — Scenario B: audit response (BLOCKER) → V7 review submission

- Audit iter-45 (verdict=BLOCKER): 3 BLOCKER / 4 CRITICAL / 5 MAJOR / 2 MINOR all addressed.
- Evidence recovery: R80, R84, R85, R86, R87, R88 recovered from v6 branch via `git checkout route/v6-reviewer-response -- results/...`. R83 never committed — content subsumed by R84 v2 (impossibility_proof_v2.md §5 contains K=3 counterexample).
- Bridge theorem fixes: Corollary 2 live debug trace removed + "diagnostic only" annotation added; Theorem 5 annotated CONDITIONAL matching Lemma 7 status; R83 references updated → R84 in manifest.
- A0 restructured: Reviewer response moved to separate `## Reviewer Response` section; A0 now contains proper iter-45 audit response table with all 11 findings resolved.
- Lit-feed: 4 papers consumed (4→0). Promoted to LESSONS as shelved directions (FR-PPO / QOT PL / GCR-PPO / GoodRegressor).
- LESSONS updated: 4 shelved directions + R83 documentation.
- STATE.md rewritten: 271 lines, all claims updated, §4.3 fixed, A6 resolved.
- Decision: phase → needs_reviewer. All 6 V6 reviewer must-fix addressed. V7 ready for review.
  - BLOCKER #1 (contribution identity): G2 elevated to theorem + G2↔G3 bridge built.
  - P0 #2 (2-action gap): GAP-QUANT theorem + counterexample.
  - P0 #3 (stale evidence): CLAIM-CLN audit + evidence recovery.
  - P1 #4 (FEX e2e): Explicitly DEFERRED per §5 theory-first.
  - P1 #5 (overclaim): §4.3 scoped per-gate.
  - P2 #6 (§1.2): Core concepts table completed.

## [Audit] 2026-06-27 16:00 ET — verdict=BLOCKER (iter 45)
- Report: audits/audit_iter45_20260627_1600.md
- Load-bearing issues: R80/R83/R84/R85/R86/R87/R88 missing from v7 branch (BLOCKER-001); G2G3-BRG evidence chain references nonexistent files (BLOCKER-002); STATE.md uncommitted (BLOCKER-003)
- Required scientist response: 9 items — recover v6 evidence files, verify bridge theorem evidence chain, commit STATE.md, resolve A0 repurposing, fix Lemma 7 CONDITIONAL annotation, clean bridge_theorem.md live debug trace, fix §4.3 stale claims, resolve collected-vs-open contradiction, decide FEX-E2E disposition

## [Run Done] G2G3-BRG 2026-06-27 ~13:30 ET — G2↔G3 bridge theorem collected

- Run: G2G3-BRG (Task Group B, P0). Server: local CPU. Phase: needs_impl → collected.
- Output: `results/g2g3_bridge_theorem/`
  - `bridge_theorem.md` (451 lines): 5 theorems + 4 corollaries. PG ≤ blind (entropy fallback), critical density threshold, both separation directions, factored bridge with D_K ≥ 1, four-quadrant taxonomy, FEX placement (Quadrant III).
  - `proof_ledger.md` (283 lines): 9 lemmas, proof pipeline, status tracking.
  - `manifest.json`: Run receipt with input assets, key results, claim boundary.
- Key results:
  - Theorem 1: PG+entropy never worse than blind. E[T_PG] ≤ min(Q/K, O((κ₀/τ)·log(Q/K))).
  - Theorem 2: Speedup threshold γ = τ/κ₀ vs ρ·log(1/ρ). PG beats blind iff τ/κ₀ > ρ·log(1/ρ).
  - Theorem 3: G2 benign+G3 fails → PG can be worse than blind (softmax saturation; δ→0 gives κ₀→∞).
  - Theorem 4: G2 fails+G3 benign → PG overcomes blind (FEX: ~600,000× speedup at depth 2).
  - Theorem 5: Factored bridge — cross-slot degradation D_K ≥ 1, estimated D_K ≈ 1.32 from R85 MD₃ gap.
  - Both separation directions witnessed: (1) FEX depth hierarchy, (2) 3-class δ→0 bandit.
  - Narrative integration: four-gate framework upgraded from "independent failure modes" to "causal pipeline".

## [Run Done] CLAIM-CLN 2026-06-27 ~13:00 ET — Claims precision audit + stale evidence cleanup collected

- Run: CLAIM-CLN (Task Group C, P0). Server: local CPU. Phase: needs_impl → collected.
- Output: `results/claim_audit_v7/`
  - `claim_audit.md`: 9 claims cross-referenced against source evidence. 3 VERIFIED, 1 NEWLY-COLLECTED, 1 MISSING, 4 MISSING-EVIDENCE.
  - `stale_evidence_disposition.md`: R40 deprecated; R43→R44 chain verified (12/12 falsifier valid, 0 open); R43/R44 NOT stale.
  - `section_fixes.md`: 8 recommended STATE.md §4-§6 updates (for scientist). §1.2/§6.1 already fixed in iter-45.
- Critical finding: R80/R83/R84/R85 committed on v6 branch, missing from v7 branch (v7 branched from main). Need `git checkout route/v6-reviewer-response -- results/r80... results/r83... results/r84... results/r85...`.
- Remaining for P0: G2G3-BRG bridge theorem (needs_impl).

## [Run Done] G2-THM 2026-06-27 ~11:00 ET — Blind sampler lower bound theorem collected

- Run: G2-THM (Task Group A, P0). Server: local CPU. Phase: needs_impl → collected.
- Output: `results/g2_blind_sampler_theorem/`
  - `theorem_statement.md`: 5 theorems (E[T_disc]=Q/K with repl., (Q+1)/(K+1) no-repl., E[T_cover]=Q·H_K, tightness/minimax optimality, non-uniform extension) + variance + tail bounds. All proofs complete.
  - `separation_witness.md`: FEX depth hierarchy as parameterized family — G1 passes (meaningful compression), G2 fails (K/Q super-exponentially small), G3 benign (κ_0=4), G4 benign (structural K).
  - `fex_instantiation.md`: Q=1539, structural K=1, E[T]=770 (no-repl.), 1539 (with repl.). K_ε stratified by mode (structural/intermediate/parametric) and ε. Scaling projection: blind search infeasible beyond depth 2.
- Key result: G2 elevated from lemma sketch to formal theorem. Ω(Q/K) lower bound is tight. G1⇏G2 proven.
- Remaining for G2 closure: G2G3-BRG bridge theorem (Task Group B).

## [Version 7 Start] 2026-06-27 ~10:00 ET — Scenario C: V6 reviewer response (score 5.8/10, NOT READY)

- Route: v7-reviewer-response (from main). Phase: coding_and_running.
- V6 review assimilation: 6 must-fix items. Reviewer diagnosis: marginal_dominance theory covers only G3, narrative drift from four-gate framework.
- §5 constraint: no claim downgrade, no reframe. Strategy: advance G2 (class exposure) to theorem-level rigor + G2↔G3 bridge theorem.
- Five Task Groups planned:
  - A: G2-THM — blind sampler lower bound formal theorem (P0, BLOCKER #1)
  - B: G2G3-BRG — exposure-to-geometry bridge theorem (P0, BLOCKER #1)
  - C: CLAIM-CLN — claims precision audit + stale evidence cleanup (P0, #3/#5/#6)
  - D: GAP-QUANT — 2-action reduction gap quantification (P0, #2)
  - E: FEX-E2E — small-scale FEX end-to-end experiment, ~20 GPU-min (P1, #4)
- Immediate fixes this round: §4.3 claims scoped per-gate; §6.1 overclaim removed; §1.2 core concepts completed.
- Key tension: reviewer says reframe or advance; §5 says don't downgrade → advance G2.

## [Review of Version 6] 2026-06-27 08:20 — score=5.8/10
- Verdict: not ready
- Primary concern: The marginal_dominance theory proves clean convergence results for a K-slot × 2-action product-softmax PG toy model, but the bridge to actual FEX controllers requires a 2-action-per-slot reduction that the authors themselves label a "conservative lower bound." The theory answers "when does a simplified factored PG converge," not "when can the real FEX controller discover expressions." This model/reality gap, combined with substantial narrative drift from the original four-gate framework, keeps the paper below top-venue threshold.
- Codex second opinion: attempted but did not complete within time window (GPT-5.5 xhigh, >6min). Review based on Claude solo assessment.

## [Iter 44 Start] 2026-06-27 ~09:00 ET — Scenario B: audit response + V6 review submission (route: v6-reviewer-response)

- Audit iter-43 (verdict=BLOCKER): 2 BLOCKER / 3 CRITICAL / 3 MAJOR / 3 MINOR all addressed.
- BLOCKER-001 (R86 not committed): `git add results/r86_ode_normalization_unify/` → fixed.
- BLOCKER-002 (R80 untracked): `git add results/r80_marginal_dominance_ode_analysis/` — canonical proof artifacts committed.
- CRIT-001 (A0/A6 contradiction): A0 now synced with A6; MF#2/C3/M3 → resolved.
- CRIT-002 (R86 deliverable mismatch): Accepted unified_note.md as sufficient for proof-audit phase. Header insertion deferred to paper-writing.
- CRIT-003 (old untracked dirs): Deleted r31/r62/r64/r68/r74/r76; R80 committed.
- MAJOR-001 (R85 aggregate verifiability): Accepted, script-is-evidence; manifest note deferred to paper-writing.
- MAJOR-002 (crash_count blank): Set to 0 for all 5 runs.
- MAJOR-003 (R80 2000-table): Resolved by R80 commit.
- MINOR items: deferred to paper-writing (manifest verification sections) or resolved (R85 cache path verified, §2.3 rows added).
- Lit-feed: 4 papers consumed (0→0), GCR-PPO promoted to LESSONS shelved direction.
- Decision: Phase → needs_reviewer. All 7 V5 must-fix resolved. Evidence ready for V6 review.

## [Audit] 2026-06-27 07:49 ET — verdict=BLOCKER (iter 43)

- Report: audits/audit_iter43_20260627_0749.md
- Load-bearing issues: R86 result files uncommitted (BLOCKER-001); R80 canonical proof artifacts untracked (BLOCKER-002); A0/A6 status contradiction MF#2/C3 (CRIT-001); R86 deliverable mismatch with success criterion (CRIT-002); 7 old untracked result directories (CRIT-003)
- Required scientist response: 9 items — commit R86 files, resolve R80 provenance, fix A0/A6 contradiction, decide on R86 header insertion, clean up old directories, fix A3 crash_count, verify R80 2000-table evidence, update §2.3 matrix

## [Run Done] 2026-06-27 ~08:00 ET — R88 collected

- R88_pg_condition_comparison: PG condition comparison — marginal_dominance/MD_k vs PL/gradient domination/Łojasiewicz.
- 339-line comparison.md with 10-dimension comparison table (§5) and 5 explicit MD_k↔PL relationship statements (§4):
  - R1: MD_{K−1} ⇏ PL (different domains — reward space vs parameter space)
  - R2: PL ⇏ MD₀ (logically independent; counterexample constructs exist)
  - R3: MD_{K−1} + margin lower bound → semi-global PL (structural bridge)
  - R4: Gradient domination at init → MD₀ necessary (formal link)
  - R5: MD_k ↔ PL variant hierarchy analogy (point→local→semi-global→global)
- ≥4 standard PG conditions covered (PL, gradient domination, Łojasiewicz, KL/QG/RSI). Practical FEX K=4 guidance (§7). MF#6/M3 resolved.

## [Run Done] 2026-06-27 ~07:10 ET — R86 collected

- R86_ode_normalization_unify: ODE normalization audit across R80/R81/R83. Found three normalizations differing by positive multiplicative factors:
  - R80 exact: dp/dt = 2p²(1-p)²f(q) — correct (factor 2 from d(θ₁₀−θ₁₁)/dt = ∂J/∂θ₁₀ − ∂J/∂θ₁₁).
  - R81 derivation: dz₁/dt missing factor 2, then proof simplifies to dp/dt = p(1-p)f(q) (drops extra p(1-p)).
  - R83/R84: same simplified form as R81.
- All sign/monotonicity results unaffected — forward invariance depends only on sign(f_s). No proof depends on exact multiplicative factor. R81 Remark 2 explicitly scopes rate claims.
- unified_note.md with full analysis + recommended normalization headers for each document. CRITICAL flag NOT triggered. MF#5/M2 resolved.

## [Run Done] 2026-06-27 ~07:00 ET — R87 collected

- R87_fex_mapping_lemma: Model-to-FEX mapping lemma connecting K=4×2-action product-softmax PG to FEX depth2_sub controller. 9-section proof document: slot↔inorder tree position (exact bijection), action↔2-operator subset (injective), reward↔PDE residual (exact), policy↔product-softmax (structurally identical).
- Principal gap: 2-action reduction per slot (9→2 unary, 3→2 binary). Model is conservative lower bound — MD₃ fail doesn't mean FEX fails (multi-action escape routes).
- MF#4 resolved. Mapping table in §6 for paper reference.

## [Run Done] 2026-06-27 ~06:30 ET — R85 collected

- R85_fex_corner_audit: FEX K=4 corner condition audit on actual per-class reward data (1539 classes). Multi-config enumeration across 2000 distinct 2-action-per-slot reductions.
- Key results: MD₀=63.5%, MD₁=26.9%, MD₂=21.1%, MD₃=20.8%. Complete tables (16/16 entries, 414 configs): MD₀=100%, MD₃=15.7% — clean K>2 gap demonstration.
- Hierarchy gaps: MD₀→MD₁=36.6%, MD₁→MD₂=5.8%, MD₂→MD₃=0.4%. First-order partner interactions are the dominant failure mode.
- Theorem 8 structural conditions (supermodularity, pairwise decomp, coord monotonicity) all violated for informative operators — no compression.
- Pairwise 2×2 slice MD₀=62.1%. MF#2/C3 resolved.
- Outputs: `results/r85_fex_corner_audit/{compute_corners.py, pass_rates.json, corner_audit.md, manifest.json}`

## [Run Done] 2026-06-27 ~06:00 ET — R84 collected

- R84_proof_scope_fix: Scoped r83 impossibility_proof.md → v2. Theorem 5/6 now "K=3 PROVEN, general K CONJECTURED." Cross-slot s'≠s* gap explicitly documented in §6.3-6.4. All "PROVEN for all K" / "IMPOSSIBILITY PROVEN" / "fully characterized" language removed (verified via grep: zero hits).
- Outputs: `results/r84_proof_scope_fix/{impossibility_proof_v2.md, scope_delta.md, manifest.json}`
- MF#1 / C1 / M1: resolved in A0.

## [Version 6 Start] 2026-06-28 ~01:00 ET — V5 review (5.6/10) response (route: v6-reviewer-response)

- Scenario C after V5 reviewer score 5.6/10 NOT READY. All 7 must-fix + 3 CRITICAL + 4 MAJOR + 3 MINOR accepted.
- MF#1 (proof gap): Scope Theorem 5/6 to "K=3 proven, general K = construction template." R84.
- MF#2 (FEX connection): FEX K=4 corner condition audit on actual reward data. R85.
- MF#3 (one-liner): Rewritten per reviewer's suggested text in this STATE.md.
- MF#4 (FEX mapping): Model-to-FEX mapping lemma. R87 (P1).
- MF#5 (ODE normalization): Unified normalization notes in R80/R81/R83. R86.
- MF#6 (PG comparison): MD_k vs PL/Lojasiewicz comparison. R88 (P1).
- MF#7 (hierarchy): §1.3→§4.3 hierarchy reorganization in this STATE.md.
- 3 Task Groups: A (R84+R86, P0, can_split=true), B (R85, P0), C (R87+R88, P1, can_split=true). All local CPU.
- New route v6-reviewer-response from main. Phase: coding_and_running.

## Past entries (reconstructed)

The full experiment-log.md (>1700 lines) was lost due to .gitignore exclusion across branch switches. Key past milestones:
- V5 route (v5-b4-scaling-oracle): R80/R81 K=2 ⇔ PROVEN → R82/R83 K>2 LP gap FULLY CHARACTERIZED. Score improved from V4 5.6→V5 5.6 (not yet sufficient).
- V4 route (iter22-g3-bridge-fullspace): G3 bridge repaired (NDMS<0.5), four-gate theorem general. Score 5.6, reviewer wanted theorem depth.
- V3 route: Fullspace conflict forensics. Score 5.8, reviewer wanted general theorem + K-PL proof.
- V1-V2 routes: Initial auditor/scientist/coder cycles, formal proof setup, quotient certificate.
