# Experiment Log: fex-synchro-prior

## [Iter 14 Start] 2026-06-28 — scientist: audit BLOCKER response + MF optimizer exhaustion plan
- Scenario B after `[Audit]` iter14 verdict=BLOCKER.
- Lit-feed inbox: `unprocessed: 0`, nothing to process.
- Audit response: Accepted all 7 findings. Key: BLOCKER-001 (MF optimizer not exhausted) is correct — capacity 1.83e-6 vs os s3 1.90e-3 gap is not proven fundamental. Must try 50K finetune, L-BFGS, combo, increased coarse eval, and perturbed-alpha probe before declaring MF infeasible.
- Evidence reconstruction:
  - g10 VES 4/5 machine (median 1.12e-6) > oracle 3/5 > std 0/5 — project's strongest result
  - g100 oracle 4/5 machine (median 1.13e-6) > VES 2/5 > std 1/5 — iter12 "VES best" fully reversed
  - g100 VES estimator data: FFT correct (22.09≈7π), gate PASS (0.075<<0.5), selected_bases identical to g10 — degradation is search noise, NOT estimation failure
  - MF: capacity PASS (1.83e-6), freq selection PASS (first_hits ep0/ep21), optimization FAIL (best 1.90e-3, search_error=1495)
- §4 rewritten with n=5 nonlinear tables and MF optimizer status.
- New A1 plan:
  - Group A P0 `can_split=true`: 5 MF optimizer diagnostics (mf-ft-50k-s3, mf-ft-lbfgs-s3, mf-ft-combo-s3, mf-search-100-oracle, mf-probe-perturbed). Total <25 GPU-h.
  - Group B P0: f12 provenance metadata fix.
  - Group C P1 depends Group A: MF full 3×5 comparison if any fix ≤1e-4.
- LESSONS.md: added 3 entries (n=1→n=5 reversal, estimator vs search noise, exhaust low-cost fixes).
- STATE.md: frontmatter iteration=14, phase=coding_and_running. 326 lines.

## [Audit] 2026-06-28 16:20 — verdict=BLOCKER (iter 14)
- Report: audits/audit_iter14_20260628_1620.md
- Load-bearing issues:
  - [BLOCKER-001] Multi-frequency optimizer bottleneck: capacity PASS (probe 1.83e-6) but additive grammar 15/15 FAIL (best 1.90e-3). 低成本 finetune 扩展未测试（50K/100K 步, L-BFGS, multi-stage opt）
  - [CRIT-001] g100 VES narrative 被 n=5 完全推翻: oracle_soft 4/5 machine (best) > VES 2/5 (middle) > standard 1/5 (worst). iter12 n=1 "VES best" 是 s0 outlier
  - [CRIT-002] §4 未反映 iter14 nonlinear n=5 聚合数据和 mf optimizer 状态变化
- g10 VES 是 strong positive: 4/5 machine, median 1.12e-6, beats oracle
- Required scientist response: (1) mf finetune budget 扩展实验, (2) §4 nonlinear 用 n=5 重写, (3) §4.3 claims 更新, (4) f12 provenance metadata 修复

[Run Done] 2026-06-28T15:30 coder-1 — **mf-additive-grammar-oracle collected**:
- Synced final os s4 (relL2=0.42, no improvement over s2). **15/15 result.json all verified with real numeric values**.
- Full summary: oracle_hard 5/5 FAIL (0.71-0.92), oracle_soft 5/5 (s2=0.42, s3=**1.90e-3**, s4=0.42 — best not ≤1e-5), standard 5/5 FAIL (0.75-0.88).
- **BLOCKER confirmed**: additive grammar has expression capacity but continuous optimization (14 params, 20+20 coarse opt + 10000-step Adam) insufficient for multi-freq convergence.
- mf-additive-estimated-prior stays queued (oracle arm not passed).
- GPU +$6.00 → $1354.63.

[Run Sync] 2026-06-28T14:00 coder-1 — **5 new result.json synced, g10/g100 collected, mf BLOCKER confirmed**:
- **mf os s3**: relL2=**1.90e-3** (200× better than s2 0.42 but 190× from 1e-5). Alpha init fix works for freq selection (both 4π/17π hit ep0/ep21). Continuous optimization bottleneck confirmed — 14 params, 20+20 coarse opt insufficient. BLOCKER not lifted.
- **g10 VES s3**: relL2=**1.33e-6** ✅, s4=**1.07e-6** ✅ — both machine precision, no-true-sol, gate_passed=True. g10 VES 3/4 machine (s1,s3,s4), s2=6.47e-3 outlier. **g10 12/12 collected**.
- **g100 VES s3**: relL2=5.67e-4, s4=3.10e-03 — moderate, gate passed. **g100 12/12 collected**.
- **Status**: mf 14/15 synced (os s4 running GPU1 ep200). g10/g100 fully collected. mf-estimated-prior stays queued/blocked.
- GPU +$11.00 → $1348.63.

[Run Sync] 2026-06-28T10:50 coder-1 — **5 new result.json synced + verified. os s2 BLOCKER persists**:
- **mf os s2 (alpha init fix)**: relL2=**0.4187 FAIL (>0.1)**. BLOCKER NOT lifted despite alpha init fix. Frequency selection improved (first_hits ep0/ep3 for both 4π/17π), but search stuck at best_err=12063 from ep299 through ep999. 20+20 coarse opt + 10000-step finetune still insufficient for multi-frequency composition convergence. **Scientist MUST re-evaluate**: alpha init helps freq selection but doesn't solve the continuous optimization bottleneck.
- **g10 VES s1**: relL2=**1.12e-6** ✅ (machine precision!), uses_true_sol=False, gate_passed=True. **First no-true-sol machine precision for nonlinear g10!**
- **g10 VES s2**: relL2=6.47e-3, uses_true_sol=False, gate_passed=True (non-machine but passed gate).
- **g100 VES s1**: relL2=**7.04e-5** ✅ (<1e-4), uses_true_sol=False, gate_passed=True.
- **g100 VES s2**: relL2=2.56e-3, uses_true_sol=False, gate_passed=True.
- **Status**: mf 13/15 synced (os s2 done FAIL, os s3 ep150, os s4 pending). g10 10/12 synced (VES s3-s4 running). g100 10/12 synced (VES s3-s4 running).
- GPU +$12.13 → $1337.63.

[Run Crash] 2026-06-28T04:35 coder-1 — **mf os s4 crash (old code, old launcher)**: TypeError: `build_oracle_prior() got unexpected keyword argument 'prefer_sin'`. Same code mismatch as iter14 coder documented. Old launcher started os s4 before 06:19 fix. Retry screen fexsp-osretry-0628-061900 processes s2→s3→s4 sequentially; s4 will be retried with fixed code after s3 completes.

[Run Sync] 2026-06-28T06:19 coder-1 — **7 new result.json synced + code mismatch fixed + retry launched**:
- **Synced**: mf oh s4 (FAIL 0.71), os s0-s1 (FAIL 0.83/0.76), g10 os s3-s4 (3.8e-3/1.2e-5), g100 os s3-s4 (1.13e-6/1.12e-6). All verified: 5 candidates, 100 search_curve pts, train.log.
- **g100 oracle_soft 4/4 machine precision!** (1.12-1.26e-6). g10 os 2/4 machine + 1/4 1.2e-5 + 1/4 3.8e-3. Nonlinear evidence now n=4 seeds, statistically meaningful.
- **Code mismatch found**: controller.py/estimator.py were stale on remote. trainer.py passed `prefer_sin`/`fft_top_k` that old signatures didn't accept → os s2-s4 crashed (TypeError: prefer_sin), g10/g100 ves s1-s4 crashed (TypeError: fft_top_k). Rsynced complete src + scripts.
- **Retry launched**: 3 screen sessions — fexsp-osretry-0628-061900 (mf os s2-s4, GPU1), fexsp-g10ves-0628-061900 (g10 ves s1-s4, GPU6-7), fexsp-g100ves-0628-061900 (g100 ves s1-s4, GPU8-9). All confirmed running with fixed code.
- **mf-additive**: 12/15 results all FAIL (old code without alpha init). os s2-s4 (retry with alpha init fix) are the critical test — if pass, oh/std re-run warranted.
- GPU +$23.38 → $1315.80.

[Run Sync] 2026-06-27T14:25 coder-1 — mf-capacity-explicit-action-probe **PASSED** (best_relL2=1.83e-6 ≤ 1e-5). Collected.
[Run Crash diagnostic] 2026-06-28T02:30 coder-1 — mf-additive-grammar-oracle **finetune bottleneck root cause identified**:
- **Observed**: all 9 completed seeds (std×5 + oh×4) FAIL with relL2 0.73-0.92. Finetune from reset_params (α=1.0) with 10000 Adam steps cannot converge.
- **Root cause**: `reset_params` sets alphas to 1.0, but target requires α₁=4π/12≈1.047, α₂=17π/24≈2.225 for component 1/2. Adam from α=1.0 can't bridge this gap. Capacity probe proves that with correct α init, relL2=1.83e-6 immediately.
- **Fix**: Added `_init_alphas_from_target` in trainer.py — sets α = target_freq/base_freq for each leaf before finetune. Smoke tested: correct (1.047, 2.225). Deployed to arnold. os s2-s4 will pick up fix.
- **Fix scope**: oracle arms use oracle_freqs from config; estimated arms use FFT-estimated freqs.
- Current batch (oh-s4, os-s0, os-s1) still running old code (~ep400/1000); will produce failure data, then os s2-s4 use fixed code.
- mf-additive-estimated-prior remains queued — unblock after oracle_soft passes with fix.

[Run Sync] 2026-06-28T02:30 coder-1 — synced 3 new oracle_hard result.json (s1,s2,s3) from mf-additive:
- oh-s1 relL2=0.733, oh-s2 relL2=0.735, oh-s3 relL2=0.917. All FAIL. oracle_hard 4/5 done (s4 running ep421).
- Finetune log confirms: 5 candidates all stuck at relL2 0.73-1.09 after 10000 Adam steps.
- mf-additive 9/15 done (std×5 + oh×4), os s0-s1 running. oracle_hard confirms: search finds correct freqs (first_hits at ep0) but continuous optimization fails.

[Run Sync] 2026-06-28T00:15 coder-1 — synced+verified 4 oracle_soft result.json from g10/g100 seedext. **CRITICAL POSITIVE**: oracle_soft dramatically superior to standard on nonlinear PDEs:
- **g10 (gamma=10)**: os-s1 relL2=4.63e-4, os-s2 relL2=**1.29e-6** (machine precision!). vs std median=1.40e-2 → 30-10000× improvement.
- **g100 (gamma=100)**: os-s1 relL2=**1.26e-6**, os-s2 relL2=**1.12e-6** (BOTH machine precision!). vs std median=4.12e-3 → 1000-3000× improvement.
- **g100 ordering resolved**: iter12 n=1 showed estimated>standard>oracle_hard (oracle worst). Now with oracle_soft, both seeds hit machine precision — oracle_soft is actually the best arm, not oracle_hard. The surprising iter12 ordering was an oracle_hard artifact.
- All oracle_soft use `gate_applicable=false`/`uses_true_sol_for_gate=not_applicable` (correct metadata for no-gate arm).
- g10: 6/12 done (std×4 + os×2), os s3-s4 just started GPU6-7, ves s1-s4 queued.
- g100: 6/12 done (std×4 + os×2), os s3-s4 just started GPU8-9, ves s1-s4 queued.
- mf-additive-grammar-oracle: 6/15 done (std×5 + oh-s0), oh s1-s3 search phase (ep850-900/1000, best_err~7e6 stuck), oracle_soft queued after oh finishes. Finetune non-convergence remains bottleneck.
- mf-additive-estimated-prior: still BLOCKED (oracle best relL2=0.748 ≫ 1e-5). oracle_soft not yet started — may help search but finetune bottleneck unclear.
GPU cost: incremental +$24.32 (7 GPU × 3.58h × $0.97/RTX8000). STATE total → $1277.14.

[Run Done] 2026-06-27T20:40 coder-1 — gate2-f12-trace-rerun → collected (2/2). Both decoy12 seeds have complete programmatic validation traces with all_results, best_rel_residual, passed, and fallback. s0: relL2=1.11e-6 (machine precision despite gate REJECT), resid=0.995 >> 0.5, uniform fallback search succeeded. s1: relL2=0.006, resid=0.994, gate REJECT correct. Both uses_true_sol_for_gate=false, gate_applicable=true. Evidence gap AUD-MAJOR-001 repaired — far-decoy f12 gate rejection now programmatic, not inferred. GPU5 freed. GPU cost +$6.38 (6.58 GPU-h × $0.97/RTX8000).

[Run Sync] 2026-06-27T20:40 coder-1 — synced+verified 8 new result.json from 3 running experiments:
- mf-additive-grammar-oracle: standard s3 relL2=0.748, s4 relL2=0.876 — both found dual frequencies (4π=12.57 + 17π=53.41) at epoch 0-2 via additive grammar, but finetune non-convergent. oracle_hard s0 relL2=0.762 (found both freqs early, oracle_hard no advantage over standard). Standard 5/5 complete — additive grammar SOLVES search problem (dual freq early finding), but NEW bottleneck: finetune continuous optimization fails to reach machine precision (best=0.748 vs 1e-5 target).
- gate3-nonlinear-g10-seedext: standard s3 relL2=0.0174, s4 relL2=0.00682. Standard arm 4/4 complete (all non-machine, expected without prior). oracle_soft s1-s2 running on GPU6-7 (train.jsonl exists, no result.json yet).
- gate3-nonlinear-g100-seedext: standard s3 relL2=0.00442, s4 relL2=0.215 (outlier). Standard arm 4/4 complete. oracle_soft s1-s2 running on GPU8-9 (train.jsonl exists, no result.json yet).
- f12 decoy2 s0/s1 verified with full validation traces (see [Run Done] above).
All 3 experiments still running on arnold (3 screens, 7 GPU processes). mf-additive-estimated-prior remains queued but BLOCKED — oracle pass criterion (relL2≤1e-5 ≥3/5 seeds) not met (current best relL2=0.748). GPU cost +$44.44 incremental since last sync (see STATE frontmatter update to $1252.82).

[Run Sync] 2026-06-27T20:00 coder-1 — synced+verified 8 result.json from 4 running experiments:
- mf-additive-grammar-oracle: standard s0 relL2=0.854, s1 relL2=0.783, s2 relL2=0.840 — all FAIL (expected, no oracle prior). Additive grammar active, schema `gate_applicable=false`/`uses_true_sol_for_gate=not_applicable` confirmed.
- gate3-nonlinear-g10-seedext: standard s1 relL2=1.08e-2, s2 relL2=1.72e-2 — standard arm on nonlinear, non-machine expected.
- gate3-nonlinear-g100-seedext: standard s1 relL2=2.04e-3, s2 relL2=3.81e-3 — similar pattern.
- gate2-f12-trace-rerun: validated_estimated_soft-helmholtz_7pi-s0 relL2=1.11e-6 (machine precision!), full validation trace present (`all_results`, `best_rel_residual`, `passed=false`), uses_true_sol_for_gate=false. This is the clean PDE calibration run; f12 decoy variant still running.
All 4 experiments still running on arnold (4 screens, 8 GPU processes). GPU cost added for elapsed time (details below).

[Run Crash diagnostic] 2026-06-27T20:00 coder-1 — mf-additive-grammar-oracle oracle_hard s0: search stuck at best_err=7.32e6 since epoch 100 (now epoch 400+), freq_entropy 2.67→1.54 (collapsing to wrong action). Standard arm search error also 7.1e6 — oracle prior not differentiating. Suspected root cause: alpha optimization too slow during search → correct structural configuration with alpha=1.0 gives huge residual for 17π component (sin(24x) vs target sin(53.4x)) → controller receives negative signal for the correct action → converges to wrong configuration. Capacity probe proves tree CAN express target (relL2=1.83e-6 with manual fixed actions), so issue is search dynamics, not grammar capacity. Monitoring continues — oracle_soft may succeed where oracle_hard fails.

## [Iter 13 Start] 2026-06-27 14:50 — scientist: audit BLOCKER response + next coder plan
- Scenario B after `[Audit]` iter12 verdict=BLOCKER. Read required refinery mindset, factory manuals/templates, topic/landscape/idea/proposal, STATE/LESSONS/lit-feed/data manifest, latest audit, and latest result evidence.
- Lit-feed inbox already `unprocessed: 0`.
- Audit response:
  1. Accepted `AUD-BLOCKER-001`: mf-depth3-capacity-oracle 15/15 all fail (relL2 0.855-0.974). This is the load-bearing gap; no Gate 2/3 evidence substitutes for the multi-frequency anchor under §5.
  2. Accepted `AUD-CRIT-001`: gate3 nonlinear gamma=10/100 is n=1 per arm. It is a strong trigger signal, not claim-bearing.
  3. Accepted `AUD-CRIT-002`: rewrote STATE §4 with iter12 evidence (Gate 2 no-true-sol, decoy/width, nonlinear, nonuniform, retention, multifreq fail).
  4. Accepted `AUD-MAJOR-001`: f12-s0/s1 only have reconstructed result.json and no validation trace; they are no longer treated as programmatic gate evidence.
  5. Accepted iter10 carryover: decoy story is now "far-frequency gate blocks; near-target acceptance is benign", not "near decoy rejected"; resume first_hit is not primary evidence.
- Server selection: used server-health. Arnold is primary (CPU 4%, GPU3-9 mostly idle, 1.6T free); 1202b fallback. Arnold pitfall carried into A2/A4: long screen runs must use direct `> file 2>&1`, not `| tee`.
- New A1/A3 plan:
  - Group A P0 `can_split=false`: `mf-capacity-explicit-action-probe` → `mf-additive-grammar-oracle` → `mf-additive-estimated-prior`.
  - Group B P0 `can_split=true`: `gate3-nonlinear-g10-seedext` and `gate3-nonlinear-g100-seedext`, seeds 1-4 for standard/oracle_soft/validated_estimated_soft.
  - Group C P0 `can_split=true`: `gate2-f12-trace-rerun` and `result-schema-gate-semantics`.
- STATE frontmatter set to `iteration: 13`, `phase: coding_and_running`; STATE line count after rewrite: 373.

## [Audit] 2026-06-27 14:00 — verdict=BLOCKER (iter 12)
- Report: audits/audit_iter12_20260627_1400.md
- Load-bearing issues: [BLOCKER-001] multi-frequency anchor capacity 15/15 ALL FAIL; [CRIT-001] gate3-nonlinear n=1 no statistical confidence; [CRIT-002] §4 results stale; [CRIT-003] STATE.md 566→264 lines trimmed
- Required scientist response: (1) diagnose+fix tree grammar for multifreq, (2) rerun gate3-nonlinear with 3+ seeds, (3) rewrite §4 with all new evidence, (4) respond to iter10 carryover items

[Run Sync] 2026-06-27T04:15 coder-1: gate2-residval-decoy-greywidth — synced+verified width batch 1 6/6 (cap0125×h7pi×3 + cap050×h7pi×3). All uses_true_sol_for_gate=false, gate PASS all (clean PDE, correct behavior).
- **cap0125** (n_sel=1/8): 3/3 PASS → selected=[21.0 (≈7π)]. relL2=2.82e-6/1.33e-6/3.35e-6. Narrow width precisely locks target base only.
- **cap050** (n_sel=4/8): 3/3 PASS → selected=[21.0, 24.0, 18.0, 15.0]. relL2=1.21e-5/1.26e-6/4.96e-5. Wider width injects 4 bases but FEX still converges to target via soft prior.
- **Key**: Width cap controls prior injection breadth without breaking gate. All 6 gate passes clean (no decoy). Batch 2 (h8pi, target 8π=25.13 outside base set) running on 6 GPU ep~130/1000. Batch 3 (cap025 + cd2d) queued. GPU cost +$23.28 (1121.37).

[Run Sync] 2026-06-27T00:15 coder-1: gate2-residval-decoy-greywidth — synced+verified 6 new decoy result.json (f21×3 + f24×3). All 9/9 decoys now have local evidence with uses_true_sol_for_gate=false.
- **f21** (≈7π=21.99 boundary): 3/3 PASS. FFT estimates ~21.43, gate ACCEPTS (best_rel_residual ~0.39 < 0.5 threshold). Controller finds machine-precision solutions (relL2=1.18e-5/9.04e-7/1.35e-6). This is expected boundary behavior — 21 and 7π=21.99 are too close for the FFT base set {3,6,...,24} to separate, and the residual fit with base 21 works because 21≈22=7π.
- **f24** (far decoy): 1/3 PASS, 2/3 marginal. FFT estimates ~23.90, gate correctly REJECTS (best_rel_residual ~0.96 >> 0.5) → uniform fallback. s0 recovers to machine precision (1.11e-6) via uniform search. s1 (5.99e-3) and s2 (1.43e-3) fail to converge without prior — uniform search is unreliable. Gate behavior correct in all 3 seeds.
- **Decoy tally**: f12 3/3 gate REJECT (correct), f21 3/3 gate ACCEPT (boundary — 21≈7π), f24 3/3 gate REJECT (correct). Gate distinguishes far decoys from clean signal; boundary case at f21 is a fundamental FFT resolution limit, not a gate bug.
- **Width sweep**: 18 conditions auto-started 22:42 EDT. Batch 1 (cap=0.125/0.50 × h7pi s0-2, 6 GPUs) running ~1.5h into search. Screen fexsp-g2r-v3-0626 alive. Estimated ~15-21h total wall time for all 3 batches.
- GPU cost +$26.36 (decoy 18.2 GPU-h + width partial 9 GPU-h × $0.97/RTX8000).

[Run Done] 2026-06-26T13:25 coder-1: gate3-nonuniform-smoke → collected (9/9). Synced+verified final 2 result.json: interp_fft s1 relL2=1.29e-6 PASS, dict_ls s0 relL2=1.31e-6 PASS (both machine precision, uses_true_sol_for_gate=false). Final tally: regular 2/3 PASS + interp_fft 3/3 PASS + dict_ls 3/3 PASS = 8/9 PASS (regular s2 1.23e-3 FAIL). Success criterion met: nonuniform estimator relL2≤1e-4 on h7pi with target freq selected. See results/gate3-nonuniform-smoke/*/result.json. GPU cost +$6.43 incremental.

[Run Sync] 2026-06-26T13:25 coder-1: rsynced gate3-nonuniform-smoke interp_fft s1 (18,157 bytes, arnold 13:01 EDT) + dict_ls s0 (18,520 bytes, arnold 12:47 EDT). Both verified: real numbers, machine precision, uses_true_sol_for_gate=false, validation passed. All 9 result.json now local.

[Run Done] 2026-06-26T11:30 coder-1: gate2-residval-clean-rerun → collected (14/14). cd2d s1+s2 finetune interrupted by combined script switching to Phase 2 (decoy). result.json reconstructed from train.jsonl. s1 relL2=1.36e-6 (rank0, 3 candidates), s2 relL2=1.25e-6 (rank1, 2 candidates). Both machine precision, uses_true_sol_for_gate=false. h7pi 8/8 + h8pi 3/3 + cd2d 3/3 = 14/14. h7+h8 combined 10/11 ≥ 8/11 criterion. See results/gate2-residval-clean-rerun/*/result.json. GPU cost included in running total.

[Run Sync] 2026-06-26T11:30 coder-1: rsynced cd2d s1+s2 train.jsonl + train.log + checkpoint.pt from arnold. Finetune evidence preserved in train.jsonl despite combined script kill. See results/gate2-residval-clean-rerun/validated_estimated_soft-convdiff_7pi_2d-s{1,2}/.

[Run Sync] 2026-06-26T06:00 coder-1: synced+verified 2 new gate3-nonuniform interp_fft result.json. s0 relL2=1.12e-6 PASS, s2 relL2=1.32e-6 PASS (both machine precision, uses_true_sol_for_gate=false). interp_fft 3/3 PASS complete. Screen fexsp-g3r-0626-0023 exited (GPU8-9 free). nonuniform 7/9 collected. dict_ls s1+s2 running GPU0+2 (ep~700, ~2h). gate2 clean 12/14: cd2d s1+s2 GPU4-5 search ep~490 (TFP=0.98, best_err=2.2e-4). decoy queued in combined script Phase 2. GPU cost added: $10.23.

[Run Done] 2026-06-26T03:20 coder-1: mf-depth3-capacity-oracle → collected. 15/15 complete (oracle_hard/oracle_soft/standard × s0-s4), all FAIL capacity (relL2 0.85-0.97). Tree depth-3 with 21 nodes/8 leaves cannot express the multifreq_4_17 anchor (sin(4πx)sin(4πy)+sin(17πx)sin(17πy)). Capacity confirmed broken — depth/grammar repair needed before retrying. mf-depth3-estimated-prior remains queued but effectively blocked.

[Run Sync] 2026-06-26T03:20 coder-1: synced+verified 6 new result.json (gate2 2 + mf3 3 + nonuniform 1). gate2 h8pi-s2 relL2=9.26e-7 PASS (no-true-sol), cd2d-s0 relL2=1.25e-6 PASS (no-true-sol) → gate2 clean 12/14. mf3 s4 oh/os/std relL2=0.97/0.85/0.97 all FAIL capacity (expected) → mf3 15/15 complete, marked collected. gate3 nonuniform s2 (regular) relL2=1.23e-3 FAIL (gate passed but search didn't converge). **Fix**: dict_ls s1+s2 failed to start in relaunch script because `mkdir -p` was after `>` redirect causing "No such file or directory". Manually created dirs and launched dict_ls s1+s2 on idle GPU0+GPU2 in screen `fexsp-g3d-0626-dictls`. Running: gate2 cd2d s1 GPU4 + s2 GPU5 (~121ep search), gate3 interp_fft s0 GPU8 + s2 GPU9 (~690ep search), gate3 dict_ls s1 GPU0 + s2 GPU2 (just started). gate2 decoy still queued after clean finishes. mf3 screens died naturally (scripts completed). GPU0-2 freed by mf3 completion. GPU cost added: $25.51.

[Run Crash] 2026-06-26T00:22 coder-1: gate2-residval-clean-rerun + gate3-nonuniform-smoke — screen PTY buffer full → tee pipe blocked → all child processes (h8pi s2, cd2d s0, nu_interp_fft s2, dict_ls s1-s2) FUTEX deadlocked on stdout write. Root cause: combined launch scripts used `| tee` and screen PTY buffer filled after ~25h of accumulated output. Fix: killed old screens, relaunched with direct `> file 2>&1` redirection (no tee). All processes resumed from checkpoint successfully (h8pi s2 ep900, cd2d s0 ep1000, gate3 s2 ep1000). New screens: fexsp-g2r-0626-0023 (GPU4-5), fexsp-g3r-0626-0023 (GPU8-9). Crash cost: ~5 GPU-h wasted on stuck processes.

[Run Done] 2026-06-25T20:30 coder-1: retention-cd2d-trajectory 6/6 complete → collected. All 6 seeds (standard×3, oracle_soft×3) synced and verified. Cross-PDE retention mechanism confirmed: early TFP ordering matches h7pi (oracle_soft >> standard). os-s0 relL2=1.24e-6, os-s1=7.69e-5, os-s2=1.55e-4; std-s0=1.24e-6, std-s1=1.24e-6, std-s2=2.58e-3. 3 crashes resolved (checkpoint FUTEX deadlock fixed, timeout truncation fixed, auto-resume verified). Success criterion: early TFP ordering matches h7pi and explains relL2 quality gap. Met.

[Run Sync] 2026-06-25T20:30 coder-1: synced+verified 7 new result.json:
- retention os-s2: relL2=1.55e-4 PASS (6/6 complete, cd2d retention confirmed)
- mf3 os-s2 recovery: relL2=0.897 FAIL (capacity broken, expected)
- mf3 s3 oh/os/std: relL2=0.915/0.961/0.970 all FAIL (12/15, s4 running)
- nonuniform s0 dict_ls: relL2=1.11e-6 PASS, uses_true_sol_for_gate=false ✓
- nonuniform s1 interp_fft: relL2=1.29e-6 PASS, uses_true_sol_for_gate=false ✓
All verified with real numbers. nonuniform first batch establishes deployable residual gate on nonuniform collocation points — key positive.

[Run Sync] 2026-06-25T19:30 coder-1: synced gate2 h8pi s0 (relL2=9.24e-7 PASS ✓), gate2 h8pi s1 (relL2=1.33e-3, gate passed but search error=951), retention std-s2 (relL2=2.58e-3, standard arm). All verified locally.

<!--
experiment-log.md: 跨 branch 持久的完整项目日志
新条目 **prepend** (时间倒序, 最新在顶部).

[Run Sync] 06-25 18:30 coder-1: gate3-nonlinear-rhs-mismatch 9/9 complete. os-g100 relL2=2.17e-3 (oracle fails at gamma=100), **ves-g100 relL2=1.24e-6** (estimated prior beats oracle at extreme nonlinearity — key positive). uses_true_sol_for_gate=false. All result.json + train.jsonl synced locally.

[Run Crash] 06-25 18:30 coder-1: **RESOLVED** checkpoint FUTEX deadlock. Root cause: trainer.py line 221 `torch.load(map_location=device)` caused CUDA autograd deadlock when resuming checkpoint across different GPU context. Fix: `map_location='cpu'` + manual `.to(device)` for pool candidates. Also added `log_file.flush()` after finetune writes. Deployed recovery screen fexsp-rec3-0625-1827 (retention std-s2 GPU7 + mf3 os-s2 queued). Verified: ckpt loaded successfully, finetune started.

[Run Sync] 06-25 18:30 coder-1: gate3-nonuniform-smoke auto-started 17:42 EDT from combined script. GPU8-9, interp_fft × helmholtz_7pi s0-s1.
-->

## [Run Sync] 2026-06-25T16:20 — coder-1: 3 new result.json (retention os-s1, mf3 oh-s2 std-s2) synced+verified; 2 crashes fixed

- **retention-cd2d-trajectory os-s1**: relL2=7.69e-5 PASS. 10 candidates finetuned, all complete. TFP trajectory: ep9=0.341, ep499=0.516, ep999=0.483. Oracle_soft mechanism confirmed on cd2d. 4/6 retention conditions done (std-s0/s1, os-s0/s1).
- **mf-depth3-capacity-oracle oh-s2**: relL2=0.898 FAIL (capacity). 4.4h. As expected — depth-3 tree cannot express multifreq_4_17.
- **mf-depth3-capacity-oracle std-s2**: relL2=0.933 FAIL (capacity). 4.3h. Confirms s0/s1 pattern. 9/15 mf3 done (s0:3, s1:3→synced previously, s2:3→oh+std synced now, os restarted). s3 all 3 arms just started (GPU0-2). s4 queued.
- **All 3 new result.json verified**: actual numbers, non-zero, finetune complete, within expected ranges.

## [Run Crash] 2026-06-25T16:30 — retention std-s2 + mf3 os-s2: checkpoint resume FUTEX deadlock, 3 restart attempts failed

- **Root cause narrowed**: checkpoint resume (search-complete → finetune) triggers a CUDA futex deadlock at process startup. Process enters 99.4% CPU spin-loop with no output. strace confirms tight FUTEX_WAKE/FUTEX_WAIT cycle — PyTorch thread synchronization deadlock.
- **Excluded hypotheses**: (a) GPU contention — reproduced on solo GPU9; (b) device map — same hang on GPU3/GPU7/GPU9; (c) checkpoint corruption — same checkpoint worked for auto-resume on std-s1.
- **Suspected cause**: Checkpoint saved on different GPU (GPU3, different process group) creates incompatible CUDA context state. When resumed on a new process, PyTorch's autograd/CUDA initialization deadlocks.
- **Attempted fixes**: (1) Kill+restart from checkpoint on GPU7 (2 processes → deadlock), (2) Kill+restart solo on GPU9 (still deadlock).
- **Workaround**: The checkpoint.pt file exists with search-complete state. Next coder should: (a) modify trainer.py to skip checkpoint loading and directly run `_finetune()` on the saved candidates, OR (b) use the train.jsonl data to manually score candidates and reconstruct result.json.
- **retention std-s2**: search done (ep=999, best_err=0.71, 1070 lines in train.jsonl), 0 finetune. Checkpoint at /scr1/scratch/sun1245/fex-synchro-prior/results/retention-cd2d-trajectory/standard-convdiff_7pi_2d-s2/checkpoint.pt.
- **mf3 os-s2**: search done (ep=999, best_err=5.85M, 1000 lines), 0 finetune. Launcher moved to s3. Checkpoint at /scr1/scratch/sun1245/fex-synchro-prior/results/mf-depth3-capacity-oracle/oracle_soft-multifreq_4_17-s2/checkpoint.pt.
- **Crash count**: retention → 2, mf3 → 1.

## [Run Sync] 2026-06-25T14:20 — coder-1: gate2 h7pi 8/8 complete, gate3 g100 OS+STD synced, retention std-s1 verified

- **gate2-residval-clean-rerun h7pi s6/s7**: both PASS with no-true-sol gate. s6: relL2=1.16e-6, s7: relL2=1.10e-6, uses_true_sol_for_gate=false, selected_bases=[21,24]. H7PI 8/8 COMPLETE (all s0-s7 converge to ~1e-6).
  - h8pi s0-s1 now running on GPU4-5 (started 13:45 EDT). h8pi s2 + cd2d s0-s2 queued.
- **gate3-nonlinear-rhs-mismatch g100**:
  - oracle_soft: relL2=0.00217 (fails 1e-4). TFP=0.998 locks frequency perfectly but search error=206 — extreme gamma. Even perfect frequency knowledge insufficient.
  - standard: relL2=5.06e-5 (passes 1e-4!). Standard exploration accidentally found a better tree for gamma=100. n=1 seed — likely luck.
  - ves g100-s0: running on GPU8 (8+ min). 8/9 nonlinear complete.
  - Gamma crossover: g1 all pass; g10 std FAIL (0.479) / os+ves PASS (~1e-6); g100 std 5e-5 / os 0.002 / ves pending.
- **retention-cd2d-trajectory std-s1**: relL2=1.24e-6 PASS. Auto-resume from checkpoint.pt — search was complete, finetune produced 10 candidates, rank 0 best. Standard 2/3 done (s2 running), OS 1/3 (s1 running).
- All 5 result.json synced+verified (gate2 s6/s7, gate3 os+std g100, retention std-s1). Numbers real, non-zero, within expected ranges.
- **Progress snapshot**: gate2 8/14 (h7pi✓ → h8pi running), gate3 8/9 (ves g100 running), mf3 6/15 (s2 running→all expect FAIL), retention 3/6 (os-s1+std-s2 running).
- GPU: GPU0-2 8-15% (mf3 s2), GPU3 84% (retention os-s1+std-s2), GPU4-5 6-7% (gate2 h8pi), GPU6-7 idle, GPU8-9 10% (gate3 ves g100 + sibling fex-sc).

## [Run Sync] 2026-06-25T12:22 — coder-1: retention os-s0 verified, standard seeds restarted

- **retention-cd2d-trajectory oracle_soft s0**: relL2=1.24e-6 PASS. TFP trajectory: ep9=0.341, ep99=0.373, ep199=0.533, ep499=0.503, ep999=0.502. First hit epoch 0 (freq=21.99).
  - **Mechanism confirmed**: oracle_soft TFP starts 4.3× higher than standard at ep9 (0.341 vs 0.081), retains advantage throughout. cd2d retention trajectory replicates h7pi pattern — retention mechanism is PDE-independent (Helmholtz + convdiff2d).
  - 5/10 candidates achieve relL2 < 1e-4. Best candidate: rank 7 (search_error=0.0102, relL2=1.24e-6).
- **Standard seeds restarted** via screen `fexsp-ret-std-restart-0625` on GPU3 (sharing with os-s1, 228+230 MiB, no conflict):
  - std-s1: auto-resume from checkpoint.pt (search done at ep999, best_err=0.138). Skipping to finetune (~17 min, 10 candidates × 10000 iters). ETA result.json: ~12:38 EDT.
  - std-s2: full restart after s1 completes.
  - Root cause note: original launch script used TIMEOUT=10800 (3h) which is too short for search+finetune. New deployment uses --timeout 21600.
- **Arnold SSH outage**: External SSH port 22 down since ~11:50 EDT (Connection Refused). Ping OK, WireGuard up. Routing all connections through majda proxy (ssh -J majda arnold). Experiments unaffected (screen sessions intact).
- **Current progress snapshot**: gate2 s6 ep800/s7 ep850 (search ~30min to completion), gate3 g100 os ep699/std ep719 (~1h), mf3 s2 all arms ep500-636 (~1.5h), retention os-s1 ep~280 (~2h to search completion).
- gate2 6/14, gate3 6/9, mf3 6/15, retention 2/6 (+os-s0). All running experiments alive and progressing.

## [Run Sync] 2026-06-25T10:40 — coder-1: 7 new result.json synced and verified; gate3 nonlinear key positive

- **gate2-residval-clean-rerun s4/s5**: both PASS. s4: relL2=1.16e-6, s5: relL2=1.08e-6, uses_true_sol_for_gate=false. H7pi 6/8 complete.
- **gate3-nonlinear-rhs-mismatch g10**: oracle_soft relL2=1.12e-6 PASS, validated_estimated_soft relL2=1.12e-6 PASS. Standard relL2=0.479 FAILED (previously synced). **Key positive: both oracle and estimated prior overcome gamma=10 nonlinearity where standard fails. Prior mechanism helps on harder PDE — aligns with §5 scope-up directive.**
- **mf-depth3-capacity-oracle s1**: all 3 arms FAIL capacity (oh=0.97, os=0.94, std=0.97). Confirms s0: tree depth-3 cannot express multifreq_4_17. 6/6 conditions fail.
- **retention-cd2d-trajectory**: no new result.json. os-s0 search complete (ep=999, best_err=3.5e-4), entering finetune. std-s1 checkpoint available for resume.
- All result.json verified: actual numbers, non-zero, within expected range. Manifests updated with completed conditions, running conditions, and pending restarts.
- GPU contention noted: GPU0 (mf3 oh-s2) and GPU9 (gate3 os-g100) shared with fex-search-complexity sibling workspace since 09:59.
- gate2 6/14, gate3 6/9, mf3 6/15, retention 1/6. No phase changes — all running continue, all queued remain queued.

## [Run Crash] 2026-06-25T08:15 — retention-cd2d-trajectory: orphan oracle_soft s2 killed, GPU3 collision fixed, std s1/s2 need restart

- GPU3 had TWO oracle_soft processes competing: PID 552722 (seed=2, no screen, launched 07:30) and PID 555430 (seed=0, in screen fexsp-ret-os-restart-0625, launched 07:36).
- Killed PID 552722 (orphan, wasted ~220 epochs). Screen now runs seed=0 solo.
- Standard s1: has completed search (1000 epochs, checkpoint.pt exists), finetune was killed by original TIMEOUT=10800. Auto-resume will skip directly to finetune on rerun.
- Standard s2: killed at epoch 370. Needs full restart.
- After oracle_soft s0 finishes (~5h from 08:00), need to launch s1 and s2 sequentially.
- Also need to restart standard s1 and s2 (can use GPU3 after oracle_soft done, or different GPU).
- Stray directories s99 exist (0 lines each) — will clean up after all seeds complete.

## [Run Sync] 2026-06-25T08:00 — coder-1: verified all previously-synced result.json, no new results

- All 12 result.json verified locally: gate2 s0-3 (4), gate3 g1/g10 (4), mf-depth3 s0 (3), retention std-s0 (1).
- gate2 s2: relL2=1.06e-6, s3: 1.17e-6 — both PASS, uses_true_sol_for_gate=false.
- gate3 std-g10: relL2=0.479 (FAILED, expected for gamma=10 without prior).
- gate3 ves-g1: relL2=1.12e-6, uses_true_sol_for_gate=false (PASS).
- All 4 "new" result.json already synced by coder-4. No additional result.json appeared since.
- Running experiments progressing: gate2 s4/s5 ~740ep, gate3 os-g10/ves-g10 ~700ep, mf-depth3 s1 ~920-999ep, retention os-s0 ~200ep.
- GPU contention noted: fex-search-complexity processes on GPU0/GPU9 share with mf-depth3/gate3.

## [Run Crash] 2026-06-25T07:30 — retention-cd2d-trajectory: TIMEOUT killed finetune for s1, oracle_soft seeds lost

- Root cause: launch script `scripts/launch_retention_cd2d.sh` used `TIMEOUT=10800` (3h). Depth-2 cd2d search takes ~3h (1000 ep × ~10.5s), finetune takes ~2h more. timeout wrapper killed Python during finetune for s1 (search complete, no result.json).
- s2 was running (ep370) when coder mistakenly used SIGSTOP on timeout wrapper → stopped child process group. Process killed during recovery attempt.
- oracle_soft s0-s2 all lost. Restarted on GPU3 (screen `fexsp-ret-os-0625-0726`, no external timeout wrapper, `--timeout 21600` internal).
- Prevention: retention launch script should use `TIMEOUT=21600` (6h) or no external timeout. Background watcher on arnold (`/tmp/kill_retention_timeouts_v2.sh`) auto-kills timeout wrappers for retention to prevent recurrence.

## [Run Sync] 2026-06-25T07:00 — gate2 s2/s3 + gate3 std-g10/ves-g1 verified

- **gate2-residval-clean-rerun h7pi-s2**: relL2=1.06e-6, uses_true_sol_for_gate=false, first_hit ep0 ✓
- **gate2-residval-clean-rerun h7pi-s3**: relL2=1.17e-6, uses_true_sol_for_gate=false, first_hit ep0 ✓
- **gate3-nonlinear-rhs-mismatch standard g10-s0**: relL2=4.79e-1 — FAILED (gamma=10 too hard for standard without frequency prior). Validates reviewer concern and shows oracle/estimated advantage needed.
- **gate3-nonlinear-rhs-mismatch validated_estimated_soft g1-s0**: relL2=1.12e-6, uses_true_sol_for_gate=false ✓. Estimated prior works on nonlinear PDE at gamma=1.
- All result.json synced to local and verified. gate2: 4/14 conditions done (h7pi s0-3 all PASS). gate3: 4/9 done.

## [Run Sync] 2026-06-25T04:15 — mf-depth3-capacity-oracle s0 FAILED capacity

- Depth-3 tree (21 nodes, 8 leaves) on multifreq_4_17 (sin(4πx)sin(4πy)+sin(17πx)sin(17πy)):
  - oracle_hard relL2=0.969, oracle_soft relL2=0.936, standard relL2=0.905
  - All ≈1.0 → tree output ≈ 0, cannot express the solution. Criterion was <1e-5.
  - Search errors stalled at 5.7-6.7 million; finetune could not recover.
  - Paradox: standard has lowest relL2. TFP ordering correct (oracle_soft 0.55 > standard 0.26) — prior mechanism works, tree grammar is the bottleneck.
  - First hits at epoch 0 for both frequency components — search found the targets, but tree can't compose them.
- **Verdict**: capacity check FAILED. mf-depth3-estimated-prior MUST NOT be deployed. Next step: depth/grammar fix per plan.
- s1 seeds auto-launched after s0 finished; s1 search in progress but expected to fail similarly.

## [Run Sync] 2026-06-25T02:30 — gate2/gate3/retention first seeds complete

- **gate2-residval-clean-rerun h7pi-s0**: relL2=1.30e-6, `uses_true_sol_for_gate=false` ✓, selected_bases=[21,24]
- **gate2-residval-clean-rerun h7pi-s1**: relL2=1.29e-6, `uses_true_sol_for_gate=false` ✓, selected_bases=[21,24]
  - Both machine precision. Residual validation works. 2/14 conditions done.
- **gate3-nonlinear-rhs-mismatch standard g1-s0**: relL2=1.16e-6 (gamma=1, no prior)
- **gate3-nonlinear-rhs-mismatch oracle_soft g1-s0**: relL2=8.26e-7 (gamma=1, oracle)
  - Both machine precision on gamma=1. Oracle_soft slightly better. 2/9 done.
  - Next batch: validated_estimated_soft g1 + standard g10 running.
- **retention-cd2d-trajectory standard-s0**: relL2=6.65e-4, TFP trajectory replicates h7pi pattern:
  - ep9 TFP=0.08 (near uniform), ep200 TFP=0.18, ep500 TFP=0.47, ep999 TFP=0.50
  - First_hit at epoch 0 but slow retention — matches h7pi mechanism evidence.
  - 1/6 conditions done.

## 2026-06-25 — coder: gate2/gate3 four runs deployed

- **Code**: nonuniform estimator (interp_fft + dict_ls) in estimator.py; --nonuniform CLI flags; pytest 18/18; arnold GPU smoke PASS.
- **gate2-residval-clean-rerun** (GPU4-5): 14 conditions; validated_estimated_soft, residual validation, width-cap 0.25.
- **gate2-residval-decoy-greywidth** (GPU4-5, after clean): 27 conditions; width sweep {0.125,0.50} + decoy {12,21,24}.
- **gate3-nonlinear-rhs-mismatch** (GPU8-9): 9 conditions; gamma {1,10,100} × 3 arms.
- **gate3-nonuniform-smoke** (GPU8-9, after nonlinear): 9 conditions; h7pi/cd2d × interp_fft/dict_ls.
- Screens: fex-sp-gate2-0625-223949, fex-sp-gate3-0625-223951.

## [Run Sync] 2026-06-25 — mf-depth3-capacity-oracle deployed + retention-cd2d-trajectory deployed

- **mf-depth3-capacity-oracle**: 15 conditions (oracle_hard/oracle_soft/standard × seeds 0-4) deployed on arnold GPU0-2
  - PDE: multifreq_4_17 (helmholtz_multifreq_sine, 2 components: sin(4πx)sin(4πy) + sin(17πx)sin(17πy))
  - Tree depth: 3 (21 nodes, 8 leaves)
  - Screens: fexsp-mf3-{oh-g0,os-g1,std-g2}-0624
  - Bring-up ep49 TFP: oracle_soft 0.426 > oracle_hard 0.359 > standard 0.159 ✓
  - Bring-up best_err all ~7e6 (high, expected for depth-3 early epochs)
- **retention-cd2d-trajectory**: 6 conditions (standard/oracle_soft × seeds 0-2) deployed on arnold GPU3
  - PDE: convdiff_7pi_2d (2D convection-diffusion, freq=7π)
  - Screen: fexsp-ret-cd2d-g3-0624
  - Bring-up ep19 standard TFP=0.081 (matches h7pi standard pattern)
- **mf-depth3-estimated-prior**: code ready (launch script written), phase=queued, depends on capacity oracle passing
- Code commit: a0e82df (Group A foundations), synced to arnold

## [Run Done] 2026-06-25 — review2-code-foundations → collected

- Group A code foundations implemented (commit a0e82df):
  - estimator.py: PDE residual+BC validation replaces true_sol; metadata `uses_true_sol_for_gate=false`
  - model.py: recursive build_tree supports depth 3+; LearnableTree uses configured depth
  - pde.py: added nonlinear_reaction_sine (γ·u³) and helmholtz_multifreq_sine (multi-component)
  - run_gate0_arm.py: --tree-depth, --validation-mode, nonlinear/multifreq PDE cases
- pytest 15/15 pass; gate2 estimator smoke grid=64 PASS (h7pi, h8pi, convdiff2d clean; decoy rejected)
- Enables Groups B/C/D/E implementation

## [Version 2 Start] 2026-06-25 — scientist: reviewer response, no-true-sol validation + scope-up plan

- Scenario C after `[Review of Version 1]` score=4.8/not ready.
- Consumed lit-feed 5/5:
  - 2512.21319, 2606.12050, 2507.02227 → promoted to no-`true_sol` residual validation design in STATE A1/A2.
  - 2601.21669, 2605.28860 → promoted to cd2d retention trajectory and LESSONS action-level TFP guidance.
- §5 user directives now active: scope must go bigger/harder, not smaller. Therefore reviewer option "(a) limit to single-frequency" is rejected as a plan path; next route attacks expressive multi-frequency / nonlinear / nonuniform PDE instead.
- A0 reviewer response:
  - Accept Gate 2 true_sol leakage: old estimated-prior evidence marked measurement-invalid; Group A/B implements residual validation and reruns clean/decoy/greyzone/widthcap.
  - Accept multi-frequency anchor gap: Group A/C adds tree-depth/capacity support and reruns a multi-frequency anchor.
  - Accept Gate 3 gap: Group D runs nonlinear RHS mismatch and nonuniform smoke.
  - Accept retention trajectory gap: Group E reruns cd2d full TFP trajectory.
  - Correct widthcap count: balanced 3×3 matrix is 25/27 search conv, 23/27 machine precision.
- Server choice: used server-health. Arnold is primary (12h busy 2.3/10, CPU 6%) despite tight disk; checkpoints should stay remote/excluded from local sync. 1202b/1202c are fallbacks.
- STATE.md: iteration 11→12, phase→coding_and_running, route `v2-review-mustfix-scopeup`. lit-feed frontmatter set `unprocessed: 0`.

## [Review of Version 1] 2026-06-25 00:30 — score=4.8/10
- Verdict: not ready
- Primary concern: Gate 2 validation gate 使用 true_sol (estimator.py:179-265) 使得 "checked without leaking" 和 decoy rejection claims 失效; Gate 0 diagnostic 扎实 (Fisher p=2.96e-5) 但 scope 局限于 single-frequency manufactured PDE, 原 proposal 的 wide-separated anchor 未成功

## [Version 1 Finished] 2026-06-24 — scientist: iter10 结果全部整合, 送审

- Consumed iter10 audit (CRITICAL). Responded to all 8 findings, especially CRIT-001 claim reframing.
- CRIT-001 response: "far/near decoy rejected" → "远 freq 被阻挡; 近目标 contamination 无害 (6/6 mach, 36/36 target_in_contam=True)"。这不是 claim 弱化而是升级: method 鲁棒性来自 spectral locality 的几何性质，不依赖 gate 的检测能力。
- §4 整合完成:
  - Width-cap: 3 PDE × {0.125, 0.25, 0.50} → 25/27 conv, 24/27 mach. NOT fragile.
  - Grey-zone: decoy24-amp400 + decoy21-amp1500 → 6/6 mach despite ACCEPTED. Spectral locality.
  - cd2d-cap0125-s2 first_hit=ep900 diagnosed as resume logging artifact (ep9 TFP=0.9995).
  - Candidate audit: 12/14 SUCCESS confirms failure taxonomy.
- §4.2 Claims: 10 claims 全部 **强**; scope limitation 明确标注 (uniform-sampled manufactured PDE only).
- Reviewer must-fix 7/7 ✅。所有 claim-bearing evidence gaps 已解决。
- STATE.md: iteration 10→11, phase→needs_reviewer, 330 lines.
- 决策: 送审。证据足够支持当前 claims; reviewer 反馈将决定是否需要 Gate 3 (nonuniform/nonlinear)。
- §5 仍为空——第 12 次提醒用户。
- LESSONS.md: +4 条 (grey-zone 无害, resume first_hit, width 确认, Gate 1+2 完成)。

## [Audit] 2026-06-24 22:30 — verdict=CRITICAL (iter 10)
- Report: audits/audit_iter10_20260624_2230.md
- Load-bearing issues:
  - [AUD-CRIT-001] §4.3 "far/near decoy rejected" 与 grey-zone 数据矛盾——6/6 near-target decoys ACCEPTED by validation gate + machine precision. "Leakage-safe" claim architecture needs rebuild
  - [AUD-MAJOR-001] Widthcap 18/18 results not integrated into §4 (16/18 machine precision, width NOT fragile)
  - [AUD-MAJOR-002] Grey-zone 6/6 results not integrated into §4
  - [AUD-MAJOR-003] cd2d-cap0125-s2 first_hit=ep900 anomaly (resume logging artifact?)
  - [AUD-MAJOR-004] A0 iter9 status stale for width/validation items
- 独立数据核验: 18 widthcap result.json + 6 greyzone result.json + boundary sweep manifest 逐条核对, all match ✓
- Coder round 评价: 连续第二轮 100% clean execution — 6/6 collected, 无 crash, 1 timeout+resume 成功
- Required scientist response: (1) §4.3 claim reframing, (2) widthcap integration, (3) greyzone integration, (4) first_hit anomaly diagnosis, (5) A0 status update, (6) §5 第 12 次, (7) mid-freq boundary sweep decision, (8) STATE ≤400 lines after integration

## [Run Sync] 2026-06-24 21:30 — gate2-widthcap-cd2d cap0125-s2 synced

- rsync'd result.json + train.jsonl + train.log from arnold to local
- s2: best_rel_l2=1.35e-06 (machine precision), min_se=2.03e-4 (converged), val_res=0.283
- Resume from ep899 completed successfully (search 1069s + 10 candidate finetune)
- All 6/6 cd2d seeds now complete: cap0125 3/3 mach, cap050 3/3 mach

## [Run Done] 2026-06-24 21:30 — gate2-widthcap-cd2d → collected

- 6/6 seeds (cap0125 s0-2 + cap050 s0-2) all synced from arnold, all result.json present
- cap0125: 3/3 search conv, 3/3 machine precision (relL2 1.33-1.35e-6), ep9 TFP=0.999
- cap050: 3/3 search conv, 3/3 machine precision (relL2 1.24-1.35e-6), ep9 TFP=0.346
- Width_cap NOT fragile — both extremes (0.125, 0.50) work for cd2d, matching h7pi/h8pi pattern
- Evidence: manifest.json updated (sync_status=complete), result.json + train.jsonl per seed

## [Run Done] 2026-06-24 19:00 — gate2-widthcap-h7pi → collected

- 6/6 seeds (cap0125 s0-2 + cap050 s0-2) synced from arnold, all result.json present
- cap0125: 3/3 search conv, 2/3 machine precision (s1=3.26e-4 structure failure with correct freq), ep9 TFP=0.999
- cap050: 2/3 search conv, 2/3 machine precision (s2 search fail min_se=654), ep9 TFP=0.35
- Width_cap NOT fragile — both extremes (0.125, 0.50) work for h7pi
- GPU cost: ~$32 (6 seeds × ~5.5h × $0.97/h Quadro RTX 8000)
- Evidence: manifest.json in cap0125/ and cap050/, result.json + train.jsonl per seed

## [Run Done] 2026-06-24 19:00 — gate2-widthcap-h8pi → collected

- 6/6 seeds (cap0125 s0-2 + cap050 s0-2) synced from arnold, all result.json present
- cap0125: 3/3 search conv, 3/3 machine precision (relL2 9.20-9.24e-7), ep9 TFP=0.999
- cap050: 3/3 search conv, 3/3 machine precision (relL2 9.21-9.24e-7), ep9 TFP=0.47
- h8pi is PERFECTLY robust across all width_cap values — all 9 seeds (3 caps) machine precision
- GPU cost: ~$32 (6 seeds × ~5.5h × $0.97/h Quadro RTX 8000)
- Evidence: manifest.json in cap0125/ and cap050/, result.json + train.jsonl per seed

## [Run Sync] 2026-06-24 19:00 — gate2-widthcap-cd2d — 5/6 synced, s2 resume running

- 5/6 seeds synced from arnold (cap0125 s0-1 + cap050 s0-2), all result.json present
- cap0125: 2/2 search conv, 2/2 machine precision (1.33-1.34e-6), ep9 TFP=0.999
- cap050: 3/3 search conv, 3/3 machine precision (1.24-1.35e-6), ep9 TFP=0.35
- cap0125-s2: timeout at ep949/1000, resumed from checkpoint (ep899) on arnold GPU0, screen fex-sp-cd2d-resume-0624
- GPU cost: ~$27 (5 done + 1 resume), cumulative ~$33
- Evidence: manifest.json in cap0125/ and cap050/ (sync_status=partial)
- Run phase: queued → running (pending s2 completion)

## [Run Sync] 2026-06-24 00:55 — gate2-greyzone-decoy-realcontroller → collected

- 6/6 seeds synced from arnold: decoy21-amp1500 (s0-2) + decoy24-amp400 (s0-2)
- ALL machine precision: best_rel_l2 7.23e-07 to 1.17e-06
- ALL search converged, first_hit epoch0 eval 0-1
- ep9 TFP: decoy21≈0.53, decoy24≈0.50 — both show real controller retention signal
- Validation gate (threshold 0.50): ALL ACCEPTED → near-target decoys (freq 21/24) indistinguishable from clean prior
- Key finding: both grey-zone decoys (residual ≈0.40 at boundary sweep) converge to machine precision
- Leakage-safe claim narrowed to strong decoys only (res≈0.97 far-decoy family)
- GPU cost: +$3.0 (remaining 2 seeds finetune), cumulative greyzone $28.7
- Evidence: manifest.json updated, all 6 result.json + train.jsonl + train.log synced locally
- Run phase: needs_sync → collected

## [Run Done] 2026-06-23 17:15 — gate2-validation-boundary-sweep collected

- Local CPU, 0.3s wall-clock, $0 compute
- **7 grey-zone scenarios found** in [0.30, 0.70] out of 36 swept (3 decoy freqs × 12 amps)
- Transition is NOT sharp: residual sweeps continuously from ~0.04 to ~0.99
- Most dangerous grey-zone: freq=24/amp=400 (res=0.3959, ACCEPTED at 0.50, bases=[24,21] — decoy base primary)
- freq=21 decoys stay below 0.50 even at amp=2000 (res=0.42) — same-base decoy is harder to reject
- Threshold sensitivity: thr=0.35 blocks 3/7 grey-zone; thr=0.50 blocks 1/7; thr=0.70 blocks 0/7
- Evidence: `results/gate2-validation-boundary-sweep/manifest.json`

## [Run Done] 2026-06-23 17:15 — gate2-candidate-source-audit collected

- Local CPU, 14 Gate 2 runs audited across h7pi/h8pi/convdiff2d
- **12/14 SUCCESS, 1 STRUCTURE_FAILURE_WITH_FREQ (h7pi-s2), 1 FINETUNE_FAILURE (cd2d-s0)**
- h7pi-s2: ep9 TFP=0.506 (normal), but min_se=71.56 → frequency was correct, structural search failed independently
- cd2d-s0: search converged (min_se=0.24) but relL2=2.84e-4 — finetune didn't reach machine precision
- 22 search_error/relL2 inversions in h7pi-s2, 17 in cd2d-s0 — confirms search/finetune decoupling
- Evidence: `results/gate2-candidate-source-audit/{manifest.json,audit_table.md,summary.json}`

## [Iter 10 Start] 2026-06-23 — scientist: Gate 2 full integration + width/validation stress plan

- Consumed latest audit `audits/audit_iter9_20260623_1900.md` (WARN) and responded to all MAJOR findings in STATE A0.
- Integrated iter-9 Gate 2 data into STATE §4.7:
  - h7pi estimated FFT prior: combined 8 seeds 7/8 search conv, Fisher p=0.020 vs standard 2/8.
  - h8pi estimated FFT prior: 3/3 search conv + machine precision, ep9 TFP≈0.58.
  - h7pi+h8pi combined: estimated 10/11 vs standard 3/11, one-sided Fisher p=0.00376, OR=26.7.
  - convdiff2d estimated FFT prior: 3/3 search conv, 2/3 machine precision, median relL2=1.25e-6 ≈ oracle_hard.
  - decoy24: validation rejected all 3, TFP=0.08≡standard; s1 marked as finetune luck (min_se=647, relL2=1.3e-6).
- Scientific interpretation:
  - Gate 2 claim upgraded to **中→强**: checked FFT prior now works across two helmholtz frequencies and non-Helmholtz convdiff2d.
  - `width_cap=0.25` concentrates support to 2/8 bases, making estimated_soft more hard-like than oracle_soft; this may be the reason cd2d reaches oracle_hard quality.
  - Validation gate is fixed `best_rel_residual < 0.5`; current evidence has pass max≈0.28 and reject min≈0.97, leaving a grey-zone blind spot.
- Next round A1/A3:
  - Task Group A: width-cap sensitivity on h7pi/h8pi/cd2d for cap 0.125 and 0.50.
  - Task Group B: validation boundary sweep plus real-controller grey-zone decoy stress.
  - Task Group C: Gate 2 candidate-source audit for h7pi-s2/cd2d-s0 structure-vs-finetune failures.
- Server choice: used server-health. Arnold 17:00 ET CPU 10%, 1h busy 3/10, GPU5 idle and GPU6-8 low-util but only 210G free; 1202c GPU freer but CPU 91%. Arnold primary, 1202c fallback.
- STATE.md: iteration 9→10, phase→coding_and_running, 384 lines (<400). LESSONS updated with width-cap and validation-gate lessons. §5 remains empty; not agent-editable.

## [Audit] 2026-06-23 19:00 — verdict=WARN (iter 9)
- Report: audits/audit_iter9_20260623_1900.md
- Load-bearing issues:
  - [AUD-MAJOR-001] §4.7 仅含 3-seed h7pi 旧表; iter-9 全部新数据 (h7pi 8-seed, h8pi 3/3, cd2d 3/3, decoy24 rejected) 未整合
  - [AUD-MAJOR-002] Cross-PDE combined Fisher test 未计算 (h7pi+h8pi gives p≈0.004 vs h7pi-alone p=0.020)
  - [AUD-MAJOR-003] Convdiff2d estimated_soft ≈ oracle_hard 水平: width_cap 浓缩机制需科学分析
  - [AUD-MAJOR-004] Validation gate 阈值未文档化 (cd2d 28% 通过, 但实际阈值不明)
- 独立数据核验: 16 result.json + 16 train.jsonl 逐 seed 核对, Fisher p=0.020 手工验算, 全部一致 ✓
- Coder round 评价: 本项目最干净的一轮——5/5 runs collected, all success criteria met, 无 crash
- Required scientist response: (1) §4.7 扩展, (2) combined Fisher, (3) cd2d ≈ oracle_hard 分析, (4) validation 阈值文档, (5) §5 第 11 次, (6) A0 status 更新, (7) decoy24 s1 finetune luck 标注

## [Run Done] 2026-06-23 16:40 — gate2-convdiff2d-est-soft-s0..s2 collected

- Arnold GPU 3-5, 3 seeds, launched 12:03, completed 16:18-16:40, ~13.4 GPU-h RTX 8000 ($12.99)
- **3/3 search converged, 2/3 machine precision** (s1 relL2=1.25e-6, s2 relL2=1.24e-6, s0 relL2=2.84e-4)
- **Median relL2 = 1.25e-6** — matches oracle_hard (1.35e-6), 62x better than oracle_soft (7.69e-5), 2600x vs standard (3.26e-3)
- ep9 TFP ≈ 0.50 for all 3 seeds (higher than oracle_soft's 0.34 due to 2/8 width cap concentration)
- Validation passed: residual 0.27-0.28, selected_bases=[21,24]. Residual ~27% is high but prior still effective
- **Gate 2 cross-PDE gap resolved** — estimated prior works on non-Helmholtz PDE at oracle_hard quality level
- All A2 success criteria met (median 1.25e-6 ≤ 1e-4)
- Evidence: `results/gate2-estimated-soft-convdiff2d-clean/manifest.json`

## [Run Done] 2026-06-23 15:40 — gate2-h8pi-est-soft-s0..s2 collected

- Arnold GPU 6-8, 3 seeds, launched 10:13, completed ~14:10, ~11.8 GPU-h RTX 8000 ($11.48)
- **3/3 search converged + machine precision** (relL2 9.21e-7 to 9.23e-7, min_se 0.17-2.25)
- ep9 TFP ≈ 0.58 for all 3 seeds (higher than h7pi's 0.51 — FFT more precise on h8pi)
- Validation passed: residual 0.14-0.16, selected_bases=[24,21] (2/8 width cap)
- Matches oracle_soft baseline (3/3 conv, median 9.2e-7). All A2 success criteria met
- **Gate 2 cross-frequency gap resolved** — estimated prior works on freq 4.7% outside base set
- Evidence: `results/gate2-estimated-soft-h8pi-clean/manifest.json`

## [Run Sync] 2026-06-23 16:45 — h8pi + cd2d rsync complete

- h8pi: rsync from arnold, all 3 seeds result.json + train.jsonl + train.log synced (excl checkpoint.pt)
- cd2d: rsync from arnold, all 3 seeds result.json + train.jsonl + train.log synced (excl checkpoint.pt)
- Verified: all result.json contain real numbers in expected range, no empty shells, train.jsonl 1010 lines each

## [Run Done] 2026-06-23 12:50 — gate2-h7pi-est-soft-s3..s7 collected

- Arnold GPU 1-5, 5 seeds (s3-s7), launched 08:15, completed ~12:00, ~19 GPU-h RTX 8000 ($18.43)
- **5/5 search converged + machine precision** (relL2 7.2e-7 to 1.2e-6, min_se 0.15-0.37)
- ep9 TFP ≈ 0.51 for all 5 seeds, consistent with seeds 0-2
- Combined 8-seed result: **7/8 search conv, 7/8 relL2<1e-5, Fisher p=0.020 vs standard 2/8**
- Only failure: s2 (from seeds 0-2), TFP=0.51 but structural search failed (min_se=71.6)
- **Gate 2 h7pi statistical power gap resolved**
- Evidence: `results/gate2-estimated-soft-h7pi-seedext/manifest.json`

## [Run Done] 2026-06-23 12:50 — gate2-h7pi-decoy24-s0..s2 collected

- 1202c GPU 2/3/4, 3 seeds, launched 08:26, completed ~11:30, ~9 GPU-h RTX A6000 ($10.44)
- **Validation gate rejected all 3 seeds**: residual 0.97 (threshold ~0.20), fallback=uniform
- ep9 TFP = 0.080 for all seeds ≡ standard baseline — prior correctly neutralized by decoy
- Search conv: 1/3 (standard-level ~25%); s1 finetune luck (relL2=1.3e-6 despite min_se=647)
- FFT estimated angular freq: 23.90 (shifted from true 21.99 toward decoy 24.0 × 2π/2π ≈ 24.0)
- Behavior identical to far-decoy (freq=12): validation residual too high → fallback → standard performance
- **Gate 2 near-target leakage gap resolved**
- Evidence: `results/gate2-estimated-soft-h7pi-decoy24/manifest.json`

## [Run Sync] 2026-06-23 12:45 — h7pi-seedext + decoy24 rsync complete

- h7pi-seedext: rsync from arnold, all 5 seeds result.json + train.jsonl + train.log + checkpoint.pt synced
- decoy24: rsync from 1202c (excluding checkpoint.pt), all 3 seeds result.json + train.jsonl + train.log synced
- Verified: all result.json contain real numbers, relL2 in expected range, no empty shells

## [Run Done] 2026-06-23 09:10 — gate2-decoy-sweep-smoke collected

- Local CPU, 6 decoy scenarios on h7pi (target 7π≈21.99, best base=21)
- **5/6 SAFE, 1 LEAKAGE-WARNING (benign), 0 FAIL**:
  - far-decoy-12-amp2000: SAFE — validation gate rejected (baseline, matches prior smoke)
  - near-decoy-18-amp100: SAFE — low amp doesn't shift FFT peak, target kept, decoy base excluded
  - near-decoy-18-amp2000: SAFE — validation gate rejected (contam estimate shifts to 18.5)
  - near-decoy-24-amp100: LEAKAGE-WARNING — but benign: clean also selects {21,24} (width_cap 2/8), decoy doesn't change prior
  - near-decoy-24-amp2000: SAFE — validation gate rejected (contam estimate shifts to 23.9, val residual 0.977)
  - twopeak-12-24-amp2000: SAFE — validation gate rejected
- Key finding: all high-amplitude (amp=2000) decoys rejected by validation gate regardless of proximity. The decoy24 real-controller run (amp=2000) should safely fallback to uniform prior like far-decoy.
- Evidence: `results/gate2-decoy-sweep-smoke/manifest.json`

## [Iter 9 Start] 2026-06-23 — scientist: Gate 2 integration + robustness plan

- Consumed latest audit `audits/audit_iter8_20260623_1030.md` (WARN) and responded to all 6 MAJOR findings in STATE A0.
- Integrated Gate 2 into STATE §4.7:
  - clean validated FFT prior: 2/3 search conv, 2/3 relL2<1e-5, ep9 TFP=0.510.
  - far decoy (freq=12, amp=2000): validation rejected, fallback to uniform, 0/3 search conv, ep9 TFP=0.080.
  - claim ceiling updated from "missing" to "weak→middle": mechanism positive, but stats/robustness insufficient.
- Scientific interpretation:
  - estimated TFP > oracle because width cap concentrates support to {21,24}; stronger early signal also means greater estimator-fragility risk.
  - clean s2 final TFP=0.902 but min_se=71.56, same failure family as h7pi-s5: frequency locked, structure search fails.
  - NCPSD failed all 3 smoke PDEs, so only checked FFT is claim-bearing for now.
- Next round A1/A3:
  - h7pi estimated prior seeds 3-7 to reach 8 seeds and re-run Fisher vs standard 2/8.
  - h8pi clean and convdiff2d clean real-controller runs for cross-frequency/cross-PDE Gate 2.
  - h7pi near-target decoy24 real-controller run plus P1 multi-peak/low-amp decoy smoke.
- Server choice: used server-health. Arnold 1h busy 0/10, CPU 7%, 415G free; 1202c CPU 96%; majda nearly full. Arnold remains primary.
- STATE.md: iteration 8→9, phase→coding_and_running, 383 lines (<400). §5 remains empty; not agent-editable.

## [Audit] 2026-06-23 10:30 — verdict=WARN (iter 8)
- Report: audits/audit_iter8_20260623_1030.md
- Load-bearing issues:
  - [AUD-MAJOR-001] Gate 2 data not in §4 (4th consecutive iteration — same pattern)
  - [AUD-MAJOR-002] Estimated prior TFP 0.51 > oracle 0.34: width_cap concentrates prior, fragility unexplored
  - [AUD-MAJOR-003] 3 seeds insufficient for Fisher test (p=0.279 vs standard); need 5-8 seeds or cross-PDE
  - [AUD-MAJOR-004] NCPSD estimator fails all 3 PDEs; only FFT works
  - [AUD-MAJOR-005] No Gate 2 real-controller data for convdiff2d (28% validation residual)
  - [AUD-MAJOR-006] Only 1 decoy scenario; need near-target + multi-peak decoys
- Gate 2 data independently verified: all numbers match result.json/train.jsonl ✓
- Required scientist response: (1) Gate 2 → §4, (2) TFP > oracle analysis, (3) sample size decision, (4) NCPSD reporting, (5) cross-PDE Gate 2, (6) decoy diversity, (7) §5 10th reminder, (8) A0 status updates

## [Run Done] 2026-06-23 07:40 — gate2-estimated-soft-h7pi-clean collected

- Arnold GPU4/5/9, 3 seeds, validated_estimated_soft prior, h7pi
- **Clean 2/3 search converged** (success criterion met):
  - s0: relL2=9.55e-7 (MACHINE), min_se=0.26, first_hit ep0/eval0
  - s1: relL2=9.05e-7 (MACHINE), min_se=0.33, first_hit ep0/eval0
  - s2: relL2=2.38e-3 (FAIL), min_se=71.56, first_hit ep0/eval1 (structure search failure despite tfp=0.82)
- ep9 TFP mean=0.510 >> oracle_soft 0.34 >> standard 0.08 (width cap concentrates prior)
- GPU: 3×3.9h×$0.97 = $11.35
- Evidence: `results/gate2-estimated-soft-h7pi-clean/manifest.json`, per-seed `result.json` + `train.jsonl`

## [Run Done] 2026-06-23 07:40 — gate2-estimated-soft-h7pi-decoy collected

- Arnold GPU0/1/2, 3 seeds, decoy-contaminated RHS (decoy_freq=12, decoy_amp=2000)
- **Decoy 0/3 search converged** (validation gate PASS):
  - Validation gate correctly rejected decoy → uniform fallback
  - s0: relL2=4.58e-3, min_se=563.70; s1: relL2=4.30e-3, min_se=696.26; s2: relL2=3.49e-3, min_se=671.93
- ep9 TFP mean=0.080 ≡ standard (no leakage)
- GPU: 3×3.9h×$0.97 = $11.35
- Evidence: `results/gate2-estimated-soft-h7pi-decoy/manifest.json`, per-seed `result.json` + `train.jsonl`

## [Run Sync] 2026-06-23 04:00 — coder: Gate 2 estimator smoke collected + 6 runs deployed

- **gate2-estimator-adapter-smoke**: local, PASS. FFT correctly maps RHS spectrum to base set {21,24} for h7pi (est 22.09 vs true 21.99); width 25% met; validation gate passes clean PDEs and rejects decoy. `results/gate2-estimator-smoke/manifest.json`
- **gate2-estimated-soft-h7pi-clean**: 3 seeds deployed arnold GPU 4/5/9, `validated_estimated_soft` mode. Estimated prior selects bases {21,24} with width 25%. ep9 TFP mean=0.510 (vs oracle_soft 0.34 vs standard 0.08). ep149 best_err: s0=180, s1=131, s2=1360.
- **gate2-estimated-soft-h7pi-decoy**: 3 seeds deployed arnold GPU 0/1/2, decoy_freq=12.0, decoy_amp=2000. Validation gate correctly rejects decoy-contaminated estimate → falls back to uniform prior. ep9 TFP=0.080 ≡ standard. ep149 best_err: s0=98K, s1=22K, s2=43K.
- Code: estimator.py (spectral adapter), run_gate2_estimator_smoke.py, trainer.py/run_gate0_arm.py updates. Commit 8d68250.
- GPU budget so far: 6 × Quadro RTX 8000 × ~0.4h = 2.4 GPU-h, ~$2.3 (search ongoing, expect ~27 GPU-h total)

## [Run Done] 2026-06-23 — gate1-candidate-source-audit collected

- Script: `scripts/analyze_gate1_candidates.py`, local CPU, <1min
- 12 runs × 10 candidates = 120 records audited
- Verification: §4.6 counts exactly reproduced; CP≡standard confirmed (min_se + TFP identical); RB finetune-luck candidates all contain target base-freq ops; 271 se/l2 inversions flagged
- Outputs: `results/gate1-injection-audit/summary.json`, `audit_table.md`, `manifest.json`
- Responds to: AUD-MAJOR-003 (RB candidate source), AUD-MAJOR-004 (se/l2 decoupling)

## [Iter 8 Start] 2026-06-23 — scientist: Gate 1 整合 + Gate 2 estimated prior plan

- Consumed latest audit `audits/audit_iter7_20260623_0800.md` (CRITICAL). Responded to all CRITICAL/MAJOR issues in STATE A0.
- Integrated Gate 1 into STATE §4.6:
  - ep9 TFP is now the key axis: oracle_soft 0.343 immediate vs reward_bonus 0.083 delayed to ep199≈0.36 vs candidate_pruning 0.080.
  - CP search is exactly standard: min_se 573.4811/596.2366/0.1117.
  - RB finetune recovery candidates contain correct base-frequency ops, but search_error remains high; this is finetune rescue, not search convergence.
  - search_error/relL2 detachment recorded (CP-s2 cand0 se=0.111→relL2=0.214; cand9 se=558→relL2=9.06e-7).
- Updated claims: Gate 1 now supports "early action-channel injection matters"; largest remaining gap is checked estimated prior in the real controller.
- Next round A1/A3:
  - local `gate1-candidate-source-audit` for structured candidate-level evidence.
  - `gate2-estimator-adapter-smoke` to bridge proxy spectra to real-controller prior.
  - arnold `gate2-estimated-soft-h7pi-clean` and `gate2-estimated-soft-h7pi-decoy` after smoke.
- Server choice: used server-health; arnold 1h busy 1.8/10, GPU4/5/9 idle-ish; 1202c CPU 96%, so arnold remains primary.
- STATE.md: iteration 7→8, phase→coding_and_running, 378 lines (<400).

## [Audit] 2026-06-23 08:00 — verdict=CRITICAL (iter 7)
- Report: audits/audit_iter7_20260623_0800.md
- Load-bearing issues:
  - [AUD-CRIT-001] Coder TFP 分析遗漏 ep9 时间维度: reward_bonus ep9=0.083=standard, 非 oracle_soft (0.34)。Injection latency 是 Gate 1 核心发现
  - [AUD-MAJOR-001] Gate 1 6-arm 消融数据未整合 §4 (reviewer CRIT #1 直接回应)
  - [AUD-MAJOR-002] CP min_se 与 standard 精确匹配 (4位小数) 确认 CP search ≡ standard
  - [AUD-MAJOR-003] RB "系统性 finetune luck" (0/3 search, 3/3 relL2) 候选来源机制未解释
- STATE maintenance: 405→383 行. 精简已完成 A1 run 规格, 更新机制表
- Required scientist response: (1) ep9 TFP 为 6-arm 对比关键维度, (2) Gate 1 整合入 §4, (3) CP≡standard 标注, (4) RB 候选分析, (5) se/relL2 脱钩讨论, (6) §5 第 9 次提醒

## [Run Done] 2026-06-23 03:00 — coder: Gate 1 lite 6 runs collected

- **gate1-reward-bonus-h7pi**: 3 seeds collected. Search conv 0/3 (min_se: 76-1524), relL2 conv 3/3 (1.12e-6, 1.39e-6, 1.17e-6). 系统性 finetune luck — TFP≈0.34 与 oracle_soft 相当但 search 不收敛。
- **gate1-candidate-pruning-h7pi**: 3 seeds collected. Search conv 1/3 (s2 only, min_se=0.11), relL2 conv 1/3 (s2=9.06e-7). Post-search pruning 不能补偿无引导搜索。
- GPU cost: 6 × Quadro RTX 8000 × ~4h = 24 GPU-h, ~$24
- Manifests updated, evidence complete (result.json + train.jsonl + train_s*.log synced; checkpoints remote-only on arnold)
- Commit: (pending)

## [Run Sync] 2026-06-22 20:12 — coder: Gate 1 lite 部署 + convdiff2d manifest fix

- **fix-convdiff2d-manifest**: code_commit 修正 2b11540→dc94c2e, phase=collected
- **gate1-reward-bonus-h7pi**: 实现 reward shaping arm (reward += λ·I(correct freq))，3 seeds 部署 arnold GPU 0-2, ep199 TFP≈0.34-0.38 ≈ oracle_soft 水平
- **gate1-candidate-pruning-h7pi**: 实现 candidate pruning arm (标准搜索 + finetune 前过滤不含正确频率的候选)，3 seeds 部署 arnold GPU 3-5, ep199 TFP≈0.09-0.13 ≈ standard
- 代码改动: trainer.py (_compute_freq_bonus, _has_correct_freq, reward_bonus/candidate_pruning 逻辑), run_gate0_arm.py (新 ARMS + --reward-bonus-lambda)
- 预计 ~23:30 完成, 下次 coder rsync + 验证 + 标 collected
- Commit: 6822d44

## [Iter 7 Start] 2026-06-22 — scientist: convdiff2d 整合 + soft/hard 反转分析 + Gate 1 lite plan

- Audit iter6 (CRITICAL) 5 条全部回应:
  - CRIT-001 (convdiff2d 未入 §4): §4.5 完整写入 (TFP 4.3x + soft/hard 反转 + advection 宽容性)
  - MAJOR-002 (convdiff2d 定位): 不是纯 scope boundary → "mechanism replicated, outcome diverges by PDE structure"
  - MAJOR-003 (soft/hard 排序反转): convdiff 结构搜索更简单, hard 不伤害; helmholtz 需探索, soft 更优
  - MINOR-001 (§5 为空): 第 8 次提醒用户
  - MINOR-002 (manifest code_commit): dc94c2e, 列入 A1 修复
- Convdiff2d 科学分析:
  - TFP 4.3x 精确复现 helmholtz (0.34 vs 0.08) → retention 机制 PDE-independent
  - Soft/hard reversal: convdiff hard(3/3 机器精度) > soft(1/3); helmholtz soft(16/17) > hard(8/12). 因 PDE 结构搜索复杂度不同
  - Convdiff 对 imperfect retention 更宽容 (standard 也搜索收敛, 但 relL2 差 500x)
- §4.3 claims 新增: "Retention PDE-independent" (**强**), "注入强度 vs 结构复杂度" (**强**), "跨 PDE 泛化" (**中→强**)
- 下一轮 plan: Gate 1 lite 注入消融 (reward_bonus + candidate_pruning × h7pi × 3 seeds = 6 runs, ~28 GPU-h)
- §5 仍为空 — **第 8 次提醒用户: diagnostic paper 还是 full method?**
- STATE.md: iteration 6→7, phase→coding_and_running
- LESSONS.md: +1 条 (soft/hard reversal 与 PDE 结构复杂度)

## [Audit] 2026-06-22 19:12 — verdict=CRITICAL (iter 6)
- Report: audits/audit_iter6_20260622_1912.md
- Load-bearing issues:
  - [AUD-CRIT-001] Convdiff2d 9/9 collected 但 §4 无任何分析 (iter6 coder round 核心产出未整合)
  - [AUD-MAJOR-002] Convdiff2d TFP 4.3x 跨 PDE 复现挑战 "纯 scope boundary" 定位
  - [AUD-MAJOR-003] oracle_soft/hard 排序在 helmholtz/convdiff 上反转, 需科学解释
- STATE maintenance: §2.3 convdiff2d 状态修正, §6 action 更新, A0 reviewer status 更新
- Required scientist response: (1) convdiff2d 整合入 §4, (2) 定性 convdiff2d 论文贡献, (3) soft/hard 排序反转分析, (4) §5 第 8 次提醒, (5) manifest code_commit

## [Run Done] 2026-06-22 19:05 — gate0-convdiff2d: 9/9 collected

- **All 9 runs collected** (3 arms × 3 seeds, 2D convection-diffusion, freq=7π)
- Search convergence: standard 3/3, oracle_soft 3/3, oracle_hard 3/3 → **scope boundary** (success criterion NOT MET)
- relL2 quality gradient: oracle_hard 3/3 ≤ 1.4e-6, oracle_soft 1/3 ≤ 1e-5, standard 0/3 (median 3.3e-3)
- TFP early difference (ep9): oracle_soft ~0.34 vs standard ~0.08 (4.3x) — same as helmholtz, supports PDE-independent mechanism
- oracle_hard-s2 auto-launched by monitor on GPU 4 at 14:58 after 4/8 initial runs completed
- GPU cost: ~$35 (36.1 GPU-h × $0.97/hr Quadro RTX 8000)
- Results: `results/gate0-convdiff2d/*/result.json` + `train.jsonl`, manifest at `results/gate0-convdiff2d/manifest.json`

## [Run Sync] 2026-06-22 19:05 — gate0-convdiff2d: rsync 9/9

- Synced all 9 `result.json` + `train.jsonl` from arnold to local
- Manifest updated with per-condition results and sync_status=collected
- Screens cleaned: 8 cd2d-*-0622 (auto-terminated), oracle_hard-s2 screen, monitor-0622

## [Run Sync] 2026-06-22 — coder: fix-convdiff-manifest collected

- Updated `results/gate0-convdiff/manifest.json` code_commit from "uncommitted (base f3d5011 + ...)" to "eae6ef8"
- AUD-MAJOR-003 resolved

## [Run Crash] 2026-06-22 — coder: gate0-convdiff2d deployed 8/9 on arnold

- Added `convdiff_7pi_2d` PDE case to `run_gate0_arm.py`: 2D convection-diffusion, freqs=[7π,7π], ε=0.01, c=1.0
- Local smoke test passed (PDE residual 1.5e-5, 2-epoch training runs clean)
- Deployed 8/9 runs on arnold GPUs 0-7 (3 standard + 3 oracle_soft + 2 oracle_hard), session prefix `fexsynchro-cd2d-*-0622`
- oracle_hard-s2 pending: 8-GPU limit + satrain project on GPU 8-9. Launcher at `/tmp/fexsynchro-cd2d-oracle_hard-s2-0622.sh`
- Early search data (ep150): oracle_soft 3/3 < 10 (0.38/0.70/1.62), standard 2/3 < 10 (4.38/4.63), oracle_hard 1/2 < 10 (1.95)
- **NOT a crash** — used [Run Crash] tag as closest available tag for deployment+monitoring log

## [Iter 6 Start] 2026-06-22 — scientist: retention 整合 + convdiff 维度诊断 + 2D convdiff plan

- Audit iter5 (CRITICAL) 7 条全部回应:
  - CRIT-001 (数字纠错): 验证通过,auditor 5处修正全部正确
  - CRIT-002 (retention 未入 §4): §4.4 完整重写为 tfp 轨迹分析
  - MAJOR-001 (convdiff criterion): 诊断为维度效应(1D vs 2D),规划 2D convdiff
  - MAJOR-002 (finetune luck): §4.2 已修正 h7pi-s7 (搜索已收敛非 finetune luck)
  - MAJOR-003 (code_commit): 代码在 eae6ef8, manifest 字段待更新
  - MINOR-001/002/003: §5 再次提醒用户; s2 tfp→0.999 分析完; A0 status 更新
- Retention 分析完成:
  - oracle_soft ep9 tfp=0.34 vs standard=0.08 (4.3x 差异) — 早期 bias 是决定性的
  - Standard s0/s1 最终也到 tfp≈0.50 但 min_se≈570-600 (搜索已失败) — 前200 epochs 不可逆
  - oracle_soft-s2 tfp→0.999: ep940 timeout + Cov reinforcement
  - KL_from_init: oracle_soft early≈1.17 vs standard≈0.0004
- Convdiff 科学判断: 1D convdiff (dim=1) vs 2D helmholtz (dim=2) 的搜索空间差异远大于 PDE 类型差异。20 vs 400 频率组合。
- 下一轮 plan: 2D convdiff (dim=2, freqs=[7π,7π]), 3 arms × 3 seeds, ~40 GPU-h
- §5 仍为空 — **第 7 次提醒用户: diagnostic paper 还是 full method (Gate 1-3)?**
- STATE.md: iteration 5→6, phase→coding_and_running

## [Audit] 2026-06-22 11:10 — verdict=CRITICAL (iter 5)
- Report: audits/audit_iter5_20260622_1110.md
- Load-bearing issues:
  - [AUD-CRIT-001] STATE §4.1 数字与修正后 gate0_results.json 不一致 (standard 3/17→4/17, p=6.3e-6→2.96e-5). Auditor 已纠错 5 处.
  - [AUD-CRIT-002] Retention rerun 数据 (6/6, tfp 4.3x 差异) 未整合入 §4, 仍仅在 coder 旁注.
  - [AUD-MAJOR-001] Convdiff success criterion NOT met (standard 也收敛), cross-PDE 仍无新证据.
  - [AUD-MAJOR-002] h7pi-s7 "finetune luck" 标签不准 (min_se=0.245, search 已收敛).
  - [AUD-MAJOR-003] gate0-convdiff code_commit="uncommitted", 代码出处断裂.
- STATE maintenance: 423→400 行. 纠正 5 处过时数字, 精简 A5/A6/coder notes.
- Required scientist response: (1) 验证数字纠错, (2) retention 数据整合 §4, (3) convdiff 方向决策, (4) finetune luck 修正, (5) convdiff code commit, (6) 提醒用户 §5, (7) A0 status 更新

## [Run Done] 2026-06-22 10:20 — retention-h7pi-rerun: 6/6 collected

- **oracle_soft-s2 completed**: search timeout ep940 (10810s), best_err=0.4454 (CONV), relL2=1.3087e-6
  - 10 candidates finetuned, best=candidate 1 (relL2=1.31e-6)
  - first_hit: epoch 0, eval 0
  - train.jsonl deduped: 978→950 lines (28 duplicate epochs 72-86 removed)
  - tfp trajectory: 0.341→0.986→0.999 (unlike s0/s1 which plateau at 0.50, s2 fully locks in)
- **Final 6/6 results**:

| arm | seed | status | min_se | relL2 | first_hit | early_tfp | final_tfp |
|-----|------|--------|--------|-------|-----------|-----------|-----------|
| oracle_soft | 0 | CONV | 0.12 | 1.31e-6 | ep0/ev0 | 0.347 | 0.506 |
| oracle_soft | 1 | CONV | 0.52 | 1.12e-6 | ep0/ev1 | 0.342 | 0.500 |
| oracle_soft | 2 | CONV | 0.45 | 1.31e-6 | ep0/ev0 | 0.341 | 0.999 |
| standard | 0 | FAIL | 573.48 | 2.50e-3 | ep0/ev4 | 0.082 | 0.503 |
| standard | 1 | FAIL | 596.24 | 2.28e-3 | ep0/ev4 | 0.080 | 0.501 |
| standard | 2 | CONV | 0.11 | 9.04e-7 | ep0/ev3 | 0.079 | 0.500 |

- **Summary**: oracle_soft 3/3 search conv vs standard 1/3. All first_hits at epoch 0 (freq discovery trivial). Early tfp 4.3x difference (0.34 vs 0.08). Retention narrative confirmed.
- **Note**: oracle_soft-s2 tfp→0.999 is unique; may be due to search timeout at ep940 freezing policy before exploration could dilute. s0/s1 ran full 1000 epochs and plateaued at 0.50.
- Screen auto-cleaned, no leftover processes on arnold.
- GPU cost: total ~$22.4 for 6 runs (23.1 GPU-h × Quadro RTX 8000 $0.97/h), cumulative $429.2

## [Run Sync] 2026-06-22 07:30 — retention-h7pi-rerun: 5/6 synced, oracle_soft-s2 still searching

- **5/6 runs completed and synced** (result.json + train.jsonl):
  - standard-s0: FAIL (relL2=2.50e-03, se=573, first_hit=ep0/ev4)
  - standard-s1: FAIL (relL2=2.28e-03, se=596, first_hit=ep0/ev4)
  - standard-s2: CONV (relL2=9.04e-07, se=0.11, first_hit=ep0/ev3)
  - oracle_soft-s0: CONV (relL2=1.31e-06, se=0.12, first_hit=ep0/ev0)
  - oracle_soft-s1: CONV (relL2=1.12e-06, se=0.52, first_hit=ep0/ev1)
- **Standard 1/3, oracle_soft 2/2 (s2 pending)**. 一致方向.
- **Retention 证据定论**: ALL first_hits at epoch 0 (频率发现是 trivial), oracle_soft early tfp≈0.35 vs standard≈0.08 (4.2x), 两 arm 最终 → tfp≈0.50 但 standard FAIL
- **oracle_soft-s2**: g5 screen (GPU5, arnold), search converged (best_err=2.70 at epoch 200+), ~epoch 300/1000 at handoff. 预计 search ~09:46, fine-tune ~10:30
- **注意**: oracle_soft-s2 train.jsonl 有 epochs 72-81 双写重复 (dual-write incident, 已杀重复进程), 完成后需 dedup
- GPU cost: ~$22 (6 runs × ~3.8h × Quadro RTX 8000), cumulative ~$405

## [Run Done] 2026-06-22 06:10 — gate0-convdiff: 6/6 collected, all arms converge

- **1D convection-diffusion**: `-0.01*u_xx + u_x = f`, manufactured `u=sin(7πx)`, domain [0,1]
- 3 arms (standard, oracle_soft, oracle_hard) × 2 seeds = 6 runs on arnold GPUs 4-9
- **All 6 runs converge** (min search_error < 10, all relL2 < 2e-6):

| arm | seed | relL2 | min_se | first_hit |
|-----|------|-------|--------|-----------|
| oracle_soft | 0 | 6.43e-7 | 1.08e-10 | eval 0 |
| oracle_soft | 1 | 6.40e-7 | 1.13e-10 | eval 1 |
| oracle_hard | 0 | 1.33e-6 | 9.93e-11 | eval 2 |
| oracle_hard | 1 | 1.57e-6 | 2.29e-10 | eval 1 |
| standard | 0 | 1.61e-6 | 2.67e-10 | eval 4 |
| standard | 1 | 1.33e-6 | 2.20e-10 | eval 4 |

- **Success criterion NOT met**: plan required oracle_soft 2/2 vs standard ≤1/2. Standard also 2/2.
- **Interpretation**: 1D convdiff is too easy—standard also converges. Oracle has marginal advantage in first_hit (0-2 vs 4) and relL2 (6e-7 vs 1.3-1.6e-6, ~2x). The 1D search space is small enough that frequency retention is not a bottleneck.
- **For scientist**: need a harder non-helmholtz PDE (e.g., 2D convdiff or 2D Poisson with oscillatory forcing) to test cross-PDE generalization. Alternatively, this is an honest scope boundary: oracle advantage requires 2D+ where frequency retention matters.
- GPU cost: 6 × 3.7h × $0.97/h = ~$21.5 (total project $404.5)
- Results: `results/gate0-convdiff/`, manifest: `results/gate0-convdiff/manifest.json`

## [Run Sync] 2026-06-22 02:32 — retention-h7pi-rerun deployed + Task Group A completed

- **Task Group A completed (3/3 local runs → collected)**:
  - fix-aggregate: convergence_eval() now uses min(candidate search_error)<10 per §2.2. gate0_results.json regenerated. Key changes: h7pi oracle_soft 6/8→**7/8** (s6 timeout-rebuilt now detected), h7pi standard 1/8→**2/8** (s7 had min_se=0.245 but was missed by old search_curve traversal). 4-PDE: oracle_soft 16/17, standard **4/17** (was 3/17), Fisher p=2.96e-5 (was 6.3e-6). Note to scientist: standard h7pi-s7 had min_se=0.245 (converged) but best_relL2 came from a different candidate with se=577 ("finetune luck" was wrong label for this seed's search convergence).
  - sync-trainlogs: 18 train.jsonl files synced from arnold (oracle_soft + standard × h7pi/h4pi/h8pi × s0-2). Oracle_soft h7pi s0/s1/s2 all ≥1010 lines.
  - fix-manifests: h4pi/h8pi manifests updated: sync_status→collected, code_commit→80a6ed8.
- **retention-h7pi-rerun deployed (6 runs on arnold GPUs 5-9)**:
  - 3 standard + 3 oracle_soft × helmholtz_7pi × seeds 0,1,2
  - trainer.py: new metrics every 10 epochs: target_freq_prob, kl_from_init, target_freq_sampled
  - Bring-up verified at epoch ~40: all GPUs 82-87% util, train.jsonl writing
  - Early signal: oracle_soft target_freq_prob=0.35 vs standard=0.08 at epoch 39 (4x difference)
  - GPU5 runs 2 jobs sequentially (std-s0 then oracle_soft-s2)
  - Estimated ~4.5 GPU-h per run, total ~27 GPU-h, ~$26
  - 5 screens: fex-ret-g{5..9}-0622-0232*

## [Iter 5 Start] 2026-06-22 — scientist: reviewer response + retention plan + convdiff plan

- Lit-feed 6 条消费完毕: promoted 4 (2505.22617, 2509.04259, 2510.18874, 2510.18927 → retention 实验设计 + 理论支撑), discarded 2 (2510.10150 overkill, 2603.11682 部分 → LESSONS)
- Reviewer 6 must-fix 逐条回应（A0）:
  - #1 aggregate 脚本: → A1 fix-aggregate
  - #2 证据链: → A1 sync-trainlogs + fix-manifests
  - #3 Retention 直接证据（核心缺口）: → A1 retention-h7pi-rerun（加 target_freq_prob + kl_from_init 日志）
  - #4 方法描述 vs 代码: → §3.1 已修正为 alpha-aware Gaussian score（本轮完成）
  - #5 非 helmholtz PDE: → A1 gate0-convdiff（convection-diffusion）
  - #6 Claim scope: 不降级，按 full proposal 继续；等用户 §5
- Audit CRIT-001 回应: 同 reviewer #1
- 核心科学判断:
  - 现有 train.jsonl 已有初步 retention 证据（standard 熵坍缩 3.22→0 锁定错误频率）
  - 但 oracle_soft 对比数据缺失（本地 train.jsonl 为空）
  - 设计 3 项新增指标 (target_freq_prob, kl_from_init, target_freq_sampled) 直接量化 retention
  - 理论支撑: top-ε 过滤机制 (BAPO) + Cov(log π, π·A) 单调递减 (2505.22617)
- §4.4 新增初步 entropy 轨迹表
- 估计下轮 GPU 成本: ~54 GPU-h (~$52): retention 27h + convdiff 27h
- STATE.md: iteration 4→5, phase→coding_and_running
- LESSONS.md: +3 条（top-ε 机制 / REPO 替代 / Gate 1-3 搁置路线）

## [Audit] 2026-06-22 02:00 — verdict=CRITICAL (iter 4)
- Report: audits/audit_iter4_20260622_0200.md
- Load-bearing issues:
  - [AUD-CRIT-001] aggregate_gate0.py convergence logic (search_curve traversal) misaligned with §2.2 (min candidate search_error). Reports 6/8 for oracle_soft h7pi, actual is 7/8. Reviewer must-fix #1 unfixed.
  - [AUD-MAJOR-001-005] Reviewer 6 must-fix items (aggregate script, evidence chain, retention evidence, code-description mismatch, PDE diversity, claim scope) all unfixed.
  - [AUD-MAJOR-003] Lit-feed inbox has 6 unprocessed papers directly relevant to reviewer must-fix #3 (retention diagnostics).
- §4 数字经原始文件独立验证: 16/17 vs 3/17, p=6.3e-6 confirmed ✓
- Required scientist response: (1) fix aggregate_gate0.py, (2) process lit-feed inbox, (3) respond to reviewer 6 must-fix, (4) fix method description, (5) update manifests

## [Review of Version 0] 2026-06-22 01:30 — score=4.3/10
- Verdict: not ready
- Primary concern: 只完成 Gate 0 oracle diagnostic，Gate 1-3 全部缺席; 4 个有效 PDE 全是 helmholtz_sine 变体不足以支撑 cross-PDE claim; 聚合脚本/manifest/方法描述与代码存在可修复但未修复的不一致

## [Iter 4 Start] 2026-06-22 — scientist analysis: retention narrative + 4-PDE integrate + send reviewer

- Audit response: 3 MAJOR + 4 MINOR 逐条回应，全部 accept
- 核心发现——**频率保持（retention）而非频率发现（finding）是主瓶颈**：
  - h4pi/h8pi 的 first_hits 显示 standard 在 epoch 0 前几次评估（eval 1-5）就找到正确频率，但 2/3 seeds 后来在 RL 探索中丢失
  - Oracle_soft 的机制是持续稳定化（prevention of RL drift），不是帮助发现
- §4 全面重写：
  - h7pi oracle_soft search convergence 修正为 7/8（min_se=0.40 for s4, not 6/8）
  - h4pi + h8pi 数据整合入 §4.1 结果表
  - 4-PDE Fisher test: search 16/17 vs 3/17, p=6.3e-6; relL2 15/17 vs 4/17, p=1.8e-4
  - §4.2 新增三种独立失败模式分类（频率漂移 / 结构搜索 / finetune）
  - §4.3 claims 全部更新为 "4 频率一致"
- Search convergence 定义明确：min(all candidates' search_error) < 10（§2.2 更新）
- "Cross-PDE" 语言修正为 "cross-frequency"——4 个 helmholtz_sine 变体不足以支撑 cross-PDE claim
- STATE.md: iteration 3→4, phase→needs_reviewer
- LESSONS.md: 新增 3 条科学经验（retention, min_se, failure taxonomy）

## [Audit] 2026-06-22 00:58 — verdict=WARN (iter 3)
- Report: audits/audit_iter3_20260622_0058.md
- Load-bearing issues:
  - [AUD-MAJOR-001] oracle_soft h7pi search convergence = 7/8 not 6/8 (s4 search_err=0.40<10). Fisher test 9/11→10/11 vs 1/11, p=0.000173 (was 0.00095). "82% vs 9%" → "91% vs 9%"
  - [AUD-MAJOR-002] §4 未更新 h4pi/h8pi. 4-PDE aggregate: oracle_soft 16/17 vs standard 3/17, p=6.3e-6
  - [AUD-MAJOR-003] first_hits 揭示 oracle 机制是频率 retention 而非 finding: standard 在 epoch 0 就找到频率但后来丢失
- Required scientist response: (1) 修正 h7pi search count + Fisher test, (2) 更新 §4 全 4 PDE 数据, (3) 分析 first_hits / retention 叙事, (4) 评估 "cross-PDE" 语言

## [Run Done] 2026-06-22 00:47 — gate0-h4pi + gate0-h8pi 18/18 ALL COLLECTED

- **oracle_hard h4pi**: 2/3 search 收敛 (s1=4.78e-7, s2=4.75e-7 机器精度; s0=4.10e-3 search 未收敛 23.24>10)
- **oracle_hard h8pi**: 3/3 search 收敛 全部机器精度 (s0=9.26e-7, s1=9.24e-7, s2=9.30e-7)
- h8pi oracle_hard 3/3 = 与 oracle_soft 3/3 完全一致，进一步确认 base set 边界 (gap 4.7%) alpha 能桥接
- h4pi oracle_hard 2/3 vs oracle_soft 3/3: s0 search 未收敛是 oracle_hard 的已知弱点（类似 h7pi-s0 oracle_hard=1.82e-3 vs oracle_soft=1.30e-6）
- aggregate_gate0.py: 91 runs total (原 73 + 新 18)
- batch3 GPU: 70.1 GPU-h × $0.97 = $68, 累计 $383
- Screens cleaned, no remaining GPU processes

## [Run Sync] 2026-06-21 21:55 — h4pi + h8pi oracle_soft/standard 12/18 synced, oracle_hard running

- **h4pi 完整结果**: oracle_soft 3/3 机器精度 (relL2~4.6e-7, search_err<0.02), standard 1/3 (s2=4.6e-7)
  - first_hit: oracle_soft 在 epoch 0 eval 0-1 即命中, standard 在 eval 3-5
  - search_err: oracle_soft max=0.016, standard 失败 seeds 259/27.3
- **h8pi 完整结果**: oracle_soft 3/3 机器精度 (relL2~9.2e-7, search_err<1.01), standard 1/3 (s0=3.87e-6)
  - h8pi freq=8π≈25.1 超 base set 4.7%, alpha_i 成功桥接了小 gap
  - standard-s0 收敛 (search_err=0.30, relL2=3.87e-6) — "finetune luck" 类似 h7pi-s7
- **oracle_hard 搜索中** (6 screen on arnold): h4pi-s1/s2 + h8pi-s1/s2 已收敛, s0 pair 还在挣扎
- 结果极度一致: 4 个有效 PDE (h4pi/h5pi/h7pi/h8pi) 全部 oracle_soft >> standard
- 12/18 result.json 已 rsync 到本地, oracle_hard 预计 00:00-00:30 完成

## [Run Sync] 2026-06-21 ~16:20 — gate0-h4pi/h8pi batch3 deployed, h4pi early results

- Deployed 18 runs (3 arms × 3 seeds × 2 PDEs) on arnold GPUs 0,3,4,5,6,7
- Code: added helmholtz_4pi (4π≈12.6, base set 内) + helmholtz_8pi (8π≈25.1, base set 边界)
- Fixed: standard-h5pi-s1 timeout result.json reconstructed from train.log (best_rel_l2=2.13e-3), aggregate now 73 runs
- **h4pi search phase results (oracle_soft 3/3 converged, standard 1/3)**:
  - oracle_soft: all 3 seeds converged by E150 (best_err 0.006-0.048), Candidate 0 relL2=4.96e-7
  - standard: s2 converged (E300, best_err=0.036), s0 locked at 259 (freq_H=0.014), s1 stuck at 27
  - This is higher standard success than h5pi (0/3) or h7pi (1/8), confirming "low freq easier" hypothesis
- h8pi + oracle_hard running next (auto-queued in same screens), expected completion ~22:00 EDT
- GPU cost estimate: 18 runs × 1 GPU × ~4.5h × $0.97/GPU-h = ~$78.6 (actual will vary)

## [Iter 3 Start] 2026-06-21 — scientist analysis of batch 2 + audit response + h4pi/h8pi plan

- Audit response: 2 CRITICAL + 3 MAJOR + 4 MINOR 全部逐条回应
- 核心叙事修订: "消除 seed 方差" → "提高搜索收敛率 82% vs 9%"（Fisher p<0.001 cross-PDE）
- oracle_soft-s5 分析: first_hit=eval 2 (oracle 成功) + search_error=520 (结构搜索失败) → 频率/结构两个独立失败模式的因果分离证据
- §4 全面重写：含 h5pi/h7pi-8seed/h11pi/h19pi/统计检验/收敛阈值定义
- §2.2 新增收敛判据: search convergence (search_error<10) 作为主判据; relL2<1e-5 收敛阈值
- Evidence gap: 有效 PDE 仅 2 个 (h5pi, h7pi)
- Next plan: helmholtz_4pi (base set 内低频, 4π≈12.6) + helmholtz_8pi (base set 边界, 8π≈25.1)
  - h4pi: 测试低频端是否复现 oracle_soft 优势; 若 standard 也成功则说明"频率越高越需要 oracle"
  - h8pi: 测试 base set boundary 是尖锐还是渐进 (h7pi gap≈0% ✓, h11pi gap=44% ✗, h8pi gap=4.7% ?)
  - 18 runs total, ~27 GPU-h, ~$26
- LESSONS.md: 新增 4 条科学经验 (小样本叙事翻转, 频率距离降解, 主判据选择)
- STATE.md: iteration 2→3, phase→coding_and_running

## [Audit] 2026-06-21 12:09 — verdict=CRITICAL (iter 2)
- Report: audits/audit_iter2_20260621_1209.md
- Load-bearing issues:
  - [AUD-CRIT-001] 8-seed data 否定 "消除 seed 方差" 叙事: oracle_soft variance 2747x (非 1.2x), s5 first_hit=eval 2 但搜索失败 (519 error)
  - [AUD-CRIT-002] 收敛标准未定义: search convergence (6/8 vs 1/8, p=0.020) vs relL2 (6/8 vs 2/8, p=0.066); 跨 PDE 合并后 p≈0.001
  - [AUD-MAJOR-002] 有效 PDE 仍只 2 个 (h5pi + h7pi), 需要 helmholtz_9pi 等
- Required scientist response: (1) 修订 variance 叙事, (2) 明确 convergence criterion + 跨 PDE Fisher test, (3) 更新 §4 全部 batch 2 数据, (4) 决策是否跑更多 PDE, (5) 分析 oracle_soft-s5 + h11pi 部分优势

## [Run Done] 2026-06-21 11:46 — gate0-h7pi-extra-seeds COLLECTED; batch 2 ALL 37/37 done

- run: gate0-h7pi-extra-seeds → collected (was running, last condition standard-h7pi-s3 finished)
- **standard-h7pi-s3**: rel_l2=1.86e-3, search did NOT converge (best_err=517, threshold=10). Interesting: freq 7π first-hit at epoch 2, but tree structure search failed.
- **h7pi final 8-seed tallies**:
  - oracle_soft: 6/8 machine precision (s0-s3,s6,s7 ≈ 1e-6; s4=4.4e-5 borderline; s5=2.5e-3 FAIL)
  - standard: 2/8 machine precision (s2=1.3e-6, s7=1.1e-6; rest 2e-4 to 1.5e-2)
- gate0_results.json re-aggregated: 72 runs total. 3 timeout-rebuilt files (oracle_soft-s6, standard-s5, standard-s7) fixed with meta fields.
- Manifest: results/gate0-h7pi-extra-seeds/manifest.json
- All screens on arnold cleaned up. GPU cost: batch 2 total ~$138, project total ~$295.

## [Run Done] 2026-06-21 08:30 — gate0-h11pi, gate0-h19pi COLLECTED; h7pi oracle_soft 7/8 converged

- runs: gate0-h11pi → collected, gate0-h19pi → collected, gate0-h7pi-extra-seeds → running (1 condition left)
- **h11pi scope boundary confirmed (9/9)**: oracle_hard s0=0.985, s1=0.859, s2=0.948. All arms 0/3 converged. 11π≈34.56 exceeds base set {3..24} by 44%.
- **h19pi scope boundary confirmed (9/9)**: oracle_hard s1=0.986 (last condition). All arms 0/3 converged. 19π≈59.69 exceeds base set by 149%.
- **h7pi oracle_soft final: 7/8 converged (87.5%)**: s3=9.10e-7 ✓, s7=1.11e-6 ✓ (new this round). Only s5=2.49e-3 failed. Standard 2/6 converged (33%).
- **standard-h7pi-s3 still running on GPU 0**: was in original deployment queue after h11pi conditions. Started at 07:53 EDT, epoch 200/1000. ETA ~12:00 EDT.
- All new result.json synced to local and validated: 6 files with real non-zero numbers in expected range.
- Manifests: results/gate0-h11pi/manifest.json, results/gate0-h19pi/manifest.json
- GPU cost batch 2: ~$135 total (6 GPUs × ~23h avg × $0.97/hr). Project total: ~$292.

## [Run Sync] 2026-06-21 04:37 — batch2 monitoring: 30/37 conditions done, h11pi scope boundary confirmed

- runs: gate0-h11pi, gate0-h19pi, gate0-h7pi-extra-seeds — all still `running` (7 conditions remaining)
- **h11pi scope boundary**: standard 0/3 (relL2 0.93-0.96), oracle_soft 0/3 (0.78-0.95). 11π≈34.56 exceeds base set {3..24} — search cannot converge (best_err ~7e5 vs h7pi's ~0.1)
- **h19pi scope boundary**: oracle_soft s1 collected (relL2=0.923). Now 8/9 done, all ~0.92-0.97. Only oracle_hard s1 remaining
- **h7pi extra seeds**: oracle_soft 5/6 converged (s0-2: ~1e-6, s4: 4.41e-5, s6: 9.05e-7; s5: 2.49e-3 FAIL). standard 2/7 converged (s2, s7: ~1e-6)
- 5 timeout-killed runs reconstructed from train.log: standard-h7pi-s5 (2.07e-4), s7 (1.12e-6), oracle_soft-h7pi-s6 (9.05e-7), oracle_soft-h11pi-s1 (0.777), s2 (0.785)
- Remaining 7 runs: oracle_hard h11pi ×3 + oracle_hard h19pi s1 + oracle_soft h7pi s3/s7 + standard h7pi s3. ETA ~12:00 EDT
- GPU cost batch2 so far: ~$114 (6 GPUs × 19.7h × $0.97/hr). Total project: $270.66

## [Run Done] 2026-06-20 23:05 — gate0-h5pi COLLECTED (8/9 result.json + 1 timeout)

- run: gate0-h5pi (helmholtz_5pi, freq=5π≈15.71, 3 arms × 3 seeds)
- oracle_soft: **3/3 near-machine precision** (6.9e-7, 7.6e-6, 6.0e-7) — replicates h7pi pattern
- oracle_hard: 1/3 machine precision (3.0e-6), 2/3 failed (5.6e-4, 8.6e-3)
- standard: 0/2 converged (2.9e-3, 7.5e-3), s1 timeout at finetune candidate 6/10 (best=2.1e-3)
- First strong cross-PDE replication of Gate 0 finding
- Manifest: results/gate0-h5pi/manifest.json
- Evidence synced to local, validated: all result.json have real, reasonable numbers

## [Run Sync] 2026-06-20 08:57 — gate0 batch2 deployed: 37 conditions on arnold GPUs 0,5-9

- runs: gate0-h5pi (9), gate0-h11pi (9), gate0-h19pi (9), gate0-h7pi-extra-seeds (10)
- conditions: 3 arms (standard/oracle_soft/oracle_hard) × 3 PDEs (h5pi/h11pi/h19pi) × 3 seeds + 2 arms × h7pi × 5 extra seeds
- code: added helmholtz_5pi/11pi/19pi PDE configs, rewrote aggregate_gate0.py with efficiency metrics
- 6 screen sessions: fex-g{0,5,6,7,8,9}-*-0620-085753
- GPU queue: gate0_gpu_queue.sh per GPU, sequential within GPU
- estimated wall-clock: ~28h (7 conditions max per GPU × ~4h each, based on epoch rate ~5/min and previous round timing)

## [Iter 2 Start] 2026-06-20 — scientist analysis of Gate 0 + multi-PDE plan

- Audit response: 9 findings 逐条回应（3 CRITICAL, 4 MAJOR, 2 MINOR），全部 accept 或 accept-and-use
- Gate 0 science:
  - oracle_soft 唯一消除 seed 方差（3/3 机器精度, variance 1.2x vs standard 3486x）
  - oracle_hard 2/3 成功但 s0 失败 → 结构搜索独立贡献方差 → verdict 修订为 "major, often dominant bottleneck"
  - init_only ≡ standard（search curves byte-for-byte identical）→ alpha 不参与 RL search
  - 效率指标已手动提取：oracle_soft first-hit eval 0-1, 搜索收敛 median 1260 evals; standard 1/3 在 1980 evals
- 最大 evidence gap: 仅 1 个 PDE (h7pi) 支撑结论
- Next plan: 3 个新 PDE (h5pi/h11pi/h19pi) × 3 arms × 3 seeds = 27 runs + optional h7pi extra seeds
- LESSONS.md 首次填写（crash 修复经验 + soft>hard 科学经验）
- STATE.md: iteration 1→2, phase→coding_and_running

## [Audit] 2026-06-20 08:33 — verdict=CRITICAL (iter 1)
- Report: audits/audit_iter1_20260620_0833.md
- Load-bearing issues:
  - [AUD-CRIT-001] init_only arm search curves identical to standard (not a valid control)
  - [AUD-CRIT-002] first_hits/search_curve/total_candidate_evals collected per-run but not aggregated into gate0_results.json; A1 success criterion never evaluated
  - [AUD-CRIT-003] oracle_hard 1/3 seeds search-failed despite forced correct frequencies → "frequency search IS the primary bottleneck" overclaim
  - [AUD-MAJOR-001] Only 1 positive PDE case → "single-frequency PDEs" scope overclaim
  - [AUD-MAJOR-003] oracle_soft-h4_17-s0=0.39 anomaly unreported (2.5x better than other arms)
- Required scientist response: (1) init_only validity, (2) aggregate efficiency metrics, (3) oracle_hard failure root-cause, (4) scope narrowing, (5) oracle_soft h4_17 anomaly, (6) integrate Gate 0 results into §4

## [Run Done] 2026-06-20 08:18 — gate0-oracle-audit COMPLETE (36/36 runs collected)

- run: gate0-oracle-audit (36 runs on arnold GPUs 1-8, 6 arms × 2 PDEs × 3 seeds)
- Total GPU cost: $156.18 (161 GPU-h on RTX 8000 @ $0.97/hr, 4 batches over 19.3h wall-clock)
- All 36 result.json validated: real numbers, non-empty, 6-10 candidates per run
- 5 runs reconstructed from train.log (timeout-killed, 6-9/10 candidates each)
- Aggregate written to results/gate0-oracle-audit/gate0_results.json

### Gate 0 Complete Results (best_rel_l2, 3 seeds):

**helmholtz_7pi** (single-frequency, freq=7π):

| arm | s0 | s1 | s2 | median | variance |
|-----|----|----|----|----|----------|
| standard | 4.58e-3 | 2.28e-3 | 1.31e-6 | 2.28e-3 | 3486x |
| standard_freeze | 5.36e-1 | 3.92e-1 | 3.98e-1 | 3.98e-1 | 1.4x |
| oracle_hard | 1.82e-3 | 1.31e-6 | 1.31e-6 | 1.31e-6 | 1397x |
| **oracle_soft** | **1.30e-6** | **1.12e-6** | **1.30e-6** | **1.30e-6** | **1.2x** |
| oracle_soft_freeze | 3.96e-1 | 3.96e-1 | 3.92e-1 | 3.96e-1 | 1.0x |
| init_only | 2.50e-3 | 2.28e-3 | 2.34e-6 | 2.28e-3 | 1067x |

**helmholtz_4_17** (multi-frequency, freqs=4π+17π): ALL arms relL2 > 0.39 — universal failure.

### Key Findings:
1. **oracle_soft eliminates seed variance on helmholtz_7pi**: all 3 seeds → machine precision (~1e-6), variance 1.2x vs standard's 3486x
2. **oracle_hard is unreliable**: s0 still fails (1.82e-3), variance 1397x. Forcing frequencies ≠ soft biasing.
3. **init_only has same variance as standard**: alpha initialization alone is insufficient, seed variance 1067x
4. **freeze_alpha degrades 100-400x**: alpha_i is infrastructure-critical component
5. **helmholtz_4_17 universal failure**: depth-2 tree cannot express well-separated multi-frequency solutions, independent of frequency prior
6. **Verdict**: frequency search IS the primary bottleneck for single-frequency PDEs; soft prior (logit bias) is the correct injection mechanism

## [Run Sync] 2026-06-20 00:43 — gate0-oracle-audit batch 3 complete, oracle_soft core results

- run: gate0-oracle-audit (36 runs on arnold GPUs 1-8)
- Batch 3 (8 runs): oracle_soft all 6 + oracle_hard h4_17 s1/s2. All search 1000/1000 epochs, all finetune completed before outer timeout.
- 2 batch-2 timeout runs reconstructed from train.log (9/10 candidates each): oracle_hard-h7pi-s1 (best relL2=1.31e-6), oracle_hard-h4_17-s0 (best relL2=0.946)
- Cumulative: 24/36 result.json collected
- GPU cost batch 3: $28.81 (8 GPUs × avg 3.7h × $0.97/hr) + $2.10 batch 4 partial = $30.91. Total: $113.58

### Gate 0 Four-Arm Results (helmholtz_7pi, freq = 7π):

| arm | s0 | s1 | s2 | median | seed range |
|-----|----|----|----|----|------------|
| standard | 4.58e-3 | 2.28e-3 | 1.31e-6 | 2.28e-3 | 3486x |
| standard_freeze | 5.36e-1 | 3.92e-1 | 3.98e-1 | 3.98e-1 | 1.4x |
| oracle_hard | 1.82e-3 | 1.31e-6* | 1.31e-6 | 1.31e-6 | 1397x |
| oracle_soft | 1.30e-6 | 1.12e-6 | 1.30e-6 | 1.30e-6 | **1.16x** |

- **Key finding**: oracle_soft eliminates seed variance entirely. All 3 seeds converge to machine precision (~1e-6). Standard only 1/3 seeds succeeds. Soft prior (logit bias on frequency actions) is the correct injection mechanism.
- oracle_hard (forced frequencies) is unreliable: s0 still fails at 1.82e-3, similar to standard
- standard_freeze confirms alpha_i is infrastructure-critical (100-1000x degradation)
- helmholtz_4_17: universal failure (relL2 > 0.39 all arms). Well-separated multi-frequency PDE exceeds depth-2 tree capacity.
- Batch 4 (oracle_soft_freeze + init_only) auto-launched, 8/12 runs in search phase

## [Run Sync] 2026-06-19 20:57 — gate0-oracle-audit batch 2 results synced

- run: gate0-oracle-audit (36 runs on arnold GPUs 1-8)
- Batch 2 (runs 9-16): 6 new result.json + 2 timeout-killed (oracle_hard-h7pi-s1 search 872/1000ep, oracle_hard-h4_17-s0 best_err=17M)
- Cumulative: 14/36 runs collected (11 remote + 3 local partial)
- GPU cost this batch: $31.17 (8 GPUs × 4.02h × $0.97/hr). Total: $82.68
- New results:
  - standard_freeze-helmholtz_7pi-s2: relL2=0.398 (alpha frozen, consistent with s0/s1 ~0.4-0.5)
  - standard_freeze-helmholtz_4_17: s0=0.920, s1=0.939, s2=0.904 (all failure, ~same as standard)
  - oracle_hard-helmholtz_7pi-s0: relL2=1.82e-3 (vs standard s0=4.58e-3, 2.5x improvement)
  - oracle_hard-helmholtz_7pi-s2: relL2=1.31e-6 (identical to standard s2, both found correct structure)
- Key finding: oracle_hard shows limited improvement over standard on helmholtz_7pi. Seed variance (1000x) persists even with oracle frequencies. Frequency search may not be the primary bottleneck — structure search + coefficient optimization dominate.
- Batch 3 running: oracle_soft + remaining oracle_hard helmholtz_4_17. Remaining 22 runs ETA ~06-20 09:00

## [Run Sync] 2026-06-19 17:00 — gate0-oracle-audit batch 1 results synced

- run: gate0-oracle-audit (36 runs queued on arnold GPUs 1-8)
- Batch 1 (8 runs): 5 completed with result.json, 3 killed by 14400s wrapper timeout (partial results recovered from screen logs)
- GPU cost so far: $31.04 (8 GPUs × 4h × $0.97/hr)
- Completed results:
  - standard-helmholtz_7pi: s0=4.58e-3(8/10), s1=2.28e-3, s2=1.31e-6(9/10)
  - standard-helmholtz_4_17: s0=0.9926, s1=0.856(6/10), s2=0.9117
  - standard_freeze-helmholtz_7pi: s0=0.5358, s1=0.3916
- Key findings: (a) helmholtz_4_17 standard complete failure despite first_hit at epoch 0; (b) freeze-alpha 100x worse than free-alpha; (c) 300x seed variance on helmholtz_7pi
- Timeout root cause: satrain (fex-dim-lift-skeleton) co-located on GPUs 1-8 at 15:48, slowing finetune ~15%
- Batch 2 (oracle_hard + standard_freeze) in search phase. All 36 runs ETA ~06-20 09:00

## [Run Done] 2026-06-19 ~13:00 — msfex-calibrate calibration complete

- run: msfex-calibrate (3 seeds × 1 GPU on arnold)
- GPU cost: $20.47 (21.1 GPU-h on RTX 8000 @ $0.97/hr)
- Calibration verdict: **PASS** (seed2 rel_l2 = 1.57e-6 < target 1e-5)
- Results:
  - seed0: rel_l2 = 5.00e-4 — search did NOT converge (freq_entropy=0.016), finetune helped but not enough
  - seed1: rel_l2 = 1.21e-3 — search did NOT converge (freq_entropy=0.073), finetune helped but not enough
  - seed2: rel_l2 = 1.57e-6 — search CONVERGED (freq_entropy=0.0003), found correct frequency, finetune achieved paper-level accuracy
- Key observation: 2/3 seeds failed to find the correct frequency in 2000 epochs of RL search. This 300x gap in relL2 between converged (seed2) and unconverged (seed0/1) seeds IS the phenomenon that Gate 0 will investigate.
- Files: results/msfex-calibrate/seed{0,1,2}/{calibration.json, checkpoint.pt, train.jsonl, train.log}
- Note: finetune was initially blocked by timeout issue (crash #3). Fixed CandidatePool persistence and redeployed with external timeout=7200s. Pool was rebuilt from last ~100 search epochs; seed2 had only 1 unique candidate (controller fully converged).

## [Run Crash] 2026-06-19 ~11:00 — msfex-calibrate crash #3 (finetune never ran)

- run: msfex-calibrate (3 seeds × 1 GPU on arnold)
- crash_count: 3
- Root cause: external `timeout 20400` command and Python-level `timeout_seconds=20400` set to same value. Search phase consumed ~20300s of the 20400s budget. After Python exited search loop, the external timeout killed the process before `_finetune()` could execute. No finetune entries in train.jsonl, no `calibration.json` produced.
- Search results (not lost):
  - seed0: 1900/2000 epochs, best_error=258.5 (raw PDE loss), freq_entropy=0.016 — did NOT converge
  - seed1: 1944/2000 epochs, best_error=557.1, freq_entropy=0.073 — did NOT converge
  - seed2: 1890/2000 epochs, best_error=0.112, freq_entropy=0.0003 — CONVERGED (found correct frequency)
- Secondary issue: CandidatePool (top-10 action/error pairs) was not persisted to checkpoint, so resume would lose all discovered candidates.
- Fix applied: (1) Added `CandidatePool.state_dict()`/`load_state_dict()` for checkpoint persistence (2) Will re-deploy with external timeout > search + finetune total time.
- Evidence: results synced to local `results/msfex-calibrate/seed{0,1,2}/` — checkpoint.pt, train.jsonl, train.log present. EXIT_CODE=0 in train.log is tee's exit code, not Python's.

## [Run Sync] 2026-06-19 07:20 — msfex-calibrate mid-run health check

- run: msfex-calibrate (3 seeds on arnold GPUs 0-2)
- PIDs: 2731337/2731393/2731450 — all alive, 99.1% CPU
- Progress: seed0 749/2000 epochs (best_err=573), seed1 771/2000 (best_err=596), seed2 749/2000 (best_err=0.112)
- Outputs: checkpoint.pt (115KB, updated ~07:10) + train.jsonl (280-295KB, updated ~07:17) per seed — healthy
- train.log only 2 lines: Python stdout block-buffered through tee; stderr warning passed through. Real data in train.jsonl.
- GPU util 6-7%, 230 MiB/process — expected for tiny FEX controller (CPU-bound, 99% CPU confirms)
- Torch kernel cache warning: `/scr1/scratch/sun1245/xdgcache/sun1245/torch/kernels` not created, JIT cache disabled (minor perf impact)
- Rate: ~10.3s/epoch. Remaining search: ~3.5h. Finetune (20k iters × 10 candidates): ~0.5-1h after. Total ETA: ~11:00-11:30 UTC.
- seed2 converged to specific frequency (entropy→0.002, best_error 235027→0.112). seed0/1 still exploring (entropy 0.36/0.17).

## [Run Crash] 2026-06-19 — msfex-calibrate crash #2 (alpha explosion)

- run: msfex-calibrate (3 seeds × 1 GPU)
- server: arnold, GPUs 0-2
- crash_count: 2
- Root cause #1 (crash #1): `base_clamp=100` 太低导致 helmholtz_sine 的所有 errors (O(1e5)) 被压平到 100，controller 梯度为零。Fix: 去掉 base_clamp，用 nan_to_num 替代。
- Root cause #2 (crash #2): LBFGS 优化 alpha_i 时无约束导致 alpha 发散到 -12618 (seed1)。Fix: 每步 Adam/LBFGS 后 clamp alpha 到 [0.01, 20.0]。
- 修复后第 3 次部署，三种修复全部应用。

## [Init] 2026-06-19 — workspace 接手 (slug: fex-synchro-prior)

- Topic: topics/0616-fex.md
- Landscape: topics/0616-fex-landscape.md
- Idea: ideas/fex-synchro-prior.v5.md
- Proposal: ideas/fex-synchro-prior-proposal.v2.md
- Cleanup commit: (pending, this init)
- First route: route/gate0-msfex-oracle
- Claim 主攻: 频率搜索是否是 Multi-Scale FEX 宽频率失败的主瓶颈（Claim 1: oracle audit）
- Plan 简述: Task Group A 实现并校准 Multi-Scale FEX（复现 paper benchmark），Task Group B 跑 Gate 0 oracle 四臂实验（standard/oracle-hard/oracle-soft/init-only + alpha audit, 3 seeds）。Pilot signal POSITIVE_BOUNDED（CuPy PG proxy FFT prior 64/64 命中 median 7 evals vs uniform 23/64 median 128; real-FEX smoke oracle 5e-8 vs standard 5.2e-2）。Pre-gate 校准是 stop/go gate：复现不了就停。
