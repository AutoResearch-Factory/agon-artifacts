# Lessons: fex-search-complexity

<!--
LESSONS.md: 本 workspace 内沉淀的可迁移知识总账.
与同目录另两份文档三足互补:
- STATE.md 是一篇 mini-文章 (送审快照), 记载了最后会呈现在文章上的精华
- experiment-log.md 答 "这个 workspace 经历了什么" (时间倒序流水账), 记录了所有细节
- LESSONS.md 补全 STATE.md 和 experiment-log.md 中间缺失的层次, 是提炼后的可迁移知识

该记什么:
1. 人类嘱托: 用户在本 workspace 工作过程中的指令, 每条带 [YYYY-MM-DD]. 记录人类嘱托时必须记录用户原文; 只允许修正明显 typo, 不得改写、概括、翻译、润色或重排。可在原文后另写"当时情况"和"Agent 注释", 但必须明确标注为 agent 注释, 不得替代、扩展或冒充用户原文。
2. 可迁移经验: 调参经验, 工程 trick, 失败的教训
3. 被搁置的路线: 之前哪些路线觉得重要性不高/性价比不高被搁置了, 将来有空可以继续做, 或者作为其他路线的参考

保持言简意赅, 并且随着新的知识的加入不断整理合并, 就像对 memory 的操作一样.
不要滥用加粗, 处处加粗等于没加粗.

不要删除本注释, 一直保留作为本文件的填写指引.
-->

- [2026-06-27] **单 gate 深入时需主动同步 multi-gate 框架。** V5-V6 在 marginal_dominance (G3) 上做了 9 个 runs (R80-R88)，深度和 rigor 都大幅提升，但 G2/G4 仍停在 V1-V4 lemma-sketch/appendix-route 水平。Reviewer 因此判为 narrative drift（"half four-gate, half marginal_dominance"）。以后任何 multi-gate theory paper，每完成一个 gate 的 deep dive 后，必须检查其他 gate 是否需要同步 advance 或至少做 scope reconciliation；不能让框架的不同部分出现代际差距。
- [2026-06-27] **"Conservative lower bound" 不是理论 closure。** R85 诚实标注 2-action reduction 是 conservative lower bound，R87 诚实记录 2-action gap。但 reviewer 不接受"诚实标注"替代"gap quantification"。以后任何 reduction/proxy/abstraction，如果它对主 claim 是 load-bearing，必须给出 bound 说明 reduction 保留了什么、丢失了什么、误差多少。诚实 scoping 只是起点，不是终点。
- [2026-06-27] **人类 §5 指令高于 reviewer 路线建议。** V6 reviewer 给出两条路径：(a) reframe to factored PG paper 或 (b) advance other gates。Reframe 会降级 claim（从"FEX discoverability factorization"变为"factored PG convergence"），违反 §5 "不许降级 claim"。选择 (b) 尽管工作量更大。以后 reviewer 反馈与 §5 冲突时，§5 是最高优先级；可以向 reviewer 解释约束但不能违抗。

## 被搁置的路线

- [2026-06-27] **FR-PPO dimension-free PG convergence (2506.03757)**：FR-PPO 的 O(1/n) 收敛率不依赖 action/state space 维度。如果 FEX controller 从 softmax 切换到 Fisher-Rao 几何，G3 scaling 问题在理论上消失。搁置原因：当前 route 聚焦证明"softmax PG 在何种条件下收敛"，切换参数化会引入新的 confounding。未来如果 reviewer 要求"证明 G3 对架构选择不敏感"，这是第一候选方向。
- [2026-06-27] **QOT 插值路径 PL 技术 (2605.27175)**：沿 θ_t = (1-t)θ_* + tθ 分析 Hessian → integral operator 最小特征值 → error bound → PL 的三步推导链。搁置原因：当前 B4 K-PL proof 的优先级低于 G2 theorem completion 和 bridge theorem。如果 reviewer 要求完整的 K-PL proof（而非 conditional PL），这是候选技术路线。
- [2026-06-27] **GCR-PPO multi-head gradient conflict resolution (2509.14816)**：multi-head critic + per-component gradient + 优先级 PCGrad 投影，直接对应 FEX 4-slot 结构。搁置原因：当前路线已从 gradient conflict analysis 转向 marginal_dominance theory，冲突分析不再是 load-bearing。如果未来回到 fullspace FEX controller analysis，这是直接可用的工具。
- [2026-06-27] **GoodRegressor depth-controlled SR at 10^28 (2510.18325)**：depth 饱和窗口概念——不同 PDE 族需要不同 depth。搁置原因：当前 evidence 全部在 depth2_sub，扩展到更深 depth 需要实际 FEX 实验，超出 theory-first 范围。如果 reviewer 要求"证明 depth scaling 的 tractability"，这是 framing 参考。
- [2026-06-27] **R83 K=3 counterexample 从未被 commit**：R83 来自 V5 era (v5-b4-scaling-oracle)，其内容被 R84 (impossibility_proof_v2.md) 吸收。K=3 counterexample (p₁→0.458) 在 R84 §5 和 bridge theorem Theorem 3 中均有引用。未来如需独立 R83 artifact，可从 R84 和 experiment-log 重建，但当前 R84 已充分覆盖其科学内容。

## 可迁移经验

- [2026-06-25] **证据链干净不等于 proof gap 关闭。** R56 已经做到 33 source paths 0 broken，但 R55/R56 仍留下 bridge repair、conflict bound、architecture sensitivity、convergence status 等 proof gaps。以后 evidence index 只能证明 provenance clean；是否能交给 reviewer 仍取决于 load-bearing claim 是否有 theorem/counterexample closure。
- [2026-06-25] **open-gap 表要先去重再决策。** R56 同时继承 R53 和 R55 的 gap 名称，7 个条目实际约 4-5 个独立问题。以后下一轮 scientist 不能按 raw gap count 做战略判断；必须先 deduplicate 成独立 proof obligations，再决定 P0/P1/P2。
- [2026-06-25] **claim-bearing source path 本身就是证据。** R53 的 G3-THEOREM 结论正确，但两个路径写成不存在的目录，audit 判 MAJOR。以后任何 claim ledger 都要先做 path-exists validation；collected artifact 不原地改，另建 corrigendum 并在 STATE/A0 标明 canonical path。
- [2026-06-25] **restricted 正信号不能自动桥接到 fullspace。** R24 product 在 restricted template pairs 上 cf=0.0，但 R28/R50 fullspace 出现 NDMS=0.243 的 destructive conflict。以后 reviewer 要 full-objective bridge 时，必须先用同一套 metric 做 restricted-vs-fullspace reconciliation；不能把 restricted non-conflict 写成 full-policy 机制结论。
- [2026-06-25] **NICE-TO-HAVE run 未执行不能留成模糊 A3 debt。** R47 是等分母 hygiene，R48 已经回答主科学问题；coder round 跳过它但没有说明，audit 判 MAJOR。以后 P1/P2 run 若不再改变 reviewer belief，scientist 下一轮必须明确 proceed/defer/cancel，并从 A3 活跃表移除或写清 root cause，不能让 dispatcher 误以为还有遗漏主线任务。
- [2026-06-25] **绑定判据和诊断画像要分开报告。** R28 的 R36 `thresholded_cf` 绑定值是 final-window median；它能决定 formal PASS/FAIL，但会隐藏 passing seeds 里的间歇性冲突。以后 slot-gradient 证据至少同时报告 median、mean、p90/max、pair-frequency、step-frequency 和 max |cosine|，不能用一个中位数替代现象解释。
- [2026-06-25] **corrected copy 不能替代 stale artifact deprecation。** R43/R44 通过 corrected copies 修复了 R38/R40 的 kappa/G4 错误，R42 也正确读取 corrected package，但 R38/R40 原目录仍含旧错值，audit 仍判 MAJOR provenance hazard。以后只要选择“不原地改 collected artifact”，同轮必须加 DEPRECATED marker 或 manifest redirect，避免未来 agent/reviewer 读到双真相。
- [2026-06-25] **2-action softmax 公式不能外推到 Q-action witness。** R38/R40 把 `kappa_0=4/delta` 用到 Q=100/1000 的 witness 上，R39 重新推导发现一般式是 `Q^2/Delta`，`4/Delta` 只是 Q=2 特例。以后 proof artifact 里凡是从二臂 bandit 推到多 action，都必须单独列 general lemma 和 Q=2 sanity check。
- [2026-06-25] **separation witness 必须让非目标 gate 真的 PASS。** R38 的 G4 witness 用 Q=1 K=0，虽然 coefficient infeasible，但 G2 也同时失败，不能证明 `G4_fail =/=> G2_fail`。以后每个 falsifier direction 都要在 ledger 里写目标 gate FAIL、其他 gate PASS/vacuous 的具体数值，不能靠直觉说"无关"。
- [2026-06-24] **Softmax PG 的全局 PL 要先做边界反例。** R33 把 G3 写成 `J*-J <= kappa||grad J||^2 for all theta`，但 2-action softmax bandit 在最优边界和错误边界都会让 `kappa` 发散。以后任何 PG geometry theorem 都必须先声明是 init-time/local/Lyapunov，或显式限制参数域；proof ledger 要包含边界 counterexample pass，不能只做格式自检。
- [2026-06-24] **§5 人类战略指令高于 reviewer-driven GPU 扩张。** iter-22 audit BLOCKER 指出 V4 plan 忠实追了 V3 reviewer 的 fullspace/K-subset 实验证据，但没有执行 §5 的 theory-first/general-theorem 要求。以后若 reviewer 要更多实验证据而 §5 要一般理论，下一轮必须先把理论对象和 proof obligation 抽象清楚，再决定 GPU 规模。
- [2026-06-24] **Pre-registration 只能有一个绑定阈值源。** R26 同一个 metric_schema/pre_register 里同时保留 calibrated thresholds (0.25/-0.15) 和原计划 thresholds (0.80/0.70)，导致同一结果可被不同字段判成 PASS/FAIL。修复策略：新建 canonical repair artifact；旧 prereg 标 invalid；分析脚本只读 repair schema。
- [2026-06-24] **restricted/proxy G3 diagnostics 不能替代 full-objective bridge。** V3 reviewer 5.8 接受 R20/R21/R24 integrity fixes，但仍判定 restricted 2-arm witness + 3 对 hand-picked templates 不够 submission-grade。G3 下一轮必须同时给 named-assumption proposition（逐项处理 E2 percentile、E3 inner fitting、full product-policy gradients）和 fullspace/larger-K FEX-anchor evidence。
- [2026-06-24] **形式 success criterion 失败就必须写失败，不能用 prose 改成 PASS。** R24 jointnorm medium 的 pre-stated SC2 是 conflict_fraction ≤0.10，实测 cf=0.333，所以 formal verdict 是 FAIL。负 dot magnitude 很小（max |cos|<0.02）可以作为解释和后续 metric 设计依据，但不能把硬判据事后改成通过。
- [2026-06-24] **slot-gradient alignment 有 reward-gap 边界。** R24 product mode 在 large/medium/small 三组都是 cf=0.0，但 jointnorm small_gap cf=0.573 且 max negative |cos|≈0.24，是真冲突，不是数值噪声。以后写 G3 结论必须区分 original product estimator、matched-estimator stress test、以及 small-gap 边界。
- [2026-06-22] **分布集中指标不能自动替代梯度对齐证据。** C(P)→1.0 能说明 policy mass 集中到少数 template，但 FPG 的 non-conflicting condition 是关于 slot-gradient dot products `G_s·G_s' >= 0`。如果用 C(P) 支撑 factored-policy κ 关系，必须额外记录 logit-space gradient、slot-gradient dot matrix、conflict fraction；否则只能写 behavioral architecture finding，不能写梯度对齐结论。
- [2026-06-22] **跨架构比较 grad_norm_sq 必须归一化参数维度。** R20 的 "100x easier" 是 2580 维 MLP vs 2-3 维 tabular 的 ||∇J||² 差异，不是 PG geometry 差异。hat{kappa}=(J*-J)/||∇J||² 中分母随 dim(θ) 缩放，使得不同参数量的模型 hat{kappa} 不可比。跨架构只能用 dimensionless 指标（如 C(P) collision probability、hit_time）或归一化后的 per-parameter gradient norm。Same-architecture cross-dim 比较（如 R3 dim20 vs dim30）不受此影响因为 dim(θ) 相同。
- [2026-06-22] **V0→V1→V2 score trajectory 是最强策略信号。** V1(5.6) 来自 formal proofs (R13-R15, CPU)。V2(5.5) 来自 $97.40 GPU 实验但有 confound。结论：evidence quality >> evidence quantity。修证据比加证据更有效率，尤其当 reviewer 具体指出 confound 时。
- [2026-06-22] Actual-controller ablation 的结果可以是 opposite direction——这比 null 更有信息量。R20 预期 MLP factored controller 比 flat tabular 更难（因共享参数约束动作空间），实际 C(P)→1.0 集中好 template。原因：shared MLP parameters 提供 inductive bias，使 policy 快速集中，不是约束。Pre-stated failure interpretation 应覆盖 opposite-direction 场景，不只是 null。
- [2026-06-22] "Success criterion NOT met" 不等于实验失败。R20 的 opposite direction 反而是最有价值的新发现——architecture-dependent G3 解释了 WHY FEX works。写 success criterion 时应该同时写 "opposite-direction interpretation"。
- [2026-06-21] Reviewer 若指出 "generic bandit witness 不够 method-specific"，继续堆同类 trace seed 不是 closure。下一轮必须至少有一个 actual code-path ablation（保留 percentile/top-k loss、inner optimizer、同一 controller representation）或一个能精确说明 theorem ceiling 的形式化结果。
- [2026-06-21] FEX controller 的 G3 诊断必须记录 slot-level 概率结构，而不只是 aggregate `grad_norm`。对 shared-MLP/factored-slot controller，`C(P)=sum_a pi_a^2`、per-slot entropy、joint-template probability、top-percentile membership、joint-vs-factored control 是 reviewer 能区分 "generic softmax" 与 "FEX-specific representation" 的最小证据。
- [2026-06-21] Theory-first route 里，diagnostic strengthening 不能替代 theorem closure。R7-R12 把 G3 证据做得更强，但 reviewer 仍给 4.5/10，因为主 separation proposition 没有 formal statement/proof。以后若 proof obligation ledger 仍是 STRONG_DIRECTION / CONDITIONAL+DIAGNOSTIC，不能把更多 seed 当作 closure。
- [2026-06-21] 机制概率必须区分 observed-sample distribution 和 expected-policy distribution。R12-B multiplicity 只覆盖约 400/1539 个 observed classes；reviewer 要的是 full 1539-class expected-policy 或 causal ablation。任何"policy-level mechanism"都必须说明概率分布来自完整 policy、采样估计，还是 observed subset。
- [2026-06-21] Observed-sample vs expected-policy multiplicity 可差 6x（+25.7% vs +4.0%）。差异来自采样 concentration bias（policy 只 visit ~400/1539 classes，在这些 class 上 multiplicity effect 更大）和 factorized slot-marginal 模型的平滑效应。论文必须同时报告两个数字，不能只报看起来更强的 observed-sample 数字。
- [2026-06-21] "All seeds pass" 不随 sample size 缩放。R11-C n=4 时 4/4 全 >10%；R12-B n=16 时 14/16 >10%（1 负值 1 below 10%）。小 n 下 "all pass" 是选择偏差；论文机制 claim 必须用 CI 不能用 "unanimity"。
- [2026-06-21] 机制 outlier 先查 explanatory covariate 再作 limitation。seed19 +81.4% 不是 bug——m2_pi=0.582（最高），r(m2_pi, effect)=0.437。把 outlier 归因于已知 mechanism covariate 比单纯报 robust CI 更有说服力。
- [2026-06-21] 偶数窗口的 median convention 是 evidence contract。R10-A/R10-B 对同一 final-50 `reward_variance` 差 5-7%，根因只是 upper median vs standard median；所有 analyzer、validation、bootstrap、LOO 必须共用一个命名 helper，并在 manifest 里写清楚。
- [2026-06-21] 百分比分解不能只靠 median-ratio identity。`log(hk_ratio)` 用 median ratios 拆 `gap/rv/geometry` 会留下 residual（R10-B 为 5.8%，R11-B 证明是 median-ratio non-additivity artifact）。Load-bearing 百分比必须用 paired-log exact identity（residual=0 by construction）。
- [2026-06-21] 机制 witness 要区分 init-time、post-hoc convergence proxy 和真实 on-policy logging。R9-C init-time +10.5% → R10-C post-hoc proxy +1.6% → R11-C real on-policy +20.5%。Post-hoc proxy 偏差 12.8x，因为模型假设 softmax 覆盖全部 2187 模板但实际 trained policy 只覆盖 ~400/1539 类。结论：量化 mechanism 必须用真实 per-sample data，不能用 entropy-calibrated analytical reconstruction。
- [2026-06-21] 机制分解要区分“量级 canonical source”和“decomposition canonical source”。本项目 R3 longtrace 更适合报 `hat{kappa}` 量级，R9-B probe 才有 `reward_variance` 字段可做分解；不能把两组独立 stochastic runs 的点估计混成一个数字。
- [2026-06-21] normalized diagnostic 的点估计低于 1 不等于机制闭合。n=5+5 时 ng2 CI 含 1（0.724 [0.548, 1.070]），必须补 seed。扩到 12+12 后 ng2=0.783 [0.621, 0.924] CI 排除 1.0，exact ng2=0.594 [0.495, 0.715] 更强。统计功效比精确点估计重要。
- [2026-06-21] 聚合方法选择影响量化但不影响方向。Exact paired-log mean（rv 64.8% + geo 33.6%，residual=0）vs median-ratio（rv 70.7% + geo 21.4%，residual 5.8%）。12pp geometry share 差距来自 mean 对高 hk 尾巴的放大（dim30 seed3=26923，7.2x median）。选 exact 作 canonical（zero residual），median 作 robustness check。
- [2026-06-20] PG geometry 诊断里，`grad_norm` 变小不能直接解释成 landscape flat。REINFORCE 梯度同时受 reward variance 和 reward gap 影响；必须报告 `reward_variance`、`mean_reward`、`reward_gap`、variance-normalized `grad_norm_sq`，且 n=2 bootstrap 只能当 confound candidate，不能当定论。
- [2026-06-20] Trace/analyzer 字段名是 evidence contract。分析脚本读取 `entropy`/`logit_std` 但真实 JSON 写 `slot_entropy_mean`/`logits_std` 时，不能静默输出 N/A；必须断言 expected fields 的非空覆盖率，或把不可用原因写进 manifest。
- [2026-06-20] 远端 GPU run 被判“stuck”前必须同时查 screen、父子进程、GPU process、remote output mtime 和 per-run JSON。仅凭本地 `runs/` 不存在会误判：本项目 R7-C 本地无 runs，但远端已完成 dim20 seed0/1、dim30 seed0 且 dim30 seed1 正在跑。
- [2026-06-20] 分析脚本如果 hardcode run source list，必须同时写 denominator assertion（例如 `n_dim20_converged=5`）和 rerun receipt。否则后完成的 seed 会被 summary 漏掉，旧 factor/CI 数字看似可复现但实际 stale。
- [2026-06-20] Exit-evidence smoke test 不能只测 wrapper 字段。凡是 claim-bearing 的结果，smoke 必须同时断言核心数据字段没有被后处理删除；本项目里就是 `pg_trace` 和 `reward_curve`，否则 `returncode/json_status` 全通过也会丢掉 G3 证据。
- [2026-06-20] JSON 结果文件默认按 strict JSON 处理。Python `json.dump` 默认会写裸 `NaN`，这不是 RFC 8259 合法 JSON；run wrapper 和 analyzer 必须先把 NaN/Inf 转成 `null` 或其他显式合法值，再用 `allow_nan=False` 写出。
- [2026-06-20] 只在 sweep-level summary 写 `returncode` 不够。长实验的 per-run JSON 必须自包含 exit evidence（`exit_status`、`returncode`、`timed_out`、`json_status`、`log_tail`、`log_sha256`、`cmd`）；否则 analyzer 或后续 agent 只读 per-run 文件时会把异常退出误当普通 partial。
- [2026-06-20] 子进程 JSON 可能损坏或截断，`json.load()` 本身必须在 try/except 内。否则 child process 已经返回的 returncode 会在 wrapper 后处理阶段丢失，形成“日志静默结束、summary 无 exit cause”的最坏证据链。
- [2026-06-19] `subprocess.run(check=False)` 会把非零退出伪装成正常完成，除非显式记录 `returncode`。长 GPU sweep 的 `summary.json` / manifest 必须登记 child return code、timeout flag、log tail 和 exit status；否则 `timed_out=false` 不能说明 run 正常结束。
- [2026-06-19] 并发远程 wrapper 不能 `tee` 到同一个 `train.log`。即使 per-run raw logs 完整，共享 sweep log 也会交错污染；每个 GPU/session 用独立 wrapper log，claim-bearing evidence 只引用 per-run logs、summary、manifest。
- [2026-06-19] `pg_trace` 覆盖率通过不等于 PG geometry 证据可信。若高维 run 只跑到早期 step 或 timeout，`hat{kappa}`/gradient summary 必须按 trace window 分层报告；跨维度比较必须保证 window 相同或明确写成 runtime/search-scaling 信号。
- [2026-06-19] 通过 coefficient fitting 后得到的 good-class count 是 parametric K，不是 structural K。所有 exposure / blind-hit-time 结论必须标明 K 的定义：fixed coefficients、global scale+bias，还是 full inner optimizer。
- [2026-06-19] Analyzer 在已同步子集上 pass 不等于 run 达到原 success criterion。若计划 denominator 是 20，但 manifest 只有 15/20，A3 必须是 `needs_sync` / `needs_fix` / `running` 之一，不能标 `collected`；§4 只能写 partial diagnostic。
- [2026-06-19] Quotient count 一旦发现 rewrite set unsound，所有下游使用该 Q 的 sampler bound、bandit sweep、claim table 都要标 stale 并重跑。不要把“syntactic artifact 可复现”写成“semantic/sound quotient 证据”。
- [2026-06-19] 若 workspace 已有 pilot git commits，但 `experiment-log.md` 只有模板占位、`STATE.md` 仍是模板，则按实验工厂初始化处理：先把 pilot 基线规范到 `main`，再从 `main` 开 route 写首轮 STATE/A1。
