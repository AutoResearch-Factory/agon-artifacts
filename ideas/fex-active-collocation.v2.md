---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-active-collocation/
---

- One-sentence summary: 将 FEX 的候选表达式缓存看成 query-by-committee, 用候选间 PDE 残差分歧选择少量 collocation 点, 并检验这种采样何时降低 reward 噪声、何时因为 reward 分布变化太快而拖慢结构搜索.
- Hypothesis: QBC active collocation 只有在 reward 估计噪声是瓶颈、且 PDE 有局部尖峰/激波/边界层时才可能有用; 对 smooth Poisson 这类全局简单解, FEX 的瓶颈是先找到正确表达式结构, 快速切换 hard set 反而会让同一候选在不同 step 得到不可比的 reward.
- Expected outcome: 成功不是单个 Poisson 加速, 而是在至少一个局部特征 PDE 上用更少 wall time 找到同等质量表达式, 幅度超过 25%, 且全局验证误差不变坏; 失败信号是等计算预算下仍不快于 uniform, 或 active hard set 提高 hold-out residual. 当前 Poisson pilot 已给出最便宜的负面信号: active refresh=5 比 standard 晚 5 个 RL step 才过 `score > 0.99`.
- Contribution type: method + diagnostic
- Risk: MEDIUM
- Estimated effort:
  - Compute: 40 GPU-hours for 3 PDE families × 3 seeds plus refresh/EMA ablations
  - Data: available
  - Implementation: 1-2 weeks
- Novelty quick-check: RL-PINNs, PINNACLE, Aikawa et al., DCM, PACMANN 和 vRBA 都研究 PINN/神经 PDE solver 的 adaptive collocation; DISCOVER 和 SSDE 做 RL/symbolic PDE search, 但没有把候选表达式之间的分歧用于采样. 本工作的差异是利用 FEX search buffer 里已经存在的多个候选表达式, 让候选间残差分歧直接服务于 expression-tree reward 估计, 并专门测量 reward 非平稳性是否抵消收益.
- Strongest objection: 如果 pilot 已经在 Poisson 上失败, 这可能只是一个不会工作的采样 trick; 只有当局部特征 PDE 或 EMA hard-set 消融显示清楚边界时, 它才值得作为 paper idea 保留.
- Why we should do this: FEX 搜索效率的瓶颈目前常被概括成 "RL search hard", 但缺少把 reward 噪声和结构探索分开的实验. 这个 idea 成本低; 正结果给出一个可插拔采样模块, 负结果也能说明 active collocation 不是 smooth high-dimensional FEX 的主杠杆.
- Pilot:
  - Setup: 在 Poisson `d=10` 上复现 stock FEX 的搜索阶段; standard 每步均匀重采样 5000 个 interior points, active 用 80% uniform + 20% top-5 候选残差分歧 hard set, 对比 refresh=50 和 refresh=5, seed 0, GPU 运行.
  - Metric: 主指标是达到 `score > 0.99` 的 RL step 和 wall time; 正信号要求 active 在同等计算口径下减少 >30% step, 且 hold-out 全局误差不变坏.
  - Result: standard 在 step 18 过阈值, final best error `5.12e-06`, 648s; active refresh=50 也在 step 18 过阈值, 但首次 refresh 发生在 step 50, 实质没有作用且总时长 1036s; active refresh=5 在 step 23 过阈值, final best error `2.13e-06`, 608s.
  - Signal: NEGATIVE

- Claims and Claims matrix: Paper 级主 claim 是 "candidate-disagreement collocation 是 FEX reward 估计的可检验干预, 其收益取决于 reward noise 是否真是瓶颈." POSITIVE 时可 claim QBC active collocation 在局部特征 PDE 上用更少 wall time 找到同等质量表达式; MIXED 时只能 claim 它区分了 smooth/global 与 localized PDE 的采样需求; NEGATIVE 时 claim FEX 在这些任务上的主瓶颈是结构探索而非 collocation coverage, 且快速 changing hard set 会制造 reward 非平稳性.
- Narrative: 不把故事写成 "把 active learning 用到 FEX". 更好的故事是: FEX 已经免费产生一个 candidate committee, 我们用它测试哪些 collocation 点真的能分辨表达式, 再用失败案例约束 FEX score 设计.
- Experiments: 先做 Poisson 多 seed、equal-compute uniform、static hard set、EMA hard set、random hard set 和 residual-only hard set; 再做 Burgers/Allen-Cahn/conservation-law 这类局部残差更集中的 PDE; 每个实验同时报告 search step、wall time、fixed-candidate reward variance、controller entropy、hold-out residual 和最终 relative L2. Proposal 阶段再决定是否加入 PINN adaptive baselines, 但不能把本文扩张成 benchmark paper.
- Assets status: FEX code、Poisson pilot artifacts 和数据交接已记录在 `workspace/fex-active-collocation/data/MANIFEST.md`; 无外部数据下载异常.

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: RL-PINNs (Song 2025, arXiv:2504.12949) — DQN-driven adaptive collocation for PINN training; DISCOVER (Du, Chen & Zhang 2022) — RL-based symbolic PDE discovery with fixed uniform sampling; Aikawa et al. (2023, New Generation Computing) — dropout-based ensemble uncertainty for PINN collocation; PINNACLE (Lau et al. 2024, ICLR) — NTK-based joint collocation optimization; Deep Collocation Method (Weng, Mao & Shen 2025) — residual-guided greedy subspace collocation.
- Key differentiator: 将 QBC (query-by-committee) 原则与 RL 表达式树搜索结合, 利用 FEX search buffer 中自然存在的候选 expression committee 进行 collocation 点选择——这一组合在 deep-lit (8 篇 PINN 自适应采样精读 + B7 反向扩展) 和今日补充搜索中均未发现任何重叠. 新颖性本质上是增量式的交叉组合 (QBC + expression tree search + PDE collocation), 而非机制突破. Pilot NEGATIVE 结果部分验证了新颖性——如果此事显而易见或已知有效, 早就有人做了.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v1 的核心 gap ("候选间分歧是否真与区分性相关?") 因 pilot 执行而部分闭合: 初步证据表明 naive QBC 在 smooth PDE 上不工作, 与 refined hypothesis 一致. 但因果链仍有多处未验证: (a) 候选残差分歧与表达式区分性的相关性未直接测量; (b) EMA hard-set 仅停留在提议层面, 未测试; (c) 缺乏对"why localized PDE specifically benefits QBC (vs 仅受益于更高 variance 的 uniform 估计)" 的机制论证. |
| Missing evidence signals | 5/10 | v2 大幅扩展了实验计划 (multi-seed / 多 PDE / EMA / static / random / residual-only ablation). 但关键证据仍缺失: (a) fixed-candidate reward variance analysis——最直接检验 "reward noise 是瓶颈" 的实验; (b) controller entropy 追踪——区分 "active collocation 帮助区分候选" 与 "active collocation 重塑 reward landscape"; (c) QBC-selected 与 residual-selected 点集的重叠度分析——若重叠 >80%, QBC 相对于 residual-based 无增量信息. Pilot 仅单 seed, 计划中的 multi-seed 已承诺但未执行. |
| Narrative | 7/10 | v2 明确采用了 v1 review 建议的 QBC framing, 并在 Narrative 段显式拒绝 "把 active learning 用到 FEX" 的弱叙事. 当前故事线——"FEX 已经免费产生 candidate committee, 用它测试哪些 collocation 点真的能分辨表达式, 再用失败案例约束 FEX score 设计"——比 v1 锐利得多. 微瑕: 仍未充分解释 WHY QBC 在表达式树搜索中应比 residual-based selection 更有效, 而不仅仅是 "it's free because FEX already has candidates". |
| Venue contribution | 5/10 | 按 FEX 发表历史 (JMLR, JCP), 作为 systematic diagnostic + method paper 有合理发表空间. Pilot NEGATIVE 反而增加了 diagnostic framing 的 credibility——研究不是 cherry-pick 正面结果. 但对 NeurIPS/ICML 仍不够: 贡献是 niche method family 的模块化插件, 缺乏可迁移到更广泛 ML 社区的洞见. 若 Burgers/Allen-Cahn 上 EMA hard-set 产生反直觉发现 (如 "QBC 改变了 controller 收敛到的表达式结构而非仅加速"), contribution 可升至 6/10. |
| Testability | 8/10 | Pilot 已执行, NEGATIVE signal 清晰 (active refresh=5 比 standard 晚 5 步过阈值). 后续实验矩阵结构良好: PDE 类型 (Poisson→Burgers→Allen-Cahn→conservation-law) × hard-set 策略 (uniform/static/EMA/random/residual-only) × 3 seeds. 最便宜证伪信号明确: 若 EMA hard-set 在 Burgers 上等计算预算下不优于 uniform, method contribution 即告死亡. 提升点: 建议加入 fixed-candidate reward variance sweep 作为 pre-PDE 诊断, 用 <5 GPU-hours 即可判定 reward noise hypothesis 是否成立. |
| Outcome realism | 7/10 | Pilot 提供了现实校准: Poisson 上已确认 NEGATIVE. POSITIVE 现在锚定在 localized PDE (Burgers/Allen-Cahn) + EMA hard-set 组合上, 这是合理的转移——这些 PDE 的残差分布更不均匀, 理论上自适应采样收益更大. NULL 和 NEGATIVE 的 claim 边界清晰, 且都有发表价值. 唯一 concern: 若所有 PDE + 所有策略全部 NEGATIVE, diagnostic story 的核心 insight ("reward non-stationarity 是主因") 可能不够深, 难以支撑独立论文. |
| Contribution type compliance | n.a. | topic 未声明 preferred-contribution-types, 跳过检查. |
| Overall Quality | 6/10 | v2 在 v1 基础上显著进步: pilot 执行提供了实证锚点, narrative 从 heuristic trick 升级为 QBC 原则验证, experiment plan 从单 PDE/single-seed 扩充为系统性消融矩阵. 核心短板仍是贡献深度——这是一个 within-FEX-community 有价值的诊断研究, 但对外部审稿人而言, 回答 "adaptive collocation 对 FEX 有用吗?" 比回答 "QBC 原则在表达式树搜索中揭示什么?" 更窄. 建议下一步实验聚焦 diagnostic depth (reward variance decomposition / entropy tracking / QBC-vs-residual point overlap) 而非仅追求 method acceleration. |

## Contribution Drift

- v1 contribution types: {method}
- v2 contribution types: {method, diagnostic}
- Status: expanded(+diagnostic)
- Hard cap triggered: no — topic 未声明 preferred-contribution-types. 扩展有据: pilot NEGATIVE 自然导向 diagnostic framing, 且 v2 body 中 hypothesis 和 expected outcome 均反映了这一扩展.

### v1 Review Concern Resolution

| v1 Concern | Status | Notes |
|-----------|--------|-------|
| Logical gaps (因果链未严格论证) | Partially resolved | Hypothesis 被 refine, 但因果链 (disagreement→informativeness→variance→convergence) 仍未被拆分验证 |
| Missing evidence signals (缺 ablation / dimension scaling / epsilon-greedy 交互) | Partially resolved | Experiment plan 大幅扩展; fixed-candidate variance / entropy / point-overlap 分析仍缺失 |
| Narrative ("apply adaptive sampling to FEX" = 弱叙事) | Resolved | QBC framing 被采纳, 反 "apply X to Y" 叙事显式写入 Narrative 段 |
| Venue contribution (30% 迭代减少不够顶会) | Partially resolved | Pilot NEGATIVE 增加了 diagnostic framing 的可信度, 但贡献深度未根本改变 |
| Testability (单 seed / 单 PDE) | Resolved | Pilot 已执行; 多 seed/多 PDE 已纳入实验计划 |
| Outcome realism (aggressive 30% threshold) | Partially resolved | Pilot 提供了现实校准; POSITIVE 锚定到 localized PDE |

Refiner pushback: refiner 采纳了 Alternative Framing (QBC), 执行了 pilot, 将 risk 从 LOW 提升到 MEDIUM (反映 pilot NEGATIVE 后的务实评估). 所有改动均有据, 无 concerning pushback.

## Alternative Framing

当前 framing 已是 v2 改进后的版本 (QBC for expression tree search), 属于 sharp 方向. 可进一步锐化为: **"Collocation as a test of reward design: when does QBC disagreement predict which expressions will succeed?"** —— 将研究问题从 "does QBC accelerate FEX?" 转向 "what does the (in)effectiveness of QBC reveal about FEX's reward surface geometry?" 这一 framing 将 diagnostic contribution 前置为 paper 的主贡献, method contribution 退为 diagnostic 的自然产物. 对 JMLR/JCP 投稿, 这比 "30% acceleration on localized PDE" 更具知识贡献深度.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | EMA-stabilized QBC active collocation 在至少一个局部特征 PDE (Burgers 或 Allen-Cahn) 上用 ≥25% less wall time 达到与 uniform 同等表达式质量, 且 hold-out 全局误差不退化; 优势来源于 EMA 抑制了 reward non-stationarity, 使得 candidate disagreement signal 得以在 reward 噪声真正是瓶颈的 PDE 区域发挥作用. |
| NULL | QBC active collocation 在所有测试 PDE 上均未显示 >15% 的 wall-time 优势, 但 revealed a systematic relationship between PDE residual concentration and collocation sensitivity; 这一 null finding 量化了 "FEX reward estimation is robust to collocation point choice under the tested budget (5000 points)", 为 FEX community 提供了关于采样预算分配的经验准则. |
| NEGATIVE | Active collocation (包括 EMA 变体) 在所有测试 PDE 上均显著慢于等计算预算的 uniform; fixed-candidate reward variance 分析证实 reward noise 不是当前 FEX 配置的瓶颈, 且快速 changing hard set 引入的 reward non-stationarity 系统性损害 controller 收敛. 这一发现表明 FEX 搜索的主瓶颈是 structure exploration (找到正确的算子拓扑) 而非 reward estimation precision——为 FEX 社区的 effort allocation 提供了证据驱动的优先级排序. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - **Likelihood = Medium**: 有明确的实验路径 (FEX 代码库可用, 3 PDE families × 3 seeds + EMA ablation ≈ 40 GPU-hours), 且 pilot 已提供 ground-truth calibration. 但做成 top-venue 级结果依赖: (a) EMA hard-set 在 Burgers/Allen-Cahn 上显示 statistically significant advantage over uniform under equal compute (pilot 表明 naive 版本失败, EMA 是关键未验证假设); (b) diagnostic depth 足够支撑独立发表——若所有 PDE 均 NEGATIVE, "non-stationarity dominates" 的 insight 可能过薄; (c) fixed-candidate reward variance 分析确认 reward noise 在 localized PDE 上确实高于 smooth PDE. 三个条件均合理但未验证, 且 pilot NEGATIVE 增加了 method-success 的不确定性. 不降为 Low 因为: (i) 实验路径清晰且成本可控; (ii) 即使 method 失败, diagnostic framing 仍有发表可能; (iii) localized PDE 相比于 smooth Poisson 在理论上对自适应采样更友好, EMA 机制在邻近领域 (变分框架 2509.14198) 已有成功先例.
  - **Impact = Medium**: 在最乐观情形 (EMA-QBC 在 localized PDE 上 ≥25% wall-time reduction): 提供一个可插拔采样模块, 验证 QBC 原则在 expression tree search 中的适用性, 并以 systematic ablation 建立采样策略选择的经验准则. 对 FEX 研究线有清楚推进, 但不会改变更广泛 ML/PDE 社区的判断——QBC 在 active learning 中的有效性是已知的, expression tree search 的 reward surface 特性不构成 paradigm-shifting discovery. 值得注意: NEGATIVE outcome 的 impact 可能高于 POSITIVE——若证明 "FEX reward estimation is robust to collocation noise, bottleneck is structure exploration", 会对 FEX 社区的 effort allocation 产生方向性影响, 且 "QBC fails on expression tree reward surfaces" 本身是一个关于 RL reward design 的有趣 negative result. 但这一 impact 仍然局限于 FEX/SR niche, 不会改变更广泛社区的研究路线.
  - Codex second opinion: codex 再次陷入非相关搜索循环 (query "query by committee" 返回 multiwinner voting / Roman telescope / diabetic retinopathy 等完全不相关文献), 未在时限内产出结构化 review. 本 review 的 Likelihood/Impact 判定完全基于 Claude 独立评估, 与 v1 review 保持一致.

## Overall

- Priority: Medium
- Score: 5
- Comments: v2 相较于 v1 在三个关键维度进步显著: (1) pilot 执行提供了实证锚点, 将 idea 从纯猜想推进到有数据校准的假设; (2) narrative 从 "apply X to Y" 升级为 QBC 原则在 expression tree search 域的实例化检验; (3) experiment plan 从单条件扩展到系统性消融矩阵. 当前 score 保持 5 是因为核心结构性问题未变——贡献深度局限于 FEX niche, 对外部审稿人而言 "自适应 collocation 对 FEX 有用吗?" 比 "FEX reward surface 的几何性质揭示什么?" 更窄. 建议: 下一步实验优先跑 (a) fixed-candidate reward variance sweep on Poisson+Burgers (diagnostic depth), (b) EMA hard-set on Burgers (method test). 两个实验合计 <15 GPU-hours, 可在一个 iteration 内完成, 为下一版 review 提供关键的 empirical differentiation signal.

</review>

## Deep Lit Addenda [deep-lit-tick, 2026-06-16]

Scope: `--scope idea 0616-fex --idea fex-active-collocation`。1 round, 7 axes × ≥1 query, 6 wiki_written + 2 tex_download_failed。B7 反向扩展 24 次调用，产出 0 篇新增 2025+ 候选 → **文献饱和**。核心结论：自适应 collocation 在 PINN 文献中已被穷尽研究，但 **没有任何论文将 QBC（候选间分歧）与 RL 表达式树搜索结合**——本 idea 的 novelty 确认。

### 6 篇精读论文

- **2603.06761** — Diversity-Aware Adaptive Collocation via Sparse QUBO & Hybrid Coresets (Salloum et al., 2026.03) [wiki](wiki/2603.06761.md): 将 PINN collocation 点选择构造为固定预算 coreset 问题，用稀疏 BQM + kNN 相似图 + coverage anchors 同时优化点的信息量和多样性。在 1D viscous Burgers with shock 上 Hybrid BQM 达到 38% wall-time 加速。**对 idea 的价值**: coreset 的 diversity+informativeness 双目标与 QBC 的"候选间分歧选点"在目标函数层面高度平行，但面向 PINN 连续神经网络而非离散表达式树。QBC 可以被理解为一种隐式的 diversity 机制（不同候选在不同区域分歧 = 自然的 diversity 度量）。应引用为 adaptive collocation 中 diversity 维度的 state-of-the-art baseline。

- **2504.09804** — BO-SA-PINNs: Self-adaptive PINNs with Bayesian Optimization (Zhang, Li, Lanteri et al., 2025.04) [wiki](wiki/2504.09804.md): 三阶段流程——Bayesian optimization 超参搜索 → EMA 动态 loss 权重 + RAR-D 自适应采样 → L-BFGS 精修。在 Helmholtz/Maxwell/Burgers/高维 Poisson 上全面优于 SA-PINN baseline。**对 idea 的价值**: EMA loss weighting 是本 idea "EMA hard-set" 提案的直接 prior art——EMA 被证明可以有效稳定自适应训练动态，但应用在 loss weight 而非 collocation set。将 EMA 从 loss weight 迁移到 hard-set refresh 是合理但未被测试的扩展，可引用为 EMA 在 PINN 自适应训练中有效性的证据。

- **2605.30910** — PINNs Failure Modes are Overfitting (Andersen & Matsubara, 2026.05) [wiki](wiki/2605.30910.md): 通过直接可视化 PDE 残差证明 PINN 失败模式不是优化困难，而是 collocation 点过拟合——train loss 和 test loss 经典发散，残差仅在配点处被最小化。提出 double PINN（对 PDE/BC/IC 残差分别施加输入梯度正则化），在 Convection/Reaction/Wave/Allen-Cahn 上用最多 23× fewer collocation points 达到 SOTA。**对 idea 的价值**: 这直接支持 idea 的 diagnostic framing——"collocation overfitting" 是 PINN/神经 PDE solver 的已知 failure mode，但它对应的是 FEX reward estimation noise 还是 structure exploration？本文的 double backpropagation 范式可被类比为 FEX 的 "reward regularization"。若 FEX 的瓶颈确实是 reward 过拟合，QBC 可能通过改变 collocation 点分布来缓解；若瓶颈在 structure exploration，则 QBC 无用。这是 idea paper 讨论中最有力的外部证据之一。

- **2511.05452** — Self-adaptive weighting and sampling for PINNs (Chen, Howard & Stinis, 2025.11) [wiki](wiki/2511.05452.md): 将 BRDR/IRDR 动态点权重与 residual-clipped probability distribution 采样合并。核心发现：单独 adaptive sampling 或 adaptive weighting 都不够，组合方法更稳定（<20% 训练开销）。在 perturbation/Allen-Cahn/Burgers/lid-driven cavity 上验证。**对 idea 的价值**: "单独自适应不够"的发现与 pilot NEGATIVE 信号一致——naive QBC 失败可能是因为只改变了采样分布但未调整 reward weighting。这为 EMA hard-set + reward EMA smoothing 的组合消融提供了外部依据。

- **2501.07700** — QR-DEIM Adaptive Collocation for PINNs (Celaya, Fuentes & Riviere, 2025.01) [wiki](wiki/2501.07700.md): 用 residual snapshot matrix SVD/randomized SVD 抽取低秩残差动态，QR pivot 选代表性 collocation points。在 wave/convection/Allen-Cahn/Burgers 四个 benchmark 上达到最低或接近最低 L2 error。**对 idea 的价值**: QR-DEIM 是 reduced-order modeling 视角的 collocation 选择，与 QBC 的 ensemble 视角正交但互补——QR-DEIM 利用残差的低秩结构，QBC 利用候选间的分歧结构。可作为对比 baseline 讨论不同选择原则的适用场景。

- **2606.06314** — DAS-PINNs for high-dim spacetime PDE (Singh & Silvester, 2026.06) [wiki](wiki/2606.06314.md): 将 DAS-PINNs 扩展到 d+1 维时变域，用 KRnet normalising flow 学习 PDE residual 诱导的采样密度。在 moving Gaussian/rotating peak/2D Burgers/6D-8D parabolic Gaussian 上显著优于 uniform（d=8 时 uniform 几乎失效）。**对 idea 的价值**: normalizing flow 的采样范式与 QBC 完全不同（flow 学习残差分布 vs QBC 利用候选分歧），但验证了高维下自适应采样有巨大收益——这间接支持 idea 中 "localized high-dim PDE 上 QBC 可能有用" 的 hypothesis，因为高维下 uniform 采样效率极低。

### 2 篇下载失败

- **2507.09757** (EDRAS for Allen-Cahn): arXiv tex 源码 404，无法获取全文
- **2502.07425** (Foundation PINN with Active Learning): arXiv tex 源码 404，无法获取全文

### Novelty Confirmation

| Claim/Dimension | 状态 | 证据 |
|----------------|------|------|
| QBC + RL expression tree search 组合 | 🟢 完全空白 | 6 篇 PINN adaptive collocation 精读 + 24 次 B7 反向扩展均未发现 overlap |
| EMA hard-set 稳定性机制 | 🟡 PINN 文献有 EMA loss weighting 先例 (2504.09804)，但无 EMA collocation set 先例 | 迁移到 collocation set 是合理扩展 |
| Diversity + informativeness 双目标 collocation | 🟡 PINN 文献有 coreset 方法 (2603.06761)，但无 committee-based 方法 | QBC 可被视为隐式 diversity 机制 |
| Collocation overfitting vs reward non-stationarity | 🟡 2605.30910 证明 PINN 失败 = collocation 过拟合，但 FEX 的 RL reward surface 是不同问题 | 提供类比但非直接证据 |
| 高维自适应采样收益 | 🟡 2606.06314 证明 d=8 下 uniform 失效，但 FEX 高维 (d=100) 采样的瓶颈可能不同 | 支持 localized high-dim PDE hypothesis |

### 对实验计划的影响

本次 deep-lit 建议在现有实验矩阵中增加两个对照条件：

1. **Residual-only baseline**（与 2511.05452 的 residual-clipped sampling 对齐）：QBC 选择的点与纯 residual-based 选择的点重叠度分析——若 >80%，QBC 无增量信息
2. **Diversity-solo ablation**：仅用候选间分歧选点（不加 EMA），与 2603.06761 的 diversity-aware coreset 形成对比——验证 committee diversity 是否是有效 proxy

这两个对照条件不增加额外 GPU-hours（可在已有 pilot 框架中实现），但为 diagnostic depth 提供关键的 comparative baseline。
