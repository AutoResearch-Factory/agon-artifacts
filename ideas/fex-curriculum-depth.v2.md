---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-curriculum-depth/
---

- One-sentence summary: 把 FEX depth curriculum 改成 transfer-tested top-k soft locking: 浅层阶段保留一组候选表达式, 用很小的下游嵌入预算检查它们能否继续组合, 再把通过检查的模块作为下一深度 controller 的软先验, 而不是硬锁一个贪心赢家.
- Hypothesis: FEX 深树搜索的难点在于 controller 很难先发现可复用中间结构, 仅靠树数量的指数增长解释不够. 对有组合结构的 PDE 解, "浅层搜索 + 下游 transfer probe + top-k soft locking"应当在相同候选评估或 wall-clock 下比 flat depth-L FEX 更快达到同等 residual/relative-L2. 如果浅层高分表达式只是短路近似, transfer probe 应在硬锁造成损害前暴露出来.
- Expected outcome: 成功信号是在 Poisson、conservation law、Schrodinger 与制造的 compositional/non-compositional PDE 上, top-k soft locking 比 flat FEX、原始 FEX expanding tree、warm-start without locking、greedy hard locking 和 random locking 少用 >=50% 候选评估而达到相同 error, 且 transfer score 能预测模块后续收益. 最便宜的反证是 equal-budget depth-3/4 测试中 top-k soft locking 不能超过 depth1-only 或 warm-start, 说明收益只是浅层早停而非可组合迁移.
- Contribution type: method
- Contribution drift note: v1 contribution type 是 `method`; v2 仍是 `method`, 没有删除或新增 contribution type. 诊断实验用于约束方法 claim, 不把本文改写成 benchmark 或 application.
- Risk: MEDIUM-HIGH
- Estimated effort:
  - Compute: 60-120 GPU-hours for controlled FEX sweeps; pilot code already runs on a single RTX 4060 Ti
  - Data: available; manufactured PDE collocation is generated on the fly
  - Implementation: 2-4 weeks
- Novelty quick-check: SPL (Sun et al. 2022/ICLR 2023) 已有 MCTS module transplantation, 但它处理的是 MCTS 搜索, 不是 FEX 的 policy-gradient controller 或 PDE residual. 原始 FEX expanding tree 从浅到深跑, 各深度独立, 没有跨深度模块迁移. VSR-DPG (2402.00254) 沿变量轴增量扩展, FePySR (2605.12704) 先抽取神经特征再交给 PySR; 两者都支持"先简化搜索空间"这一直觉, 但没有覆盖 FEX 的 depth-axis transfer probe.
- Strongest objection: 浅层 reward 高的表达式可能只是对最终解的短路近似, 不是正确子模块; 如果直接硬锁, 后续深度会继承错误结构.
- Why we should do this: v1 的 Poisson pilot 显示 shallow-first 能以极小预算找到好表达式; v2 radial-sine pilot 又显示硬锁机制很脆弱. 这把问题收窄成一个更值得写的方法问题: FEX 能否在不贪心冻结的情况下识别可迁移子表达式.
- Review response decisions: Accept novelty critique: 本文不再声称"curriculum learning for FEX"本身新, 而把贡献放在 FEX/PDE/RL 场景下的 transfer-tested soft module frontier. Pushback on "SPL/FEX already cover it": SPL 是 MCTS reward-driven transplantation, 原始 FEX 各深度无迁移, 都没有 FEX controller 的下游 transfer probe. Accept logical-gap critique: 删除"指数分解成多项式"主张, 改为可测试的 optimal-substructure 条件. Accept evidence critique: 正式实验加入 equal-budget、多 seed、depth1-only、warm-start、hard/top-k/random locking 和 compositional vs non-compositional PDE. Accept narrative critique: 叙事改为"FEX search 是否有可利用的组合迁移结构". Accept outcome-realism critique: pilot 只支持小预算 sample efficiency, 不支持绝对能力提升.
- Pilot:
  - Setup: 原 Poisson d=10 跑标准 depth3 40 iter vs depth1->2->3 curriculum 8/8/8, 4 seeds; v2 又加入 radial-sine `u=sin(sum x_i^2)` d=2, 比较 flat depth3 60 iter、greedy curriculum 20/20/20 和 depth1-only 20 iter, 3 seeds, 均在本机 RTX 4060 Ti 上运行.
  - Metric: positive mechanism signal 要求 curriculum 在 equal-budget 下达到 flat depth3 的 >=90%, 且比 depth1-only 高 >=5%; 否则只能说明浅层早停或 baseline undertraining.
  - Result: Poisson curriculum mean score 0.860 vs flat 0.134, 但每个 seed 的最佳都来自 depth1, 所以它证明的是 shallow-first. Radial-sine 上 curriculum mean score 0.303 vs flat 0.251 (1.21x), 但 depth1-only 已有 0.298, curriculum 只高 1.6%; depth3 grow phase 常常退化.
  - Signal: WEAK

- Claims and Claims matrix: POSITIVE: 在 compositional PDE 类上, transfer-tested top-k soft locking 用更少候选评估达到 flat FEX 同等 error, 且模块 transfer score 与后续收益显著相关; claim 限定为有可组合浅层结构的 PDE. NULL: 若 top-k soft locking 只等同于 depth1 early stopping, 本文 claim 降为"FEX 需要先测 shallow sufficiency, 不能默认 grow-and-lock 有用". NEGATIVE: 若 hard 和 soft locking 都伤害非组合 PDE, 可声称 optimal-substructure violation 是 FEX curriculum 的边界条件, 但不能声称 curriculum 普遍加速.
- Narrative: 这篇论文不讲一个通用 curriculum trick, 而问一个更窄的问题: FEX 的深树搜索里, 哪些浅层表达式真的能作为后续树的构件? 方法贡献是把"高浅层 reward"替换成"可转移模块"这个筛选对象.
- Experiments: 主实验只做方法证据, 不构建 benchmark. 比较 flat FEX、原始 expanding tree、depth1-only、warm-start without locking、greedy hard locking、top-k soft locking、random locking. 任务覆盖原 Poisson、conservation law、Schrodinger、radial-sine、乘积分离解和一个故意违反最优子结构的非组合制造 PDE. 指标为候选评估数、wall-clock、best residual、final relative-L2、transfer-score/后续收益相关性、锁定模块多样性和失败时的 phase-wise regression.
- Assets status: Poisson pilot 和 radial-sine mechanism check 均已完成; 数据与结果交接见 `workspace/fex-curriculum-depth/data/MANIFEST.md`.

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: SPL (Sun et al. 2022, ICLR 2023) — MCTS-based module transplantation; VSR-DPG (Jiang & Xue 2024, IJCAI 2024) — vertical SR via deep policy gradient; original FEX expanding tree (Liang & Yang 2022) — no cross-depth transfer.
- Key differentiator: 不是泛泛的"curriculum for FEX", 而是在 FEX/PDE residual/policy-gradient controller 这一特定场景下, 用 cheap downstream transfer probe 区分 reusable module 与 shallow shortcut, 再做 top-k soft locking. SPL 做 module transplantation 但基于 MCTS reward 而非 RL policy gradient + PDE residual; VSR-DPG 沿变量轴而非深度轴; 原始 FEX expanding tree 各深度独立无迁移. Delta 真实但偏窄 — "soft over hard locking"和"transfer test before commit"是通用直觉, 贡献在于 FEX/PDE 场景下的具体机制设计.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v2 删除了"指数分解成多项式"的过强 claim, 改成可测试的 optimal-substructure 条件, 明显进步. 剩余 gap: transfer probe 的具体定义 (用什么 metric, 多少 budget 算够)、soft prior 如何注入下一深度 controller (是 reward bonus / action-space bias / initial distribution)、top-k 中 k 的选择策略, 均未具体化. |
| Missing evidence signals | 6/10 | 计划中的 baselines 非常完整 (flat FEX、expanding tree、depth1-only、warm-start、hard/soft/random locking、compositional vs non-compositional PDE). 但现有 pilot evidence 弱: radial-sine 上 curriculum 仅比 depth1-only 高 1.6%, depth3 grow phase 常退化. 最急需的信号: (a) transfer score 与后续模块收益的 rank correlation; (b) oracle-module ablation (若已知最优子表达式, locking 是否有效); (c) matched wall-clock comparison. |
| Narrative | 7/10 | 已从 generic "curriculum for FEX" 升级为 "哪些浅层表达式真能成为后续构件", 叙事更像论文问题. 可进一步把 transfer probe 作为核心 scientific object (类似"compositionality litmus test"), 而不是把 soft locking 写成工程 trick. |
| Venue contribution | 6/10 | topic 未声明 target-venue; 按 NeurIPS/ICML/JMLR-level method paper 标准推断. 若只证明"FEX 也能做模块复用"(已知 SPL/VSR/FePySR 均有同质直觉), 顶会增量不足. 若确实发现 transfer score 能稳定预测可组合性, 且 soft locking 在 compositional PDE 上超越所有 baseline, 则贡献可支撑顶会. 目前处在一个"可测的好问题"阶段, 尚未到"已有顶会级证据的方法". |
| Testability | 8/10 | Expected outcome 有清楚的 cheap falsifier: equal-budget depth-3/4 测试中 top-k soft locking 不能超过 depth1-only 或 warm-start. 建议再加一个更便宜反证: transfer score 排序不优于 shallow reward 排序或 random ranking — 这个仅需已完成 pilot 数据即可验算. |
| Outcome realism | 6/10 | v2 明确承认"pilot 只支持小预算 sample efficiency, 不支持绝对能力提升", claims 边界合理. 但期望的 ">=50% candidate reduction + matched error + 多 PDE + transfer score predictive" 仍偏乐观. 成功更可能局限在 compositional PDE 子类; non-compositional case 更可能给出边界诊断而非强 method win. |
| Contribution type compliance | N/A | topic 0616-fex 未声明 `preferred-contribution-types`, 跳过. Idea 声明 type: method, 自身一致. |
| Overall Quality | 7/10 | v2 在叙事、claims discipline、实验设计上显著优于 v1 (5→7), 但核心正信号仍未出现. 现在是一个 well-formed 的可测假说, 不是一个已有证据的方法. 若下一轮能产出 transfer-score rank correlation 和至少一个 compositional PDE 上的 soft-locking-win, Quality 会再上一个台阶. |

## Contribution Drift

- v_{n-1} contribution types: {method}
- v_n contribution types: {method}
- Status: unchanged
- Hard cap triggered: no. topic 未声明 `preferred-contribution-types`, 无 silent downgrade (idea body 明确写了 `Contribution drift note` 说明诊断实验用于约束方法 claim 而非改写成 benchmark/application, 理由成立).

## Alternative Framing

当前 framing 已较锐 ("FEX search 是否有可利用的组合迁移结构"), 但可更聚焦: **"Can a cheap transfer probe distinguish reusable FEX sub-expressions from shallow shortcuts?"** 这样 positive 是方法+发现 (transfer probe works and soft locking 利用它), null/negative 也是关于 FEX search compositionality 的有效发现 (transfer probe 不比 shallow reward 强, 说明 FEX search 缺乏可被廉价探测的组合结构). 比"soft locking 加速 FEX"更抗失败.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在有组合浅层结构的 PDE 类上, transfer-tested top-k soft locking 在 matched candidate/wall-clock 下比 flat FEX、expanding tree、warm-start、hard/random locking 更快达到同等 residual/L2, 且 transfer score 显著预测后续 module gain (rank correlation > shallow reward 的 rank correlation). |
| NULL | 若 top-k soft locking 在 equal-budget 下不能超过 depth1-only 或 warm-start, 只能声称 early shallow sufficiency, 不能声称跨深度模块迁移有效. |
| NEGATIVE | 若 hard/soft locking 在非组合 PDE 上均伤害性能, 可声称 optimal-substructure violation 是 FEX curriculum 的边界条件; 不能声称 curriculum 普遍加速. |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale: **Likelihood: Low** — 顶会成功依赖多个尚未观察到的关键条件: (a) transfer score 必须比 shallow reward 更能预测后续模块收益; (b) top-k soft locking 必须在 compositional PDE 上超过 depth1-only 和 warm-start; (c) 需要找到至少 2-3 个 PDE 展示正信号. 当前 radial-sine pilot (更 rigorous 的那个) 上 curriculum 仅比 depth1-only 高 1.6%, 不支持上述条件. v2 机制 (transfer probe + soft locking) 虽未在 pilot 中验证, 但依赖的基础现象 (浅层表达式可作为深层构件) 的存在性尚未确定. **Impact: High** — 若成立, 直接影响 FEX 树搜索的效率和"可组合模块"这条研究线, 提供关于 FEX 搜索空间结构的可出版洞察. 但不是 Exceptional, 因为不会改写整个 SR/PDE solver 领域的判断. Claude 与 codex 对 Likelihood 和 Impact 判断一致, 无分歧.

## Overall

- Priority: Medium
- Score: 5
- Comments: v2 相比 v1 的质量提升是实质性的 (叙事、claims discipline、实验设计均有明显改进), 但核心正信号仍未出现. 当前是一个 well-formed 的可测假说, 建议下一步优先跑最便宜的证据信号 — 在已有 pilot 数据上验算 transfer score vs shallow reward 的 rank correlation — 如果这个信号为负, 可能需要重新评估整个 transfer probe 机制的可行性. Score 与 v1 持平 (5), 不是因为 v2 没进步, 而是因为 refinement 主要提升了 clarity/testability 而非降低了根本风险 (compositional transfer 可能压根不存在或太弱).

</review>

## Deep Lit 2026-06-16 (idea scope, 2 rounds, 10 papers)

以下论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-curriculum-depth` 于 2026-06-16 系统性搜索并精读后收录。搜索覆盖 method / application / data / evaluator / failure-mode / adversarial 六轴 + B7 反向扩展，R2 B7 产出 0 篇新 SR 相关论文，文献调研饱和。

### R1: 直接相关论文 (6 篇)

- **2511.07372** [wiki](wiki/2511.07372.md): Bu, Huang, Han, Suzuki 等, 2025.11. **Provable Benefit of Curriculum in Transformer Tree-Reasoning Post-Training**. 理论框架：将 CoT 建模为 autoregressive reasoning tree，证明 depth-increasing + hint-decreasing curriculum 使 RL finetuning 获得多项式样本复杂度（vs 直接训练指数瓶颈）。对 idea 价值：(a) 为 "depth-staged curriculum for RL tree search" 提供形式化理论支持，(b) coverage coefficient 概念可适配为 FEX controller 的 "表达式树修复难度" 度量，(c) test-time scaling via hierarchical verification 思路可启发 transfer probe 的分层验证设计。**撞车风险: 低**——面向 LLM reasoning post-training 而非 SR/PDE/FEX，但会压缩 "curriculum for tree search" 的 novelty claim。

- **2509.19710** [wiki](wiki/2509.19710.md): Roy, Dey, Pati, Mallick, 2025.09. **HierBOSSS: Hierarchical Bayesian Operator-induced SR Trees**. 贝叶斯层次树先验 + MCMC (Metropolis-within-partially-collapsed Gibbs) 做 SR，建立 near-minimax posterior concentration rate（首个 SR 贝叶斯理论保证），用 JMP-ensemble / Occam window 做模型选择、mGED 做结构距离。对 idea 价值：(a) 树先验的 GROW/PRUNE 操作与我们的 depth-staged search 有方法学共鸣，(b) Occam window 模型选择可作为 top-k soft locking 中 "保留几个候选" 的 principled 替代方案，(c) mGED 结构距离可作为 transfer score 的备选度量。**撞车风险: 低**——贝叶斯 vs RL 范式不同，但树结构先验和模型选择机制可借鉴。

- **2602.23561** [wiki](wiki/2602.23561.md): Roy, Dey, Mallick, 2026.02. **VaSST: Variational Soft Symbolic Trees**. 用 Binary Concrete + Gumbel-Softmax 将离散表达式树连续化为 soft symbolic trees，变分推断优化 ELBO + 温度退火，最后采样 hard trees。**此论文与本 idea 的 "soft locking" 概念最接近**——两者都替换 hard discrete decision 为 soft probabilistic prior。对 idea 价值：(a) soft tree relaxation 的梯度优化方法可直接为 "soft prior 注入下一深度 controller" 提供实现参考，(b) Gumbel-Softmax 温度退火策略可类比 transfer probe 中 "soft→hard" 的 annealing schedule，(c) hard-tree posterior sampling 可作为 top-k selection 的替代方案。**撞车风险: 中**——若 idea 强调 "soft/continuous relaxation of expression tree search"，VaSST 是直接 prior；差异化在于 FEX 使用 RL policy gradient + PDE residual（而非 VI + data-driven）且 VaSST 不做跨深度模块迁移。

- **2511.20506** [wiki](wiki/2511.20506.md): Varughese, Sankaranarayanan 等, 2025.11. **SR + RL for Interatomic Potentials**. Equation learner network + continuous-action MCTS + gradient descent 从 DFT 数据学习 Cu 的 EAM-like 解析势。对 idea 价值：RL+expression search 的应用范例，continuous-action MCTS 的 exploration 机制可对比 FEX controller 的 policy gradient。**撞车风险: 低**——应用领域 (materials) 和方法 (MCTS vs policy gradient) 均不同。

- **2605.31276** [wiki](wiki/2605.31276.md): Morales & Sheppard, 2026.05. **Neuro SR for N-response Curves**. Multi-Set Transformer 预测共享 symbolic skeleton → GA 拟合系数。对 idea 价值：skeleton-first (结构先于参数) 的 staged pipeline 与我们的 "shallow skeleton → deep completion" 思路一致，但它是 Transformer 提取 + GA 拟合而非 RL controller + transfer probe。**撞车风险: 低**。

- **2602.22964** [wiki](wiki/2602.22964.md): Floren & Swevers, 2026.02. **Guided Residual Search for Nonlinear State-Space ID**. 线性基线 → guided residual search → multiple shooting 的 staged decomposition。对 idea 价值：staged decomposition 方法论参考，residual-based refinement 可类比 "浅层残差 → 深层补全"。**撞车风险: 低**——面向系统辨识而非 SR。

### R2: 反向扩展发现 (4 篇)

- **2606.06567** [wiki](wiki/2606.06567.md): Reuter & França, 2026.06. **Are you sure? UQ in Symbolic Regression Survey**. 首篇 SR-UQ 系统综述，覆盖 frequentist / Bayesian / model selection 三类 18 篇工作。对 idea 价值：为 transfer probe 的可靠性评估提供方法论框架（profile likelihood、conformal prediction、MDL），可作为 top-k selection 中 UQ 驱动的候选排序参考。**撞车风险: 低**——综述，非方法竞品。

- **2512.10849** [wiki](wiki/2512.10849.md): Bomarito & Leser, 2025.12. **Bayesian SR via Posterior Sampling (SMC-SR)**. SMC + normalized marginal likelihood + GP-style MCMC rejuvenation 做 SR 后验采样，在噪声数据上比 GP baselines 更少过拟合。对 idea 价值：(a) posterior population mode selection 可作为 top-k 中 "选哪个候选" 的替代准则，(b) 噪声鲁棒性实验设计可借鉴。**撞车风险: 中**——若 idea 涉及 Bayesian/posterior expression selection 则需引用。

- **2510.24832** [wiki](wiki/2510.24832.md): Wang, Hao 等, 2025.10. **Scheduling LLM RL with Reasoning Trees (Re-Schedule)**. 提出 r-score：基于 reasoning tree 结构度量 query 学习难度，按 easy-to-hard 做动态样本权重调度。对 idea 价值：**r-score 的 "tree structural repairability" 思路可直接适配为 FEX expression-tree 的难度度量**——浅层表达式树结构越简单 (高 r-score)，controller 越容易修复/改进，应优先训练。这是本 tick 发现的与本 idea 方法论最接近的外部工作（虽面向 LLM 而非 SR）。**撞车风险: 低**——面向 LLM RLVR 而非 SR，但 tree-structure-based difficulty curriculum 的 claim 会部分重叠。

- **2506.06632** [wiki](wiki/2506.06632.md): Parashar, Gui, Li 等, 2025.06 (67 citations). **E2H Reasoner: Curriculum RL Easy-to-Hard for LLM Reasoning**. 核心发现：easy tasks 早期缓解 sparse reward，但必须及时淡出否则过拟合；提供 finite-sample complexity bounds 证明 curriculum stages 需要更少总样本。对 idea 价值：(a) "easy must fade" 的洞察直接对应我们的 "shallow shortcut" 担忧——浅层高分表达式可能是 shortcut 而非 reusable module，(b) Gaussian scheduler 的 easy-to-hard 过渡机制可适配 transfer probe 的 annealing schedule，(c) zero-advantage batch 诊断可帮助检测 FEX controller 的 curriculum 失效。**撞车风险: 低**——面向 LLM reasoning，但 "curriculum RL for structured search" 的 claim 会部分重叠。

### 对 idea claims 的影响

| Idea claim | Deep-lit 后发现 | 关键证据 |
|-----------|----------------|---------|
| Depth-staged curriculum 可提升 RL tree search 效率 | 获得外部理论支持 | 2511.07372 (curriculum → poly sample complexity) + 2506.06632 (E2H finite-sample bounds) |
| Transfer probe 区分 reusable module vs shortcut | 仍无直接 prior | 无外部工作研究 FEX/PDE 场景下的下游 transfer probe |
| Soft locking 优于 hard locking | 获得方法学共鸣 | 2602.23561 (VaSST soft trees) 提供连续松弛范本；2510.24832 (r-score) 提供 tree-structure difficulty 度量 |
| Compositional PDE 存在可迁移子表达式 | 仍无外部验证 | 无 work 在 PDE residual 场景下测试表达式树的可组合迁移性 |

**总体评估**: 本次 deep-lit 确认了 idea 的核心差异化空间（transfer probe + soft locking for FEX/PDE）仍然是空白的。最接近的外部工作 (VaSST, Re-Schedule, E2H) 分别来自 Bayesian SR 和 LLM reasoning 社区，均未触及 FEX 的 RL policy-gradient controller + PDE residual 场景。curriculum RL for tree search 的理论基础已由 2511.07372 和 2506.06632 建立，可作为 related work 引用并差异化（FEX 的 continuous parameter inner loop + BFGS 噪声 + 表达式等价类 vs LLM 的 discrete token-level RLVR）。
