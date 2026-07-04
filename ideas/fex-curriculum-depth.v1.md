---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-curriculum-depth/
---

- One-sentence summary: 设计渐进式树深搜索策略: 从 depth-1 开始搜索最优子表达式, 锁定后作为叶节点嵌入更深的树中继续搜索, 用 curriculum learning 替代 FEX 当前的固定深度一次性搜索.
- Hypothesis: FEX 的搜索空间随树深指数增长 (|U|^(2^L - 1) * |B|^(2^(L-1) - 1) 种组合); 分阶段搜索可以将指数级搜索分解为多次多项式级搜索, 总搜索量远低于一次性搜索, 且找到的表达式质量不低于固定深度搜索.
- Expected outcome: 成功: 在 depth-4 和 depth-6 的 benchmark 上, curriculum 策略的总搜索迭代次数 < 固定深度搜索的 30%, 且最终精度不劣于固定深度; 失败: 早期锁定的子表达式在后续阶段成为瓶颈 (greedy locking 的局部最优陷阱), 精度显著下降.
- Contribution type: method
- Risk: MEDIUM
- Estimated effort:
  - Compute: 30 GPU-hours on A100
  - Data: available (FEX 标准 benchmark)
  - Implementation: 2-3 weeks
- Novelty quick-check: FEX 原始论文的 convergence 实验扫了 depth 2-8 但每次从零搜索. SPL (Sun et al. 2022) 在 MCTS-SR 中引入 modularity 概念 (复用子表达式) 但机制不同. 无人在 RL-based expression tree search 中做过显式的 curriculum depth growing.
- Strongest objection: 子表达式锁定可能导致不可恢复的 greedy error -- 真正的全局最优解可能需要不同的子表达式组合.
- Why we should do this: FEX 论文中已观察到搜索时间随树深指数增长, 这是 FEX scalability 的核心瓶颈; curriculum depth 是解决这一瓶颈的最直接方案, 且提供了关于搜索空间结构的可出版洞察.
- Pilot:
  - Setup: Poisson d=10, (A) 标准 depth-3 搜索 40 iter, (B) 3-phase curriculum (depth1->depth2->depth3, 8 iter/phase, 24 iter total), seed 0
  - Metric: 若 curriculum (B) 的 score > standard (A) 的 90% 且迭代数更少, 信号为正
  - Result: Standard: score=0.186, error=19.05, 40 iter, 343s. Curriculum: score=0.889, error=0.016, 24 iter, 119s. Curriculum 达到 4.8x 更高的 score, 用了更少的迭代 (24 vs 40) 和更少的时间 (119s vs 343s).
  - Signal: POSITIVE

<review date="2026-06-16">

## Novelty

- Score: 5/10
- Closest prior work: (1) SPL (Sun et al. 2022, ICLR 2023) — Module Transplantation: high-reward MCTS parse trees are locked and reused as leaf nodes in future searches, with incremental size growing throughout iterations. (2) Original FEX (Liang & Yang 2022, JMLR 2025) — Algorithm 2 "FEX with progressively expanding trees" iterates through trees of increasing depth with early stopping, but each depth search is independent (no knowledge transfer between stages).
- Key differentiator: The idea's explicit mechanism — train separate RL controllers at each depth stage, embed the locked depth-(k) output as leaf nodes in the depth-(k+1) controller's action space — differs from both SPL (MCTS-based, not RL) and original FEX expanding tree (no knowledge transfer). The novelty lies in adapting sub-expression reuse to FEX's RL policy gradient framework, where the controller generates operator sequences in one shot rather than building trees step-by-step. However, the underlying concept of "lock good sub-expressions and reuse as building blocks" is present in SPL, and the original FEX paper already recognized the need for multi-depth search. The delta is genuine but modest.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | The exponential-to-polynomial decomposition claim relies on an unstated optimal-substructure assumption — greedy locking only works if the globally optimal depth-L expression contains the optimal depth-(L-1) sub-expression. This assumption is not tested or even stated. The idea acknowledges the greedy failure mode ("早期锁定的子表达式在后续阶段成为瓶颈") but doesn't characterize when it occurs. |
| Missing evidence signals | 4/10 | (1) Equal-budget baseline: pilot compares 40-iter fixed-depth vs 24-iter curriculum — different compute budgets. Need 40-iter fixed-depth vs 24-iter curriculum + 16 extra depth-3 iters. (2) Multi-seed evaluation (pilot uses seed 0 only). (3) Warm-start baseline without locking: 24-iter depth-3 search initialized near depth-2 best but without hard leaf embedding. (4) Top-k soft locking baseline (vs greedy hard locking). (5) Random locking baseline to test whether any locking beats no locking. (6) Ablation on when locking helps vs hurts (compositional vs non-compositional PDEs). |
| Narrative | 5/10 | "Curriculum learning for FEX" is too generic and undersells the specific contribution. The stronger framing is: "Does FEX search exhibit exploitable optimal substructure?" — this turns it from a heuristic trick into an empirical investigation of search structure, which is more publishable. The pilot result (4.8x score improvement) should be the hook, not buried. |
| Venue contribution | 5/10 | As a pure method contribution ("add curriculum to FEX"), this would struggle at JMLR/NeurIPS given prior art in SPL and original FEX expanding tree. Could become stronger if framed as a systematic empirical study of compositional structure in FEX search, with the algorithm as a consequence of the finding. The pilot signal is encouraging but the gap to a full paper is substantial. |
| Testability | 6/10 | The expected outcome contains a falsifiable claim (curriculum iter count < 30% of fixed-depth, accuracy not worse). A cheaper falsifier exists: equal-budget depth-3 tests across 5 seeds — if fixed-depth catches up with matched compute, the speedup claim collapses. Also: correlation between low-depth sub-expression score and its usefulness when embedded in deeper trees is a cheaper diagnostic than full depth-6 runs. |
| Outcome realism | 5/10 | The pilot baseline is concerning: fixed-depth FEX at 40 iterations scores only 0.186 with error 19.05 — this suggests severe undertraining, making the 4.8x improvement potentially an artifact of the baseline being broken rather than the curriculum being powerful. Speedup on compositional PDEs is plausible, but "accuracy not worse" at depth 4-6 and across diverse PDE types is risky given greedy locking's failure mode. |
| Contribution type compliance | N/A | Topic 0616-fex 未声明 `preferred-contribution-types`, 跳过. Idea 声明 type: method, 自身一致. |
| Overall Quality | 5/10 | Pilot signal is genuinely encouraging and the direction is sensible, but the novelty delta is narrower than the idea claims (SPL module transplantation + original FEX expanding tree), the pilot baseline is weak, and the narrative needs reframing from "curriculum trick" to "compositional structure investigation." |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- N/A (v1)

## Alternative Framing

Frame as: **"Does FEX search exhibit exploitable optimal substructure, and can soft module locking exploit it without the greedy failure mode?"** This reframes the idea from a generic curriculum heuristic into a sharper empirical claim: some PDEs have compositional solution structure that FEX can exploit through staged search, but the greedy locking strategy fails when the substructure assumption is violated. The algorithm becomes a consequence of characterizing WHEN locking works, rather than an asserted method. This would make the negative result (greedy locking fails on non-compositional PDEs) equally publishable.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | On FEX PDE benchmarks with compositional structure (Poisson, conservation law), staged module search with top-k soft locking reduces total candidate evaluations by >=50% while matching or exceeding fixed-depth FEX accuracy, under equal compute budgets and multi-seed evaluation. |
| NULL | Under equal compute budgets, curriculum depth search gives no reliable advantage over (a) fixed-depth FEX, (b) original FEX expanding tree, or (c) warm-start FEX without locking. |
| NEGATIVE | Greedy hard locking degrades final accuracy on PDEs where locally optimal low-depth sub-expressions do not compose into globally optimal deep expressions; top-k soft locking partially mitigates this but fundamental optimal-substructure violation limits the approach. |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale: **Likelihood: Low** — the novelty delta is narrower than claimed (SPL module transplantation + original FEX expanding tree significantly reduce the gap), the pilot baseline comparison is not fair (different compute budgets, single seed, likely undertrained baseline), and the greedy locking failure mode is a genuine risk that may limit the general applicability. A strong top-venue paper would require either (a) a systematic empirical characterization of when/why locking works that produces publishable insights, or (b) a non-greedy locking mechanism that provably avoids the substructure trap. **Impact: High** — if the idea succeeds robustly (curriculum depth search reliably reduces FEX search cost without quality degradation), it addresses the core scalability bottleneck of FEX and provides useful insights about search space structure that could influence the FEX research line. However, Impact is not Exceptional because this is a search heuristic improvement rather than a paradigm shift.

## Overall

- Priority: Medium
- Score: 5
- Comments: Claude 与 codex 对 Likelihood 和 Impact 判断一致 (均认为 Low Likelihood x High Impact), 无分歧。核心问题: (1) 需要公平基线比较 (equal budget, multi-seed); (2) 需要回应当前 pilot 中 baseline 可能 undertrained 的疑虑 (fixed-depth score 0.186 异常低); (3) 叙事应从 "curriculum for FEX" 转向 "compositional structure in FEX search", 使正面和负面结果均可发表; (4) 探索 top-k soft locking 作为 greedy hard locking 的替代方案以缓解最优子结构假设失效的风险。

</review>

## Deep Literature Review (2026-06-16, --scope idea)

Deep-lit 系统性搜索了 curriculum/progressive depth/staged search 在 RL-based 符号回归中的相关工作，2 轮循环，共读 12 篇论文（10 wiki_written + 2 already_read），B7 反向扩展 40 次。核心发现：**没有论文做与本 idea 完全相同的事（RL policy gradient + 逐深度 curriculum + sub-expression locking for FEX）**。

### 直接相关的增量式/分层 SR 方法

- **VSR-DPG (2402.00254)** [wiki](wiki/2402.00254.md): Nan Jiang, Yexiang Xue (Purdue), IJCAI 2024. **最接近的 prior work**。垂直符号回归：从少量变量开始，逐轮添加变量，每轮找到 best reduced-form equation 后将其作为 placeholder 嵌入下一轮。与 fex-curriculum-depth 的关键区别：(a) 沿变量轴增量而非深度轴；(b) 使用 grammar rule RNN 而非 expression tree RL controller；(c) 假设 data oracle 可做控制变量实验（FEX 不依赖此假设）。VSR-DPG 在 ≥5 active variables 时 recovery rate 降到 40%，是 FEX 可瞄准的 regime。**不撞车但高度相关，该引**。

- **SPL (2205.13134)** [已读]: Sun et al., ICLR 2023. Module Transplantation: MCTS 高 reward parse tree 锁定并作为叶节点复用于后续搜索。与 fex-curriculum-depth 的核心区别：(a) MCTS-based 而非 RL policy gradient；(b) 非显式 depth-staged 而是 reward-driven。该引为最直接的 prior。

- **Original FEX expanding tree** [已知]: Liang & Yang 2022, Algorithm 2. 从 depth-1 到 depth-8 逐深搜索但各深度搜索独立、无知识迁移。fex-curriculum-depth 创新恰好在于跨深度知识迁移。

### 课程学习与 SR

- **SDSR (2025)**: Zexin Lin et al., MIND 2025. 多 decoder LSTM + curriculum learning（逐步增加表达式长度）+ RL policy gradient 做耦合 PDE 系统 SR。课程学习机制与 fex-curriculum-depth 表面相似，但：(a) 在表达式长度（非树深）上做 curriculum；(b) LSTM decoder 生成而非 expression tree RL search；(c) 面向一般 SR 而非 PDE 求解。该论文无 arxiv preprint（仅有 DOI: 10.1109/MIND67540.2025.11351702），无法下载全文深度审读。**低撞车风险，方法论参考价值中等**。

### RL/搜索效率改进（间接相关）

- **Improved MCTS for SR (2509.15929)** [wiki](wiki/2509.15929.md): Huang et al., 2025. UCB-extreme + evolution-inspired state-jumping (mutation/crossover)。关键发现：state-jumping 的贡献远大于 bandit 改进（去掉 state-jumping 后 recovery 从 93% 降到 53%），提示**模块复用/非局部编辑**比 exploration constant 调整更关键。这支持 fex-curriculum-depth 的核心直觉（复用子表达式比调参更重要）。reward-tail KDE 诊断框架可直接移植到 FEX。

- **GP Inefficiency (2404.17292)** [wiki](wiki/2404.17292.md): Kronberger, França, Desmond et al., 2024. 用 exhaustive enumeration + equality saturation 量化 GP 搜索效率。关键发现：GP 只探索了少数 unique expressions，大量重复评估语义等价表达式。对 fex-curriculum-depth 的启示：(a) 逐深度锁定子表达式可类比 equality saturation 的语义去重；(b) 论文的 exhaustive enumeration 方法论可作为 curriculum depth 搜索效率的理论分析工具。

- **FormulaGPT (2404.06330)** [wiki](wiki/2404.06330.md): Li et al., 2024. 将 DSR/DSO 的 RL 搜索历史蒸馏进 Transformer，推理时做 in-context RL。Algorithmic Distillation 范式和 shortcut 数据增强技术可迁移用于加速 FEX controller 训练。

- **Complexity-Aware DSR (2406.06751)** [wiki](wiki/2406.06751.md): Bastiani et al., 2024. 在 risk-seeking policy gradient 中引入 complexity-aware 正则化。对 fex-curriculum-depth 的启示：低深度阶段应加入复杂度约束，避免锁定过于复杂的子表达式。

- **SR-Scientist (2510.11661)** [wiki](wiki/2510.11661.md): Xia et al., 2025. LLM as autonomous AI scientist for equation discovery with GRPO。正交方法（LLM agent + tool use），但 experience buffer 机制和 accuracy-to-tolerance 指标值得参考。

### 表示层工作（间接相关）

- **Fixed-Depth SR Grammars (2410.08137)** [wiki](wiki/2410.08137.md): Finkelstein, 2024. Faultless fixed-depth prefix/postfix grammar。可嵌入 FEX controller 消除无效表达式采样，减少 RL 训练浪费。

- **MMSR (2402.18603)** [wiki](wiki/2402.18603.md): Li et al., 2024. SR as multi-modal information fusion。SetTransformer 编码 data features 的方法可参考。

### 其他 B7 发现（边际相关，记录备查）

- **APPS (2409.01416)**: Nan Jiang et al., 2024. Active learning for ODE symbolic discovery via phase portrait sketching. 同组后续工作，边际相关。
- **Test-time Computation (2505.22081)**: Sato & Sato, 2025. Neural SR 中的 reproduction bias 缓解。低相关。
- **Knowledge Integration SR (2509.03036)**: Taskin et al., 2025. LLM for physics-informed SR。低相关。
- **SR Survey (2512.15920)**: Bartlett et al., 2025. Royal Society SR 特刊引言综述。提供 SR landscape 概览，无直接竞争。

### 撞车风险评估（针对 fex-curriculum-depth 的 3 个核心 claim）

| Claim | 撞车风险 | 证据 |
|-------|---------|------|
| 逐深度 curriculum + sub-expression locking for FEX | 🟢 **未被覆盖** | VSR-DPG 沿变量轴、SPL 沿 reward-driven 而非 staged-depth、SDSR 沿表达式长度、Original FEX expanding tree 无知识迁移。无任何论文在 RL-based expression tree search 中做显式 depth-staged curriculum with locking。 |
| Exponential→polynomial decomposition | 🟢 **理论空白** | 无任何论文分析 RL-based 表达式树搜索的 staged decomposition 复杂度。GP Inefficiency 提供了 equality saturation 的方法论参考但未涉及 RL。 |
| Greedy locking 失败模式表征 | 🟢 **完全空白** | 无任何论文系统研究 sub-expression locking 在何种条件下导致局部最优。这是可发表的 negative result 方向。 |

### 对 Idea 的修正建议

1. **叙事应从 "curriculum for FEX" 升级为 "compositional structure in FEX search"**：VSR-DPG 的存在使得 "curriculum" 本身不再是完全新颖的概念，但 "depth-axis staged search with sub-expression locking" 仍是未探索的。应聚焦于 **何时/为何 FEX 搜索空间展现出可被 curriculum depth 利用的 compositional structure**。

2. **公平基线至关重要**：reviewer 指出 pilot baseline 可能 undertrained（score 0.186 异常低）。建议在正式实验中使用：(a) equal-budget baseline (40-iter fixed-depth vs 24-iter curriculum + 16 extra depth-3 iters)；(b) warm-start baseline without locking；(c) top-k soft locking vs greedy hard locking。

3. **GP Inefficiency 的分析方法论可借鉴**：用 exhaustive enumeration + semantic deduplication 量化 FEX 各深度的 unique expression 数量，为 curriculum depth 的搜索效率提供理论下界。

4. **EGG-SR (2511.05849) 的等价类剪枝 + state-jumping 的复用逻辑**：这两个机制都与 curriculum depth 的 sub-expression 复用有精神上的共鸣，可作为 baseline 或增强组件。
